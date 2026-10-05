# ada-spark

> Skill that teaches any coding agent to write idiomatic, correct, current Ada and SPARK. Distributed via the agent-sh marketplace.

**Repository**: https://github.com/agent-sh/ada-spark

## Writing Ada/SPARK here

Before writing, porting or reviewing Ada or SPARK in this repo, apply `skills/ada-spark/SKILL.md` over what you remember: the toolchain and ecosystem moved a lot in 2022-2026, so pretrained knowledge is likely stale. This project rides the latest toolchain (latest stable plus snapshots); check exact versions against the `alire-project/GNAT-FSF-builds` releases.

## Conventions

- Output is plain text with the status markers `[OK]`, `[ERROR]`, `[WARN]`, `[CRITICAL]`, and no emojis or ASCII art.
- In prose, write a spaced single dash (` - `), not ` -- ` or an em dash. CLI flags like `--help` are fine.
- Put summaries, plans and audit notes in the PR or issue, not in committed files.
- Changes reach main through a PR. Keep git hooks on, and answer every review comment, in the thread when you disagree.
- When a script or tool fails, report the failure before working around it, so the tool gets fixed.
- When goals conflict, rank them: plugin users' experience, automation that needs no babysitting, token cost, output quality, simplicity.

## Skill conventions

- `skills/ada-spark/SKILL.md` is the source of truth for how agents write Ada/SPARK; mirror any guidance added here into it.
- Keep the skill file to the correction map and the rules that apply to every task. Short, verified notes per area go in `skills/ada-spark/references/`, which ship with the skill; `agent-knowledge/` does not. Link to current upstream docs (learn.adacore.com, docs.adacore.com, ada-auth.org) for depth.
- Prefer cross-tool features, and mark any tool-specific behavior as such.
- Triggers should fire on realistic prompts ("write Ada", "prove this with SPARK", "is this idiomatic Ada", "set up an Alire project"). Keep the description short and name what it does not cover (Apache Spark, other languages).
- Ground Ada/SPARK guidance in current upstream docs, not memory, and cite sources for non-obvious rules: pre-2022 assumptions are the common failure.

## Testing

- `agnix --config .agnix.toml .` with zero errors, and `claude plugin validate .` for the manifests. CI runs both.
- For a trigger or description change, check that the skill activates on realistic prompts in Claude Code and one other tool (Cursor or Codex).
- Add a `CHANGELOG.md` entry under Unreleased.

## References

- Part of the [agent-sh](https://github.com/agent-sh) ecosystem
- Ada/SPARK learning portal: https://learn.adacore.com
- AdaCore docs (SPARK User's Guide, GNAT, GNAT SAS): https://docs.adacore.com
- Ada 2022 Reference Manual: http://www.ada-auth.org/standards/22rm/html/RM-TOC.html
- Alire (package manager): https://alire.ada.dev
- https://agentskills.io
