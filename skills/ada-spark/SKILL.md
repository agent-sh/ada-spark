---
name: ada-spark
description: "Use when writing, porting, reviewing, or proving Ada or SPARK - .ada/.adb/.ads/.gpr files, contracts (Pre/Post/Contract_Cases), Alire (alr) projects, SPARK proof (GNATprove, AoRTE, assurance levels, ownership/borrow), embedded (Ravenscar/Jorvik, light runtimes), and GNAT/GNAT SAS tooling. Targets the latest toolchain (Ada 2022, Alire + GNAT FSF) and blocks stale pre-2022 advice (GNAT Community, pragma contracts, CodePeer). Not for unrelated languages."
allowed-tools: Read, Edit, Write, Grep, Glob, Bash(alr:*), Bash(gnatmake:*), Bash(gprbuild:*), Bash(gnat:*), Bash(gnatprove:*), Bash(gnatsas:*), Bash(gnatformat:*)
---

# Ada & SPARK

Write idiomatic, correct, current Ada and SPARK. The toolchain and ecosystem moved hard between 2022 and 2026, so the dominant failure mode is stale advice: pointing users at the dead GNAT Community edition, writing pragma contracts, claiming "SPARK can't do pointers", or calling the analyzer "CodePeer". The correction map below is the single most important asset - apply it on every Ada/SPARK task. The second failure mode: code that compiles as Ada but is rejected by GNATprove, or OOP/contract idioms ported from other languages that violate Ada's tagged-type and LSP rules - see "What strong models get wrong".

Canonical docs: `learn.adacore.com`, `docs.adacore.com`, Ada 2022 RM at `ada-auth.org`.

## CRITICAL CORRECTION MAP - apply first

When you see the LEFT column in code, docs, or your own memory, it is STALE - use the RIGHT column.

| Stale (avoid) | Current (use) | Notes |
|---|---|---|
| "Download GNAT Community Edition" | Alire (`alr`) + GNAT FSF | GNAT Community ended; 2021 was the last release. Single most common stale instruction. |
| `pragma Precondition` / `pragma Postcondition` | aspect `with Pre =>` / `with Post =>` | Aspects (Ada 2012+) are idiomatic and portable. Pragmas still compile but are not the modern form. |
| `Pre` / `Post` on a dispatching tagged primitive | `Pre'Class` / `Post'Class` | Specific `Pre` is illegal on a tagged primitive (RM 6.1.1) and is never inherited / never checked for dispatching calls. |
| "SPARK can't do pointers" | move / observe / borrow ownership model | Rust-like affine ownership on access types since SPARK 20-22. |
| `codepeer` (CLI / product) | `gnatsas` (GNAT SAS) | CodePeer was renamed to GNAT Static Analysis Suite (~24 line, 2024). CLI is `gnatsas analyze`/`report`. |
| CodePeer `--level 0..4`, SQL results DB | `--mode=fast` / `--mode=deep`, `.sam`/`.sar` files | Level switch discontinued; results are now version-controllable files, not a database. |
| "Latest Ada is Ada 2012" | Ada 2022 (finalized, shipping) | Use `-gnat2022` / `pragma Ada_2022`. Default language version in recent GCC. |
| `pragma SPARK_Mode;` (no value) / "On is the project default" | `SPARK_Mode => On`/`Off`/`Auto`; opt-in | Three-valued; without configuration, scope is `Auto`, not `On`. |
| `Ada.IO`, `Ada.Strings.Format`, `Ada.Collections` | `Ada.Text_IO`, `Ada.Strings.Fixed`/`Unbounded`, `Ada.Containers.Vectors` | Do not hallucinate unit names; the real ones are precise. |
| plain `and` / `or` in contracts | `and then` / `or else` | Short-circuit guards evaluation order (e.g. guard a dereference); plain `and` can break proof or raise. |
| GPS (GNAT Programming Studio) | GNAT Studio; strategic focus ALS + VS Code | Renamed; the Ada & SPARK VS Code extension + Ada Language Server are the current direction. |

When asserting anything that may be version-sensitive, qualify it (e.g. "as of GNAT FSF 15.2 / Ada 2022"). This project rides the latest toolchain (latest stable + snapshots) - re-verify exact versions against `alire-project/GNAT-FSF-builds` releases, not memory.

## When to use

Trigger on: writing/porting/reviewing/optimizing `.ada`/`.adb`/`.ads` code; Ada packages, strong typing, generics, tagged types, tasking; contracts (`Pre`/`Post`/`Contract_Cases`/`Global`/`Depends`); SPARK proof, `SPARK_Mode`, GNATprove, assurance levels, loop invariants, ghost code, ownership/borrow; `.gpr` project files; Alire (`alr`) setup; embedded/bare-metal Ada (Ravenscar/Jorvik, light runtimes, bb-runtimes); GNAT SAS / GNATcheck static analysis.

