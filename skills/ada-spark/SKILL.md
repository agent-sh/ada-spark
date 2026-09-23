---
name: ada-spark
description: Use when writing, porting, reviewing or proving Ada or SPARK code (.adb, .ads, .gpr, alire.toml), including contracts, GNATprove, Alire projects and embedded runtimes. Not for Apache Spark or other languages.
allowed-tools: Read, Edit, Write, Grep, Glob, Bash(alr:*), Bash(gnatmake:*), Bash(gprbuild:*), Bash(gnat:*), Bash(gnatprove:*), Bash(gnatsas:*), Bash(gnatformat:*)
---

# Ada and SPARK

Write Ada and SPARK that builds on the current open toolchain, proves where proof is the goal, and does not repeat advice that stopped being true years ago. Docs: `learn.adacore.com`, `docs.adacore.com` (SPARK User's Guide, GNAT SAS), the Ada 2022 RM at `ada-auth.org`, and `alire.ada.dev`.

Most Ada in training data predates 2022, and the ecosystem moved: GNAT Community ended, Alire became the way to get compilers and libraries, SPARK gained pointer ownership, CodePeer became GNAT SAS. The table below is the core of this skill; the references hold the detail.

## Check the toolchain first

Look at `alire.toml`, the `.gpr` file and `alr toolchain` (or `gnat --version`, `gnatprove --version`) before writing code. Pass `-gnat2022` (or `pragma Ada_2022`) when you use Ada 2022 features, because GNAT still defaults to Ada 2012. Versions move several times a year; check `alire-project/GNAT-FSF-builds` releases rather than quoting a version from memory, and say which one you targeted.

## Old to current

| Old | Current | Notes |
|---|---|---|
| "Install GNAT Community Edition" | Alire (`alr`) with GNAT FSF toolchains | GNAT Community's last release was 2021. `alr toolchain --select` picks the compiler for Alire projects; `alr install gnat_native gprbuild` puts tools on your PATH for use outside Alire. |
| `pragma Precondition` / `pragma Postcondition` | aspects: `with Pre => ..., Post => ...` | The pragmas are GNAT-specific; aspects are standard Ada 2012+. |
| specific `Pre`/`Post` as the contract of a dispatching operation | `Pre'Class` / `Post'Class` | In Ada a specific `Pre` is legal on a tagged primitive (not on abstract subprograms or null procedures) but is not inherited. SPARK rejects a plain `Pre` on a dispatching subprogram outright (SPARK RM 6.1.1(2)); a specific `Post` next to `Post'Class` is allowed. `Pre'Class` on an override is only legal if an ancestor declared one (RM 6.1.1). |
| "SPARK can't use pointers" | ownership: move, observe, borrow | Access-to-variable types follow a Rust-like ownership model under GNATprove. |
| `codepeer`, `--level 0..4`, results database | `gnatsas analyze` / `gnatsas report`, `--mode=fast` (default) or `deep`, `.sam` / `.sar` files | GNAT SAS ships to AdaCore customers (through GNAT Tracker), not through Alire. |
| "Ada 2012 is the latest" | Ada 2022 | Declare expressions, delta aggregates, `@`, `'Reduce`, `'Image` for all types, `Put_Image`, bracket aggregates, the Jorvik profile. |
| `SPARK_Mode => Auto` as an aspect | `SPARK_Mode` aspect takes `On` or `Off`; `Auto` exists only as a configuration pragma | `pragma SPARK_Mode;` or `with SPARK_Mode` alone means `On`. Without a configuration pragma, only code marked `On` is analyzed. |
| invented units such as `Ada.IO`, `Ada.Strings.Format`, `Ada.Collections` | `Ada.Text_IO`, `Ada.Strings.Fixed` / `Unbounded`, `Ada.Containers.Vectors` / `Hashed_Maps` / ... | Check the RM annex index before naming a unit. |
| `and` / `or` in contract guards | `and then` / `or else` | Short-circuit keeps a guard such as `X /= null and then X.all > 0` from failing; plain `and` evaluates both sides. |
| GPS | GNAT Studio, or VS Code with the Ada & SPARK extension (Ada Language Server) | |

## Rules that hold across tasks

- Alire is the project manager: `alr init --bin|--lib`, `alr with <crate>`, `alr build`, `alr run`, `alr exec -- <cmd>`, `alr search`, `alr publish`. `gprbuild -P proj.gpr` is the project builder outside Alire. Real crates include `aws`, `gnatcoll`, `vss`, `libadalang`, `gtkada`; search before naming one.
- Contracts use aspects. `F'Result` names a function's result, `X'Old` the value on entry. `Contract_Cases` guards must be disjoint and complete, which GNATprove checks.
- SPARK is a subset of Ada plus contracts. Code that compiles can still be rejected or unprovable; `SPARK_Mode => On` on a spec with `Off` on its body is the usual way to prove callers against an interface whose implementation is outside SPARK.
- Proof needs what the tool cannot infer: loop invariants (`pragma Loop_Invariant`) for any property that must survive a loop, `Loop_Variant` for termination, and enough postcondition on callees for the caller to prove its own.
- GNAT SAS finds likely bugs heuristically (it can miss some and flag false ones); GNATprove proves properties soundly for SPARK code. Neither substitutes for the other, and absence of runtime errors is a GNATprove result (Silver level), not a GNAT SAS one.
- On light runtimes, exceptions cannot propagate out of a subprogram and controlled types (finalization) and tasking are unavailable, so code relying on them will not build there. Light-tasking and embedded runtimes add back Ravenscar or Jorvik tasking and more of the library.

## References

Read the one that matches the task:

- `references/spark-proof.md`: assurance levels, flow versus proof, initialization, loops, ghost code, ownership, generics, class-wide contracts and LSP.
- `references/toolchain.md`: Alire workflows, running GNATprove, GNAT SAS versus GNATprove, finding current versions.
- `references/embedded-and-portability.md`: runtimes, Ravenscar and Jorvik, `'Image` and `Put_Image`, fixed point, representation clauses, elaboration.

`agent-knowledge/ada-spark-best-practices.md` in the repository is the longer sourced research this skill was built from; it is not installed with the skill.

## Done

The code builds with the project's toolchain (`alr build` or `gprbuild`), SPARK code reaches the assurance level the task asked for with `gnatprove` (and you say which checks remain unproved and why), no construct from the left column remains, and any version-specific claim you could not check is named as unverified.
