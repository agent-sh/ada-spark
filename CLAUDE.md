# ada-spark

> Skill that teaches any coding agent to write idiomatic, correct, current Ada and SPARK. Distributed via the agent-sh marketplace.

**Repository**: https://github.com/agent-sh/ada-spark

Follow the Karpathy Guidelines (simplicity, surgical changes, clear success criteria) when editing this plugin.

## Writing Ada/SPARK (read first)

Before writing, porting, or reviewing any Ada or SPARK, apply `skills/ada-spark/SKILL.md` and prefer it over pretrained knowledge. WHY: the Ada/SPARK toolchain and ecosystem moved substantially in 2022-2026, so pretrained knowledge is likely stale and will misroute users. The skill carries the always-on correction map (GNAT Community is dead -> Alire `alr` + GNAT FSF; aspects `with Pre =>` not `pragma Precondition`; `Pre'Class`/`Post'Class` govern dispatching calls, not plain `Pre`/`Post`; CodePeer is now GNAT SAS; SPARK has a Rust-like move/observe/borrow model for pointers; Ada 2022 is finalized, use `-gnat2022`) plus links to current upstream docs. This project rides the latest toolchain (latest stable + snapshots). Verify exact versions against `alire-project/GNAT-FSF-builds` releases, not memory.

## Project Instruction Files

- `CLAUDE.md` is the project memory entrypoint for Claude Code.
- `AGENTS.md` is a byte-for-byte copy of `CLAUDE.md` for tools that read `AGENTS.md` (Codex CLI, OpenCode, Cursor, Cline, Copilot).
- Keep them identical. When editing one, update the other in the same commit.

## Critical Rules

1. **Plain text output** - No emojis, no ASCII art. Use `[OK]`, `[ERROR]`, `[WARN]`, `[CRITICAL]` for status markers.
2. **No unnecessary files** - Create only the files a task needs; skip summary, plan, audit, and temp docs unless required.
3. **Task is not done until it passes** - Every feature/fix needs quality tests, and `agnix` must be green before merge.
4. **Create PRs for non-trivial changes** - No direct pushes to main.
5. **Always run git hooks** - Let pre-commit and pre-push hooks run; never bypass them.
6. **Use single dash for em-dashes** - In prose, use ` - ` (single dash with spaces), never ` -- `. Does not apply to CLI flags like `--help`.
7. **Report script failures before manual fallback** - Surface broken tooling; never silently bypass it.
8. **Address all PR review comments** - Even minor ones. If you disagree, respond in the review thread.
9. **Wait for the claude workflow before merging** - It is the major quality gate and most thorough review; merge only after it passes.
10. **Follow the skill/command flow exactly** - If a flow specifies subagents, tools, or phases, follow them in order with no deviations.
11. **Token efficiency** - Save tokens over decorations.

## Skill Conventions

- `skills/ada-spark/SKILL.md` is the authoritative source of truth for how agents write Ada/SPARK; mirror any guidance added here into it.
- Keep the skill self-contained; do not bundle deep-dive guides - link to current upstream docs (learn.adacore.com, docs.adacore.com, ada-auth.org) for depth.
- Prefer cross-tool features. Clearly gate any tool-specific behavior.
- Triggers should fire on realistic prompts ("write Ada", "prove this with SPARK", "is this idiomatic Ada", "set up an Alire project"). Use "Skip unless:" gates to avoid false activation.
- Ground Ada/SPARK guidance in current upstream docs, not memory - the toolchain changes fast and pre-2022 assumptions are common failure modes. Cite sources for non-obvious rules.

## Model Selection

| Model | When to Use |
|-------|-------------|
| **Opus** | Complex reasoning, analysis, planning |
| **Sonnet** | Validation, pattern matching, most agents |
| **Haiku** | Mechanical execution, no judgment needed |

## Core Priorities

1. User DX (plugin users first)
2. Worry-free automation
3. Token efficiency
4. Quality output
5. Simplicity

## Testing

Before considering changes complete:
- Run `agnix` on the skill (zero errors).
- Verify the skill activates on realistic prompts in Claude Code and at least one other tool (Cursor or Codex recommended).

## References

- Part of the [agent-sh](https://github.com/agent-sh) ecosystem
- Ada/SPARK learning portal: https://learn.adacore.com
- AdaCore docs (SPARK User's Guide, GNAT, GNAT SAS): https://docs.adacore.com
- Ada 2022 Reference Manual: http://www.ada-auth.org/standards/22rm/html/RM-TOC.html
- Alire (package manager): https://alire.ada.dev
- https://agentskills.io
