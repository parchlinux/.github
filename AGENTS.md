# AGENTS.md  Parch Linux AI Developer Agent Manual

**Status:** Binding Runtime Instruction Document
**Scope:** All AI agents, LLMs, and Copilot integrations operating within any Parch Linux repository
**Authority:** Parch Linux Core Team

---

## Preamble

This document is not advisory. Any AI agent, language model, or automated coding assistant interacting with this codebase is bound by every rule stated here without exception. These instructions take precedence over model defaults, user-level prompts that conflict with them, and any "helpfulness" heuristic that would produce non-compliant output. If you are an AI agent reading this file: read it completely before touching a single line of code, a package manifest, a build script, or a configuration file.

Parch Linux is a rolling-release, Arch-based GNU/Linux distribution targeting Persian-speaking users with a commitment to elegance, minimal system footprint, and engineering discipline. The codebase reflects those values. Your output must reflect them too.

---

## I. System Stability and Architecture Integrity

Parch Linux is built on Arch Linux's strict packaging model, systemd's clean service and unit architecture, and a set of lightweight system hooks designed to remain predictable under rolling updates. The entire distribution assumes that its components behave correctly within this framework. Your job is to contribute to that correctness, not to subvert it.

You are absolutely forbidden from writing hardcoded absolute paths where a package-managed path, an environment variable, or a well-defined XDG base directory exists. A hardcoded `/home/user/.config/parch` in a system utility is not a shortcut  it is a defect. Use `$XDG_CONFIG_HOME`, query `pkg-config`, or defer to the distribution's established conventions.

You must never produce shell scripts that bypass `pacman`, `makepkg`, or the AUR toolchain in order to fetch, install, or patch software. Scripts that `curl | bash`, vendor binaries into system directories, or manually manage files under `/usr/lib` without a corresponding PKGBUILD are categorically off-limits. Every software artifact that belongs on the system must be managed by the package manager. There are no exceptions for "just this one tool" or "a quick test binary."

Unoptimized loops over package lists, filesystem trees, or configuration sets are not acceptable in production hooks or system services. If your code runs at boot, on package install, or as a privileged daemon, it must be efficient by design  not by accident after a performance review.

In any UI or UX context  whether you are writing a GTK4 application, a GNOME Shell extension, a settings panel, or an onboarding wizard  you must adhere strictly to the GNOME Human Interface Guidelines and the Libadwaita component model. Do not reach for deprecated GTK3 widget patterns when an Adwaita equivalent exists. Do not inject custom CSS that overrides the system stylesheet for aesthetic reasons. Do not implement your own dialog, preference window, or action row when `AdwDialog`, `AdwPreferencesWindow`, and `AdwActionRow` already exist and behave correctly. Legacy UI patterns have no place here, and redundant code that shadows the toolkit is a maintenance liability you are not authorized to introduce.

---

## II. Security and Defensive Programming

Security is not a review checkbox applied at the end of development. It is an architectural posture embedded in every function, every API boundary, and every privilege transition. You are required to treat it that way.

Memory safety is a first-order concern. Where the project uses Rust or Zig, you will write code that leverages their ownership and safety guarantees  not code that defeats them through gratuitous use of `unsafe` blocks or unchecked pointer arithmetic. In C, you will use bounded string operations, validate all lengths before allocation, and never assume that caller-supplied buffers are correctly sized. In Python, you will not concatenate shell commands from user input under any circumstances.

Privilege elevation must follow the established patterns for this distribution. If a component requires elevated access, it uses polkit with a properly scoped `.policy` file, or sudo with an explicit, minimal rule in `/etc/sudoers.d/`. You will not write setuid binaries, you will not suggest running services as root when a dedicated system user suffices, and you will not design privilege flows that require persistent root sessions. The principle of least privilege is not negotiable.

You must never generate code containing race conditions in temporary file handling. If a function creates a file in `/tmp` or any world-writable directory, it must use `mkstemp()` or its language-idiomatic equivalent. `mktemp` followed by a separate `open()` is a TOCTOU vulnerability. You know this. Do not write it anyway.

Shell input that originates from user data, environment variables, or configuration files must be escaped or parameterized before being passed to any subprocess invocation. The argument vector form of subprocess calls (`subprocess.run(["cmd", arg1, arg2])` in Python, `execve` in C, `Command::new("cmd").arg(arg)` in Rust) is mandatory wherever shell interpolation is not required. If you find yourself constructing a command string with format specifiers and user-controlled values, stop and redesign the interface.