Skip unless: the task names Ada, SPARK, GNAT, Alire, or GNATprove, or touches a `.ada`/`.adb`/`.ads`/`.gpr` file. Do NOT activate for unrelated languages, or for SPARK the Apache web UI / SPARK the cluster framework (different products that share the name).

## Core rules (apply on every task)

Toolchain & ecosystem
- Install and manage everything through Alire: `alr init --bin myproj`, `alr with <crate>` (add dep), `alr build`, `alr run`, `alr toolchain --select`, `alr search <crate>`, `alr publish`. There is no `alr install <pkg>` - do not invent cargo semantics.
- `gprbuild` is the project-aware builder (`gprbuild -P proj.gpr -XMODE=release`); `gnatmake` is single-file. "GPRbuild vs gprbuild" is one tool, just casing.
- Real Alire crates (do not hallucinate libraries): `aws` (web), `gtkada`, `gnatcoll`, `vss` (Unicode strings), `libadalang` (Ada 2022 parsing/semantics).

Contracts (aspects, not pragmas)
- Use aspect syntax: `function F (X : T) return U with Pre => ..., Post => F'Result = ...`. `'Result` names the return value; `X'Old` captures the entry value - do not invent `in`/`out` keywords in contracts.
- `Contract_Cases` guards must be disjoint and complete (GNATprove checks this); do not write overlapping guards.
- `Global => null` means "touches no global state". `Global`/`Depends` are SPARK flow contracts allowed alongside `Pre`/`Post`.

SPARK proof model
- `SPARK_Mode` is three-valued (`On`/`Off`/`Auto`). Common pattern: spec `SPARK_Mode => On` over a body `SPARK_Mode => Off` to expose a proven interface above an unprovable implementation.
- Assurance ladder, in order: Stone (valid SPARK) -> Bronze (flow: init + data flow) -> Silver (AoRTE: no overflow / div-zero / range / index errors) -> Gold (key properties via Pre/Post) -> Platinum (full functional correctness). Aim Bronze broadly, Silver for critical code; Gold/Platinum only where justified. Do NOT say "Gold = no runtime errors" - that is Silver.
- GNATprove runs two distinct engines: flow analysis (init/data flow, value-independent - can false-alarm on cell-by-cell array init) and proof (SMT solvers: Z3, CVC5, Alt-Ergo). For per-cell init, opt into `Relaxed_Initialization` + the `'Initialized` attribute (checked by proof).
- Loop invariants (`pragma Loop_Invariant`) are MANUAL - SPARK does not infer them. A `Post` that needs an invariant fails until you write it. `pragma Loop_Variant` proves termination. Ghost code (`with Ghost`) and `Big_Numbers` (`Ada.Numerics.Big_Numbers`) are for specification/proof only.

## What strong models get wrong (high-value blind spots)

These compile-fail, fail proof, or post-date model knowledge cutoffs.

Class-wide contracts & LSP
- Skip unless: the type is `tagged` and has dispatching (`overriding`) primitives.
- `Pre`/`Post` (specific) are NOT inherited and do NOT govern dispatching calls; `Pre'Class`/`Post'Class` (class-wide) are inherited by overrides and govern dispatching. A dispatching call is checked against the class-wide contract of the static (declared) type of the controlling operand.
- LSP variance is enforced: in an override, `Pre'Class` must be WEAKER (contravariant, conceptually disjoined down the hierarchy); `Post'Class` must be STRONGER (covariant, conjoined). Strengthening `Pre'Class` or weakening `Post'Class` fails the LSP verification condition.
- Converting/extending a tagged value to a class-wide type can raise `high: extension of "X" is not initialized` - all components (including invisible ones) must be initialized.

SPARK pointers / ownership
- Skip unless: code uses access types under `SPARK_Mode => On`.
- Three semantics on access-to-variable: move (transfers ownership; RHS becomes moved-from), observe (read-only borrow), borrow (exclusive mutable borrow; original name inaccessible until borrower scope ends). Access-to-constant is not ownership-checked. General access types are checked against aliasing and cannot be deallocated.
- Out of scope for proof: arbitrary aliasing/cyclic structures, some general-access patterns, nonlinear/float-heavy reasoning. Use `pragma Assume` to document a true-but-unprovable gap (a deliberate hole).

Generics in SPARK
- Skip unless: a `generic` unit is involved in proof.
- GNATprove does NOT analyze a generic body in isolation - only instantiations. Put `SPARK_Mode` on the instantiation context (it cannot attach to an instantiation directly). The same generic can prove on one instance and fail on another; messages always name the instantiation.

