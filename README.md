# ada-spark

A skill that teaches any coding agent to write idiomatic, correct, current Ada and SPARK.

Ada/SPARK tooling and ecosystem changed substantially between 2022 and 2026. The dominant failure mode for agents is stale advice: pointing users at the discontinued GNAT Community edition, writing `pragma Precondition` instead of contract aspects, claiming "SPARK can't do pointers", or calling the static analyzer "CodePeer" after it became GNAT SAS. This plugin ships an always-on correction map plus focused guidance for contracts, the Alire ecosystem, SPARK proof, and embedded - so agents target the current toolchain, not their training data.

## What it does

- Embeds a stale-to-current correction map agents apply on every Ada/SPARK task: GNAT Community is dead -> Alire (`alr`) + GNAT FSF; `with Pre =>` aspects not `pragma Precondition`; `Pre'Class`/`Post'Class` govern dispatching calls, not plain `Pre`/`Post`; CodePeer is now GNAT SAS; SPARK has a Rust-like move/observe/borrow model for access types; Ada 2022 is finalized (`-gnat2022`).
- Core rules for idiomatic packages, contracts (Pre/Post/Contract_Cases), strong typing, controlled types, and tasking.
- SPARK guidance: assurance levels (Stone/Bronze/Silver/Gold/Platinum), proof of absence of runtime errors (AoRTE), loop invariants, ghost code, ownership/borrow.
- Embedded guidance: Ravenscar vs Jorvik profiles, light/embedded runtimes, fixed-point and representation clauses, elaboration order.
- Links to current upstream docs (learn.adacore.com, docs.adacore.com, ada-auth.org) for depth, rather than bundling a deep-dive guide that would drift.

## Installation

```bash
claude plugin marketplace add agent-sh/ada-spark
claude plugin install ada-spark
```

Or add it through the agentsys marketplace alongside the rest of the agent-sh ecosystem.

## Usage

The skill activates automatically on Ada/SPARK work. Triggers include:

```text
write an Ada package for a bounded stack with contracts
prove this subprogram free of runtime errors with SPARK
is this idiomatic Ada 2022?
set up an Alire project targeting a Cortex-M board
why won't GNATprove discharge this loop invariant?
```

It does not activate for unrelated languages.

## Layout

```
skills/ada-spark/SKILL.md   the skill (correction map + core rules + workflow + doc links)
agent-knowledge/            research foundation the skill routes to for deep dives
.claude-plugin/             Claude Code plugin + marketplace manifests
.codex-plugin/              Codex plugin manifest
```

## Development

```bash
agnix .                  # validate skill + manifests (zero errors required)
claude plugin validate . # validate marketplace and plugin manifests
```

Ada/SPARK guidance is grounded in current upstream docs, not model memory - re-verify any toolchain version or stdlib unit against the installed toolchain and `alire-project/GNAT-FSF-builds` releases.

## Related Plugins

- `skill-curator` - for writing and reviewing skills
- `agnix` - the linter that validates this plugin
- Part of the [agent-sh](https://github.com/agent-sh) ecosystem

## License

MIT
