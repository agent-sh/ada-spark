# Toolchain

Read this when setting up, building, proving or analyzing a project. Checked against the Alire user-changes log (through 2.1), the GNAT User's Guide, the GNAT SAS User's Guide and the `alire-project` repositories on 2026-09-24.

## Alire

- New project: `alr init --bin name` (or `--lib`). Dependencies: `alr with crate`, `alr with --del crate`. Build and run: `alr build`, `alr run`. Any tool in the crate's environment: `alr exec -- <command>`.
- Toolchains: `alr toolchain --select` chooses the compiler and `gprbuild` Alire uses. `alr install gnat_native gprbuild` (Alire 2.0+) installs binaries to the Alire bin directory in your home (`.alire/bin`) for use outside Alire; `alr toolchain --install` is deprecated in favor of it.
- Crates: `alr search <word>` before naming a library. Well-established ones include `aws` (web server), `gnatcoll`, `vss` (Unicode text), `libadalang`, `gtkada`, `gnatformat`.
- Publishing: `alr publish`.

## Versions

Check current releases instead of quoting from memory:

- GNAT FSF compilers and `gnatprove`: GitHub releases of `alire-project/GNAT-FSF-builds` (tags `gnat-<ver>-<n>`, `gnatprove-<ver>-<n>`, plus `-snapshot` builds).
- Alire itself: `alire-project/alire` releases.
- The crate index: `alr show gnat_native`, `alr show gnatprove`.

GNAT's default language mode is Ada 2012; use `-gnat2022` in the project's compiler switches (or `pragma Ada_2022`) for Ada 2022 code.

## GNATprove through Alire

`alr with gnatprove` adds it to the crate's environment; run `alr exec -- gnatprove -P <project>.gpr --level=2` (raise the level when proofs time out). Proof results and session data go under the project's object directory (`gnatprove/`).

## GNAT SAS versus GNATprove

- GNAT SAS (formerly CodePeer) runs several engines (Inspector, Infer, GNAT warnings, GNATcheck) over plain Ada without annotations. It is heuristic: useful for finding likely bugs and CWE-class weaknesses, with both false positives and misses. `gnatsas analyze -P p.gpr` then `gnatsas report -P p.gpr`; `--mode=fast` (default) or `--mode=deep`; results live in `.sam` message files and `.sar` review files, can be compared to a baseline, and export to text, CSV, SARIF, Code Climate or HTML. It is distributed to AdaCore customers, not through Alire.
- GNATprove is sound for SPARK code: when it reports no unproved checks at Silver, those runtime errors cannot occur (under the tool's documented assumptions).
- Use GNAT SAS where code is not in SPARK, GNATprove where it is; do not describe one as the other.

## Builders and editors

`gprbuild -P proj.gpr -XMODE=release` builds a project file; `gnatmake` handles single files. GNAT Studio (formerly GPS) and VS Code with AdaCore's Ada & SPARK extension (Ada Language Server) are the current editors.
