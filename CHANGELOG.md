# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-05-21

### Added
- Initial release of the `ada-spark` skill plugin.
- `skills/ada-spark/SKILL.md` - teaches agents to write idiomatic, correct, current Ada and SPARK, with an always-on correction map (GNAT Community is dead -> Alire + GNAT FSF; aspects `with Pre =>` not `pragma Precondition`; `Pre'Class`/`Post'Class` for dispatching contracts; CodePeer renamed to GNAT SAS; SPARK move/observe/borrow ownership; Ada 2022 with `-gnat2022`), plus core rules and blind-spot guidance for contracts, the SPARK proof model and assurance ladder, class-wide contracts/LSP, generics in SPARK, GNAT SAS vs GNATprove, embedded runtimes (Ravenscar/Jorvik), and portability traps.
- `agent-knowledge/ada-spark-best-practices.md` - 50-source research foundation the skill routes to for deep dives.
- Claude Code and Codex plugin manifests; `agnix` and `claude plugin validate` CI; release-notify workflow; `CLAUDE.md`/`AGENTS.md` project memory.
