# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-09-24

### Changed
- Rewrote the skill for current models: a short trigger description, a toolchain check, the correction table, rules that hold across tasks, and a definition of done. Removed the all-caps headings, the "Do NOT" list that restated the table, the workflow steps and the `Skip unless` gates.
- Moved detail into `skills/ada-spark/references/` (`spark-proof.md`, `toolchain.md`, `embedded-and-portability.md`), which ship with the skill. The skill previously pointed at `agent-knowledge/`, which is outside the skill directory and missing wherever only the skill is installed.

### Fixed
Checked against the Ada 2022 RM, the SPARK User's Guide sources, the GNAT User's Guide, the GNAT SAS User's Guide, the Alire changelog and AdaCore `bb-runtimes` on 2026-09-24. Also corrected in `agent-knowledge/ada-spark-best-practices.md`.
- `alr install` exists since Alire 2.0 (binary releases such as `gnat_native`, `gprbuild`, `gnatprove`); the skill said it did not.
- The skill called a specific `Pre` on a tagged primitive illegal Ada. It is legal (RM 6.1.1 forbids it only on abstract subprograms and null procedures) but not inherited; it is SPARK that rejects it on dispatching subprograms (SPARK RM 6.1.1(2)). Added the rule that `Pre'Class` on an override needs a `Pre'Class` on an ancestor.
- `SPARK_Mode => Auto` is not a valid aspect (`Auto` is configuration-pragma only), and `pragma SPARK_Mode;` with no value means `On`; the skill listed the bare pragma as stale.
- GNAT's default language mode is Ada 2012, not Ada 2022.
- GNAT SAS is distributed to AdaCore customers, not through Alire; said so.

## [0.1.0] - 2026-05-21

### Added
- Initial release of the `ada-spark` skill plugin.
- `skills/ada-spark/SKILL.md` - teaches agents to write idiomatic, correct, current Ada and SPARK, with an always-on correction map (GNAT Community is dead -> Alire + GNAT FSF; aspects `with Pre =>` not `pragma Precondition`; `Pre'Class`/`Post'Class` for dispatching contracts; CodePeer renamed to GNAT SAS; SPARK move/observe/borrow ownership; Ada 2022 with `-gnat2022`), plus core rules and blind-spot guidance for contracts, the SPARK proof model and assurance ladder, class-wide contracts/LSP, generics in SPARK, GNAT SAS vs GNATprove, embedded runtimes (Ravenscar/Jorvik), and portability traps.
- `agent-knowledge/ada-spark-best-practices.md` - 50-source research foundation the skill routes to for deep dives.
- Claude Code and Codex plugin manifests; `agnix` and `claude plugin validate` CI; release-notify workflow; `CLAUDE.md`/`AGENTS.md` project memory.
