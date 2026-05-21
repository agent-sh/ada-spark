# Agent Knowledge Base

Synthesized research the `ada-spark` skill routes to for deep dives. The skill
(`skills/ada-spark/SKILL.md`) is the always-on correction layer; this directory holds the
sourced long-form material behind it. Prefer these dated, sourced facts over training data,
especially for the fast-moving toolchain.

## Guides

| Topic | Guide | Sources | Generated |
|-------|-------|---------|-----------|
| Ada & SPARK best practices, gotchas, ecosystem, LLM blind spots | [ada-spark-best-practices.md](ada-spark-best-practices.md) | 50 | 2026-05-21 |

## When to read the guide

Read `ada-spark-best-practices.md` for depth on:

- Ada / SPARK language questions, contracts (Pre/Post/Contract_Cases), proof, GNATprove
- Class-wide / inheritance contracts: `Pre'Class` / `Post'Class`, dispatching calls, Liskov Substitution (LSP), generics in SPARK (instances vs generic body)
- Alire / `alr` / package manager / crates / GNAT FSF / GNAT Pro / the discontinued GNAT Community
- Ada 2022 features (target name, `'Image`, delta aggregates, declare expressions, `'Reduce`, string interpolation)
- SPARK assurance levels (Stone/Bronze/Silver/Gold/Platinum), AoRTE, loop invariants, ghost code, ownership/borrow, Big_Numbers
- Static analysis: GNAT SAS (renamed from CodePeer), modes, baselines, SARIF, GNATcheck/MISRA, and how it differs from GNATprove (unsound heuristic vs sound proof)
- Tooling: GNAT, gprbuild, project files, GNAT Studio, Ada Language Server (ALS), VS Code, libadalang
- Embedded / bare-metal: Cortex-M, bb-runtimes, light/embedded runtimes, Ravenscar, Jorvik
- Gotchas: elaboration order, controlled types, fixed-point, representation clauses, exception propagation

## Why this base exists

The guide corrects stale or hallucinated LLM knowledge: GNAT Community was discontinued (use
Alire), Ada 2022 is finalized, SPARK has a Rust-like borrow checker, aspects (not pragmas) are
idiomatic for contracts, `Pre'Class`/`Post'Class` (not plain `Pre`/`Post`) govern dispatching
calls on tagged types, CodePeer is now GNAT SAS, and standard-library unit names are precise.
Re-verify exact version numbers against official release notes for production decisions.

## Maintenance

- Source metadata with confidence ratings lives in the `resources/` directory alongside this file.
- This project rides the latest toolchain; versions in the guide are the latest as of its
  generation date and should be re-verified against the GNAT FSF builds releases each session.