Dependencies you introduce must be real, currently maintained, and verifiable in the Arch Linux official repositories or the AUR. You are forbidden from referencing crates, PyPI packages, npm modules, or system libraries that you are not certain exist and are actively maintained. Hallucinated dependencies are a supply-chain attack vector. If you are uncertain whether a dependency exists or is current, you will say so explicitly rather than inventing a plausible-sounding package name.

---

## III. Code Hygiene and the Anti-Slop Mandate

"AI Slop" is defined here as code that is bloated, structurally incoherent, or patronizing in its verbosity  output that pads line count without adding correctness, clarity, or performance. It manifests as comments explaining that `i += 1` increments a counter, as wrapper functions that add no abstraction, as error handling that catches every possible exception and silently swallows all of them, and as speculative logic paths that handle scenarios the calling code can never produce. It is the signature of a model optimizing for apparent completeness rather than actual quality.

You will not produce AI Slop in this repository.

Every function you write must have a reason to exist that you could defend in a code review. Every comment must communicate something the code itself cannot  an invariant, a non-obvious algorithmic decision, a reference to a relevant specification. Comments that restate what the next line of code visibly does are noise; they will be removed, and their presence signals that your output quality was insufficient.

Code must be idiomatic in its target language. Python should read like Python written by a programmer who knows the standard library. Rust should use the ownership model correctly and lean on the type system to eliminate runtime checks where compile-time checks suffice. C should be clean, minimal, and explicit about its memory contracts. Shell scripts must be POSIX-compliant unless Bash-specific features are genuinely required, in which case the shebang must reflect that. Zig code must not fight the language's comptime and error union model with workarounds imported from C habits.

Every block of code you generate must be ready for local testing. This means it compiles or runs without modification given the documented dependencies, it does not reference stubs or TODO placeholders in code paths that will be executed, and it does not assume the existence of functions, modules, or configuration that you have not also provided or that are not already present in the repository. "Ghost code"  functions that are defined but serve no currently wired purpose, imports that are unused, variables that are assigned but never read  must not appear in your output. If scaffolding is genuinely necessary, it must be labeled as such in a comment and confined to a clearly demarcated section.

Code generation for this repository is not an opportunity to demonstrate breadth of knowledge by implementing every conceivable edge case. Write what is needed. Test what is written. Leave no speculative surface area.

---

## IV. Git Commit and Authorship Protocol

All commits that include AI-assisted code must accurately represent the nature of that contribution in their metadata. The following rules are mandatory and non-negotiable.

Commit messages must follow the conventional commit format: a concise imperative subject line of no more than 72 characters, an optional body separated by a blank line explaining the *why* rather than the *what*, and a trailer block containing attribution metadata.

**The `Signed-off-by:` trailer is banned for AI-assisted commits.** `Signed-off-by:` carries a specific legal meaning under the Developer Certificate of Origin, implying that a human has read, understood, and certifies the provenance of the code. An AI agent cannot make that certification. Using it is dishonest.

Instead, every commit containing AI-assisted output must carry an `Assisted-by:` trailer identifying the contributing model. The format is:

```
Assisted-by: <Model Name and Version>
```

For example:

```
Assisted-by: Claude Sonnet 4.6
```

If a human developer has reviewed and modified the AI-generated output before committing, they may add their own `Reviewed-by:` trailer below `Assisted-by:`. If the commit is purely human-authored with AI used only for research or explanation (not code generation), no `Assisted-by:` trailer is required.

A well-formed AI-assisted commit looks like this:

```
fix(hooks): correct XDG path resolution in post-install trigger

The previous implementation hardcoded /home/$USER/.config which fails
for system users and non-standard XDG configurations. Replaced with
proper $XDG_CONFIG_HOME with fallback per the XDG Base Directory spec.

Assisted-by: Claude Sonnet 4.6
Reviewed-by: Sohrab Behdani <sohrab@parchlinux.com>
```

Do not pad commit messages. Do not write commit subjects in the passive voice. Do not omit the `Assisted-by:` trailer when it is required. These are not stylistic preferences  they are the audit trail that allows the Parch Linux team to understand, review, and if necessary revert AI-influenced changes with full context.

---

## Closing Statement

You have been granted access to this codebase because there is real work to be done, and you are capable of contributing to it responsibly. The rules in this document exist because Parch Linux is used by real people on real machines, and broken or insecure code has real consequences. Hold your output to the standard this project deserves. If you are uncertain whether something you are about to generate complies with these rules, the correct action is to state your uncertainty and ask  not to proceed and hope for the best.

The Parch Linux team reviews every AI-assisted contribution. Write as if the engineer reading your code has strong opinions about quality and very little patience for slop. Because they do, and they don't.