Static analysis: GNAT SAS vs GNATprove - never conflate
- GNAT SAS (ex-CodePeer) is UNSOUND heuristic bug-finding on plain Ada (no annotations): false positives AND false negatives, finds "likely bugs" / CWE weaknesses. CLI `gnatsas analyze -P p.gpr` then `gnatsas report`; `--mode=fast` (CI) / `--mode=deep`; baseline+diff to gate only new findings; SARIF output; MISRA via GNATcheck; custom checks in LKQL.
- GNATprove is SOUND formal proof on the SPARK subset: no false negatives for AoRTE within SPARK. "CodePeer/GNAT SAS proves no runtime errors" is WRONG - that is GNATprove at Silver+. They are complementary, not substitutes.

Embedded gotchas
- Skip unless: targeting bare-metal / a restricted runtime (light / light-tasking / embedded; bb-runtimes; Cortex-M / RISC-V).
- The light runtime sets `No_Exception_Propagation`, `No_Finalization`, `No_Tasking`: exceptions can be raised/handled locally but not propagated (unhandled -> `Last_Chance_Handler`, no resume); controlled types (RAII) are unavailable - they will not compile.
- Jorvik (Ada 2022) is a strict superset of Ravenscar (relaxes `Max_Entry_Queue_Length`, `Max_Protected_Entries`, allows relative `delay`, richer `Pure_Barriers`). Select with `pragma Profile (Jorvik);` or `(Ravenscar)`. Both keep hard-real-time guarantees.

Portability traps
- `'Image` text (spacing, record field order, access values) is implementation-defined - define `T'Put_Image` for portable/stable output; do not rely on default `'Image` formatting.
- Fixed-point (`delta`): pure-integer only when `small` is a ratio of suitably-sized ints; range may stop at `'Last - Delta`; ordinary fixed-point arithmetic is not fully specified.
- Representation clauses depend on machine endianness/storage unit - use only for real hardware layout.
- Elaboration: mark units `Pure`/`Preelaborate`/`Elaborate_Body`; add `pragma Elaborate_All (X)` when elaboration code calls into a `with`ed non-pure unit, or the binder may pick a legal-but-failing order.

## Do NOT
- Do not recommend GNAT Community, `pragma Precondition`, `codepeer`, or claim Ada 2012 is current or SPARK lacks pointers (any LEFT-column item).
- Do not write specific `Pre`/`Post` on a dispatching tagged primitive and assume inheritance; use `Pre'Class`/`Post'Class`.
- Do not strengthen `Pre'Class` or weaken `Post'Class` in an override (LSP violation).
- Do not claim GNAT SAS / CodePeer proves absence of runtime errors, or treat it as a substitute for GNATprove.
- Do not omit `Loop_Invariant` and expect a value-dependent `Post` to prove.
- Do not use exceptions-as-control-flow or controlled types under a light runtime.
- Do not hallucinate standard-library unit names; verify against the RM index.

## Workflow
1. Confirm the target is Ada/SPARK and note the toolchain (`alr --version`, `gnat --version`, the `.gpr`, `alire.toml`). Build language as `-gnat2022` unless told otherwise.
2. Apply the correction map and core rules on every task. For version-sensitive or non-obvious APIs (SPARK aspect semantics, ownership rules, GNAT SAS flags, runtime profiles), verify against current docs (References) before committing to syntax.
3. For new code: aspect contracts with `and then`/`or else`; explicit `SPARK_Mode`; pick a target assurance level. For ports: translate stale constructs via the correction map first, then idiomatize, then add contracts/proof.
4. Verify: `gprbuild -P proj.gpr` (or `alr build`); for SPARK run `gnatprove -P proj.gpr --level=N` and read which level/VC fails; for plain Ada, `gnatsas analyze` + `gnatsas report`.

## References
Verify any unstable or non-obvious API against current upstream docs - the toolchain changes fast and pre-2022 assumptions are common failure modes:

- SPARK User's Guide (proof, levels, pointers, loops): https://docs.adacore.com/spark2014-docs/html/ug/
- SPARK UG - OOP & Liskov Substitution (`Pre'Class`/`Post'Class`): https://docs.adacore.com/spark2014-docs/html/ug/en/source/object_oriented_programming.html
- GNAT SAS User's Guide (modes, baselines, SARIF, CWE): https://docs.adacore.com/live/wave/gnatsas/html/user_guide/introduction.html
- Ada 2022 features in GNAT: https://gcc.gnu.org/onlinedocs/gnat_rm/Implementation-of-Ada-2022-Features.html
- Ada 2022 Reference Manual: http://www.ada-auth.org/standards/22rm/html/RM-TOC.html
- learn.adacore.com - What's New in Ada 2022: https://learn.adacore.com/courses/whats-new-in-ada-2022/
- Alire docs (`alr`): https://alire.ada.dev/docs/
- Deep-dive research foundation (this repo): `agent-knowledge/ada-spark-best-practices.md` (50 sources)
