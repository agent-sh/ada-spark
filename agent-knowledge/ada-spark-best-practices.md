# Learning Guide: Ada & SPARK Best Practices, Gotchas, and LLM Blind Spots

**Generated**: 2026-05-21
**Sources**: 50 resources analyzed (AdaCore official docs/blog, Ada Reference Manual 2022, SPARK User's & Reference Guides, GNAT SAS docs, learn.adacore.com, Alire docs, Ada Forum, GitHub)
**Depth**: deep
**Primary purpose**: Capture knowledge that state-of-the-art LLMs (GPT-5.5, Opus 4.7, etc.) typically get WRONG or are UNAWARE of due to knowledge cutoffs or material rarity.

---

## How To Use This Guide

This is not a from-scratch Ada tutorial. It is a correction layer over what an LLM "thinks it knows" about Ada/SPARK. Each section flags **[LLM TRAP]** for things models commonly hallucinate or state with stale confidence, and **[CURRENT]** for facts that postdate typical training cutoffs.

**This project rides the latest toolchain** — latest stable line *and* bleeding snapshots, not pinned/old versions. Always assume the newest available; accept snapshot/dev builds for newest features. The versions below are the latest snapshot as of 2026-05-21 — they move fast, so re-verify against `alire-project/GNAT-FSF-builds` releases each session rather than treating them as fixed.

- **Compiler**: track newest. Latest *stable* `gnat_native` via Alire is **15.2.1** (GCC 15.2.0; `GNAT-FSF-builds` tag `gnat-15.2.0-1`, 2025-08-28). Newest snapshot is **`gnat-16.0.1-snapshot`** (2026-05-19) — use it for latest features. Upstream **GCC 16.1 is released**, 17.0 is dev. GCC 15+ carries finalized Ada 2022. *(Verified against GitHub `alire-project/GNAT-FSF-builds` releases + `alire-index` + `gcc.gnu.org` tags, 2026-05-21.)*
- **Builder**: `gprbuild` **26.0.0-1** (2026-05-07) — latest.
- **SPARK / GNATprove**: stable **15.1.0**; track snapshot **`16.0.0-snapshot`** (2026-05) for newest proof features. SPARK 22+ uses the simplified memory model.
- **Package manager**: Alire (`alr`) 2.x is the de facto standard; `alr toolchain --select` to pull newest crates.
- **GNAT Pro** (commercial, only if licensed): 2026 cycle is **26.x**; exact point releases / internal "wave" dev tag are not publicly verifiable — do not cite a precise Pro version authoritatively.

---

## Prerequisites

- Basic Ada syntax (packages, subprograms, types). This guide assumes you can read Ada.
- A working `alr` install (see Ecosystem section). Do NOT assume "GNAT Community" exists — it is gone (see below).
- For SPARK: conceptual familiarity with Hoare-logic style pre/postconditions.

---

## TL;DR — The Highest-Value Corrections

- **[LLM TRAP] "Download GNAT Community Edition" is wrong.** GNAT Community was discontinued; the **2021 release was the last one**. The replacement for open-source users is **Alire (`alr`)** plus **GNAT FSF** builds. Telling a user to grab GNAT Community 2020/2021 is the single most common stale instruction.
- **[LLM TRAP] Prefer aspects over pragmas for contracts.** `with Pre => ..., Post => ...` (aspect syntax, Ada 2012+) is idiomatic. The old `pragma Precondition`/`pragma Postcondition` GNAT pragmas still compile but are not the modern style and are not the portable contract mechanism.
- **[LLM TRAP] Ada and SPARK are not the same subset.** SPARK is a *subset* of Ada plus *annotations*. Code that compiles as Ada may be rejected by GNATprove (e.g., uncontrolled aliasing, exception handlers in proven code, certain access patterns).
- **[LLM TRAP] `Pre` and `Pre'Class` are different contracts.** On a dispatching primitive of a tagged type, plain `Pre` is *not* inherited by overrides and is *not* what a dispatching call checks; you must use `Pre'Class`. See the class-wide contracts section.
- **[CURRENT] SPARK has a Rust-like ownership/borrow model for pointers since ~2019/SPARK 20–22.** "SPARK can't do pointers" is outdated. It can, via move/observe/borrow semantics on access types.
- **[CURRENT] CodePeer is now GNAT SAS (GNAT Static Analysis Suite).** The bundle was renamed and the CLI is now `gnatsas`, not `codepeer`. It is *unsound heuristic* bug-finding, NOT formal proof — do not conflate it with GNATprove.
- **[CURRENT] Ada 2022 is finalized and shipping** (string interpolation, `'Image` for any type, `@` target name, `'Reduce`, declare expressions, square-bracket array aggregates, parallel loops, the **Jorvik** profile). Use `-gnat2022`. GNAT 2022 is the default language version in recent GCC.
- **[LLM TRAP] Stop hallucinating standard library units.** There is no `Ada.Strings.Format`, no `Ada.IO`, no generic `Ada.Collections`. The real names are precise (`Ada.Text_IO`, `Ada.Strings.Unbounded`, `Ada.Containers.Vectors`, etc.).

---

## Core Concepts

### 1. The Toolchain & Ecosystem Reality (where LLMs are most stale)

#### GNAT Community is dead; Alire is the way [CURRENT]
AdaCore announced the end of the GNAT Community release line. The **2021 release was the final GNAT Community**. The ecosystem split cleanly into two:
- **GNAT Pro** — provided and supported by AdaCore for commercial/industrial projects.
- **GNAT FSF** — built by the community from FSF GCC releases, with familiar (GCC-runtime-exception) licensing rather than pure GPL runtimes.

Everything GNAT Community offered (GNAT Studio, SPARK, native + cross ARM + cross RISC-V compilers) is now obtained through **Alire**.
(Source: AdaCore blog "A New Era For Ada/SPARK Open Source Community"; alire.ada.dev transition page.)

#### Alire (`alr`) — the de facto package manager [CURRENT]
- Role analogous to Rust's `cargo` / OCaml's `opam`. Catalog at **alire.ada.dev**; community index lives on GitHub (`alire-project`).
- Ecosystem size: **595 crates / 27,455 tests as of 3 Nov 2025** — it is real and active, not a toy.
- `alr 2.x` added shared dependency caches (fewer rebuilds), binary toolchain installation, and a streamlined publish flow vs `1.x`.
- Toolchain crates you select with `alr toolchain --select`: `gnat_native` (stable **15.2.1**), `gprbuild` (FSF build **26.0.0-1**, 2026-05-07 — the 25.x line was 2025), `gnatprove` (stable 15.1.0; `16.0.0-snapshot` exists 2026-05).
- GNAT FSF binaries are produced by the `alire-project/GNAT-FSF-builds` repo (tags like `gnat-15.2.0-1` for the stable line, `gnat-16.0.1-snapshot` for dev).

**[LLM TRAP]** Do not invent `alr install <pkg>` semantics from cargo. Common real commands: `alr init --bin myproj`, `alr with <crate>` (add dependency), `alr build`, `alr run`, `alr toolchain --select`, `alr search <crate>`, `alr publish`.

#### IDEs and editors [CURRENT]
- **GNAT Studio** still exists (versioned like `2026.x`) but AdaCore's strategic direction is the **Ada Language Server (ALS)** + the **Ada & SPARK VS Code extension** (on both VS Marketplace and Open VSX).
- The VS Code extension now defaults to **GNATformat** for formatting (fall back to `gnatpp` by disabling `Ada: Use GNATformat`).
- ALS now runs a **separate LSP instance for GPR files** (`--language-gpr`), giving completion/outline/tooltips/navigation inside `.gpr` project files.
- VS Code GNATprove integration ships predefined tasks + code lenses to prove a subprogram / file / project from the editor.

**[LLM TRAP]** `gprbuild` is the builder binary; `GPRbuild`/`GPRBuild` are just casing of the same project-aware multi-language builder. There is no separate "GPRbuild vs gprbuild" tool — it is one tool. Single-file builds use `gnatmake`; project builds use `gprbuild`.

#### Real, current libraries (don't hallucinate) [CURRENT]
Available as Alire crates: `aws` (Ada Web Server), `gtkada`, `gnatcoll` (+ `gnatcoll_sql`, `gnatcoll_xref`, etc.), `vss` (Virtual String Subsystem — modern Unicode strings), `libadalang` (full Ada 2022 parsing + semantic analysis, used by GNAT SAS and the formatter; bindings in Ada and Python). For SPARK-provable containers, see `ada-traits-containers` / unbounded-containers-in-SPARK work.

---

### 2. Contracts: Aspects, Not Pragmas [LLM TRAP-heavy]

Ada 2012 introduced executable contracts; SPARK builds on them. Use **aspect syntax**:

```ada
function Sqrt (X : Float) return Float
  with Pre  => X >= 0.0,
       Post => Sqrt'Result >= 0.0
              and then abs (Sqrt'Result * Sqrt'Result - X) <= 0.001;
```

Key facts LLMs get wrong:
- **Aspects vs pragmas**: every SPARK aspect has an equivalent pragma so the code still compiles on any Ada implementation, but the **aspect form is idiomatic**. Prefer `with Pre =>` over `pragma Precondition`.
- **`'Result`** is the postcondition's name for the return value. **`X'Old`** captures the entry value of `X`. LLMs frequently omit `'Old` or use a made-up `in`/`out` keyword.
- **`Contract_Cases`** is a SPARK aspect for disjoint case-by-case contracts:
  ```ada
  procedure Update (X : in out Integer)
    with Contract_Cases =>
      (X < 0  => X = X'Old + 1,
       X = 0  => X = 0,
       X > 0  => X = X'Old - 1);
  ```
  The guards must be disjoint and complete; GNATprove checks this. LLMs often write overlapping guards.
- **`Global` and `Depends`** are SPARK aspects (data/flow contracts), allowed in the same places as `Pre`/`Post`. `Global => null` means "touches no global state."
- **Use `and then` / `or else`** in contracts to control evaluation order and avoid evaluating an undefined sub-expression (e.g., guard a dereference). LLMs use plain `and`, which can break proof or raise at runtime.

### 3. SPARK_Mode — three values, applied with care [LLM TRAP]

`SPARK_Mode` is **three-valued**: `On`, `Off`, `Auto`.
- `On` — the construct must be valid SPARK and **will** be analyzed by GNATprove.
- `Off` — not analyzed, need not obey SPARK restrictions (use for code that can't be SPARK, e.g., uses exceptions or unrestricted pointers).
- `Auto` (the default for code without an explicit setting) — not analyzed, GNATprove infers whether it can be used from SPARK code.

You can set it as a configuration pragma (whole project), per package, or split spec/body (common pattern: spec `SPARK_Mode => On`, body `SPARK_Mode => Off` to expose a proven interface over an unprovable implementation).

**[LLM TRAP]** Models often write `pragma SPARK_Mode;` with no value, or assume `On` is default project-wide. It is not — without configuration, analysis scope is `Auto`/opt-in.

### 4. SPARK Assurance Levels (the ladder) [CURRENT, frequently misordered]

Five levels, increasing cost/benefit. The correct order and meaning:

| Level | Name | What it guarantees |
|-------|------|--------------------|
| 1 | **Stone** | Code is valid SPARK (in the subset). No properties proven. |
| 2 | **Bronze** | Stone + correct initialization and data flow (flow analysis). No uninitialized reads, no unintended global access. |
| 3 | **Silver** | Bronze + **Absence of Run-Time Errors (AoRTE)**: no overflow, no division by zero, no index/range/discriminant violations, no infinite loops/recursion. |
| 4 | **Gold** | Silver + proof of **key integrity properties** (selected safety/security invariants via Pre/Post). |
| 5 | **Platinum** | Gold + **full functional correctness**. |

Practical guidance from AdaCore: aim **Bronze broadly**, **Silver** as the default for critical code, **Gold** on the subset with key properties, **Platinum** only for the highest-integrity parts.

**[LLM TRAP]** Models routinely swap Silver/Gold or claim "Gold = no runtime errors." Gold is *properties*; Silver is AoRTE.

### 5. Flow Analysis vs Proof — two distinct engines [LLM TRAP]

GNATprove does two different things:
- **Flow analysis** — checks initialization and data/information flow (`Global`, `Depends`). **Not value-dependent**: it cannot track the value of an array index, so cell-by-cell array init can trigger false alarms.
- **Proof** — discharges verification conditions in first-order logic (overflow, contracts, etc.) using SMT solvers (Z3, CVC5, Alt-Ergo).

The choice between them for initialization hinges on `Relaxed_Initialization`:
- Default strong initialization policy → checked by **flow analysis**.
- `Relaxed_Initialization` aspect → opt out per object; initialization then specified by contracts and checked by **proof**, using the **`'Initialized`** attribute. Useful precisely for the cell-by-cell array case flow analysis can't handle.

### 6. SPARK Pointers, Ownership & the Borrow Checker [CURRENT — major LLM blind spot]

"SPARK doesn't support pointers" is **outdated**. Since SPARK 20–22 there is a Rust-inspired ownership model based on permission/affine-type static alias analysis. Three assignment semantics on access-to-variable types:
- **Move** — assigning a named pool-specific/general access-to-variable pointer **transfers ownership**; the RHS loses the right to access (becomes "moved-from").
- **Observe** — read-only borrow (think `&T`).
- **Borrow** — exclusive mutable borrow (think `&mut T`); the borrowed object is inaccessible through the original name until the borrower goes out of scope.

Other rules:
- Named **access-to-constant** types are not subject to ownership checking.
- **General access** types are checked to prevent aliasing and **cannot be deallocated** (might point to the stack).
- SPARK 22+ uses a simpler memory model where allocators and `Ada.Unchecked_Deallocation` no longer read/write a special abstract state.

**Not yet (easily) provable / out of scope**: arbitrary aliasing/cyclic structures, certain general-access patterns, and properties needing nonlinear arithmetic or floating-point reasoning that SMT solvers choke on. When a property is true but unprovable, `pragma Assume` documents the gap (and is a deliberate hole in the proof).

### 7. Loop Invariants & Ghost Code [CURRENT]

- **Loop invariants** (`pragma Loop_Invariant`) are **manual** assertions stating exactly what holds at each iteration. SPARK does *not* infer them. Common LLM failure: writing a `Post` that needs an invariant, then omitting the invariant and being surprised proof fails.
- **`pragma Loop_Variant`** proves loop termination (a value that strictly de/increases).
- **Ghost code** (`with Ghost`) — variables, functions, even whole packages used only for specification/proof; compiled out of the final executable (unless you enable execution for testing). Functional containers (`Ada.Containers.Functional_*`) are designed for ghost use.
- **`Big_Numbers`** (`Ada.Numerics.Big_Numbers.Big_Integers` / `Big_Reals`) — unbounded integers/rationals, heavily used in ghost specs (e.g., AdaCore proved 255-bit Curve25519 arithmetic by reasoning over Big_Integers in the spec).

### 8. Class-Wide Contracts (`Pre'Class`/`Post'Class`) & Liskov Substitution [LLM TRAP-heavy]

This is where LLMs reliably confuse two *different* contracts that look almost identical. For a tagged type with dispatching (`overriding`) operations, `Pre`/`Post` (**specific**) and `Pre'Class`/`Post'Class` (**class-wide**) are **not** the same and are **not** interchangeable.

| Aspect | Applies to | Inherited by overrides? | Drives a *dispatching* call? | Drives a *static* (non-dispatching) call? |
|---|---|---|---|---|
| `Pre` / `Post` (specific) | the one body it's written on | **No** | **No** | Yes |
| `Pre'Class` / `Post'Class` (class-wide) | the operation *and all overrides* | **Yes** (inherited down the hierarchy) | **Yes** | contributes (specific must be compatible) |

**[LLM TRAP] Plain `Pre` is NOT inherited and is NOT what a dispatching call checks.** Models routinely write `Pre`/`Post` on a tagged primitive and assume an override inherits it, or that a dispatching call through a class-wide reference enforces it. It does not. Per the SPARK UG, the contract that applies in proof to a dispatching call is the first present of: (1) the `Pre'Class`/`Post'Class` attached to the subprogram, or otherwise (2) the `Pre'Class`/`Post'Class` **inherited** from the operation it overrides. Specific contracts are used only for **static** calls.

**[LLM TRAP] `Pre` is forbidden on a primitive at a point where the type is tagged** — you must use `Pre'Class` there (Ada RM 6.1.1; carried into the SPARK subset). A model emitting `with Pre => ...` on a dispatching primitive of a tagged type is writing something the language rules reject in that position. (A specific `Post` *is* permitted alongside `Post'Class`, subject to the compatibility VC below.)

**Liskov Substitution Principle (LSP) — the variance rule SPARK enforces.** An override must be safely substitutable for the parent operation, so the two contracts vary in **opposite directions** going down the hierarchy:
- **`Pre'Class` is weakened (contravariant).** An override's class-wide precondition must be **weaker (more permissive)** than the overridden one — every caller that satisfied the parent's precondition must still be accepted. Conceptually the effective dispatching precondition is the **disjunction** of applicable class-wide preconditions.
- **`Post'Class` is strengthened (covariant).** An override's class-wide postcondition must be **stronger (more restrictive)** — it must *imply* the overridden one. Conceptually the effective dispatching postcondition is the **conjunction** of applicable class-wide postconditions.

GNATprove checks these as verification conditions on every override, emitting `info: class-wide precondition is weaker than overridden one` and `info: class-wide postcondition is stronger than overridden one`. Mixing rules: a specific `Post` (if present) must be **stronger than** (imply) the `Post'Class`; a specific `Pre` must be **weaker than** the `Pre'Class`. When only a class-wide contract is given and no specific one, the class-wide contract *also serves* as the specific contract.

```ada
package Logging with SPARK_Mode => On is
   type Log_Type is tagged private;
   Max_Count : constant := 1000;

   function Log_Size (L : Log_Type) return Natural;

   --  Class-wide contract: inherited by every override; governs dispatching calls.
   --  (Use Pre'Class, NOT Pre, on a dispatching primitive of a tagged type.)
   procedure Append_To_Log (L : in out Log_Type; Incr : Integer)
     with Pre'Class  => Log_Size (L) < Max_Count,
          Post'Class => Log_Size (L) = Log_Size (L)'Old + 1;
private
   --  ...
end Logging;

package Logging.Range_Logging with SPARK_Mode => On is
   type Log_Type is new Logging.Log_Type with private;

   function Get_Max (L : Log_Type) return Integer;

   overriding
   procedure Append_To_Log (L : in out Log_Type; Incr : Integer)
     --  Pre'Class inherited (weaker-or-equal -> LSP precondition check passes).
     --  Post'Class STRONGER: keeps the parent guarantee AND adds max tracking.
     with Post'Class => Log_Size (L) = Log_Size (L)'Old + 1
                        and then Get_Max (L) >= Get_Max (L'Old);
private
   --  ...
end Logging.Range_Logging;
```

**Why SPARK proves dispatching calls without whole-program knowledge:** because LSP is *enforced*, the class-wide contract of any override is guaranteed no stricter than the parent's. So GNATprove analyzes a dispatching call using **only the class-wide contract of the static (declared) type of the controlling operand** — it never enumerates the runtime targets. This is what makes modular, separate proof of OOP code tractable.

**[LLM TRAP] Redispatch and extension initialization.** Specific contracts are required on operations of tagged types so GNATprove can check LSP per override. Subtle flow check: converting a specific tagged value to a class-wide type (or extending it) can trigger `high: extension of "X" is not initialized` if hidden extension components are not initialized — a class-wide object must have *all* components (including invisible ones) initialized. Models porting OOP idioms from other languages miss this.

### 9. Generics in SPARK — instances are analyzed, not the generic body [LLM TRAP]

**[LLM TRAP] GNATprove does NOT analyze a generic unit's body in isolation.** Per the SPARK UG: *"GNATprove does not directly analyze the code of generics."* Only **instantiations** are analyzed. Consequences models get wrong:
- `SPARK_Mode => On` on a generic means instances *can* be in SPARK — it depends on the **actual parameters** at instantiation, not a property of the generic alone. The advice is to put `SPARK_Mode` on the *non-generic* context (the instantiation site / enclosing declaration), since the aspect cannot attach directly to an instantiation in Ada syntax.
- **SPARK-rule violations in a generic may not surface until instantiation.** A generic body can look fine yet yield unprovable/illegal-SPARK results for a particular instance.
- The same generic can prove on one instance and **fail on another** ("some checks are proved on an instance and not on another one"). GNATprove reports messages on the generic code but **always states which instantiation** the message belongs to.
- You may put contracts on generic formal subprograms, but verification is driven by the concrete instance's actuals — do not assume the generic body is verified abstractly over all possible actuals.

---

## Code Examples

### Modern Ada 2022 syntax LLMs underuse [CURRENT]

```ada
pragma Ada_2022;  -- or build with -gnat2022

--  Target name '@' (no more repeating the LHS)
Total := @ + Item;            --  instead of Total := Total + Item;

--  Square-bracket array aggregates + iterated component association
A : array (1 .. 5) of Integer := [for I in 1 .. 5 => I * I];

--  Delta aggregate (copy with changes)
P2 : Point := (P1 with delta X => 0);

--  'Image on ANY type (Ada 2022); prefer Value'Image form
Put_Line (Some_Record'Image);   --  formatting is GNAT-specific, see gotcha

--  'Reduce expression
Sum : constant Integer := [for X of A => X]'Reduce ("+", 0);

--  String interpolation (Ada 2022, GNAT)
Name : constant String := "Ada";
Put_Line (f"Hello, {Name}!");
```

### Correct contract + proof skeleton (Silver target)

```ada
package Stack with SPARK_Mode => On is
   Max : constant := 100;
   type Stack_Type is private;

   function Is_Full (S : Stack_Type) return Boolean;
   function Is_Empty (S : Stack_Type) return Boolean;

   procedure Push (S : in out Stack_Type; V : Integer)
     with Pre  => not Is_Full (S),
          Post => not Is_Empty (S);
private
   type Index is range 0 .. Max;
   type Data_Array is array (1 .. Max) of Integer;
   type Stack_Type is record
      Top  : Index := 0;
      Data : Data_Array := [others => 0];
   end record;
   function Is_Full  (S : Stack_Type) return Boolean is (S.Top = Max);
   function Is_Empty (S : Stack_Type) return Boolean is (S.Top = 0);
end Stack;
```

### A minimal, idiomatic `.gpr` (LLMs often get attributes wrong)

```ada
project My_Proj is
   type Mode_Type is ("debug", "release");
   Mode : Mode_Type := external ("MODE", "debug");

   for Source_Dirs use ("src", "src/" & Mode);
   for Object_Dir  use "obj/" & Mode;   --  must already exist? GPRbuild creates it
   for Main        use ("main.adb");

   package Compiler is
      case Mode is
         when "debug"   => for Switches ("Ada") use ("-g", "-gnata", "-gnat2022");
         when "release" => for Switches ("Ada") use ("-O2", "-gnat2022");
      end case;
   end Compiler;
end My_Proj;
```

`.gpr` notes: it is Ada-like syntax; source dirs are read-only to most tools; generated files go to `Object_Dir`; scenario variables come from `external (...)`; build with `gprbuild -P my_proj.gpr -XMODE=release`.

---

## Common Pitfalls (LLM-prone)

| Pitfall | Why It Happens | How to Avoid |
|---------|---------------|--------------|
| Recommending "GNAT Community 2020/2021" | Stale training data | Use Alire + GNAT FSF; Community ended after 2021 |
| `pragma Precondition`/`Postcondition` for contracts | Old GNAT pragmas in old docs | Use aspects: `with Pre =>`, `with Post =>` |
| Treating SPARK == Ada | Conflation | SPARK is a subset + annotations; verify with `SPARK_Mode => On` |
| "SPARK can't do pointers" | Pre-2019 knowledge | Ownership/borrow model since SPARK 20–22 |
| Plain `and`/`or` in contracts | C-style habit | Use `and then`/`or else` to guard evaluation |
| Writing `Pre`/`Post` on a dispatching tagged primitive and expecting inheritance | Treating tagged like flat subprograms | Use `Pre'Class`/`Post'Class`; specific `Pre` is illegal on a tagged primitive and is never inherited for dispatch |
| Strengthening `Pre'Class` (or weakening `Post'Class`) in an override | Forgetting LSP variance | `Pre'Class` weakens down, `Post'Class` strengthens down |
| Expecting GNATprove to verify a generic body abstractly | Assuming whole-generic analysis | Only instances are analyzed; put `SPARK_Mode` on the instantiation context |
| Conflating CodePeer/GNAT SAS with GNATprove | Both are "SPARK/AdaCore static tools" | GNAT SAS is unsound heuristic bug-finding; GNATprove is sound proof of AoRTE |
| Calling the static analyzer `codepeer` on the CLI | Stale name | Binary is now `gnatsas` (CodePeer was renamed to GNAT SAS) |
| Omitting `Loop_Invariant` then expecting `Post` proof | Assuming inference | Invariants are manual; state them explicitly |
| Relying on `'Image` output format | Assuming portability | Format is implementation-defined; define `Put_Image` for portable output |
| Ignoring elaboration order | Hidden init-time calls | Mark units `Pure`/`Preelaborate`/`Elaborate_Body`; add `pragma Elaborate_All` when elaboration code calls into a `with`ed unit |
| Hallucinating units (`Ada.IO`, `Ada.Strings.Format`, `Ada.Collections`) | Pattern-matching other languages | Real names: `Ada.Text_IO`, `Ada.Strings.Fixed/Unbounded`, `Ada.Containers.Vectors` |
| Using exceptions in light-runtime embedded code | Desktop assumptions | Light runtime sets `No_Exception_Propagation`; unhandled → `Last_Chance_Handler`, no resume |
| `delta`/fixed-point assumed portable in arithmetic | Treating like float | Ordinary fixed-point ops not fully specified; range may stop at `'Last - Delta`; small-as-ratio-of-ints keeps it integer-only |
| Representation clauses ignoring endianness | Assuming layout is portable | Rep-clause semantics depend on machine endianness/storage unit; use only for real hardware layout |
| Confusing Ravenscar with Jorvik | Jorvik is new (Ada 2022) | See profiles section; Jorvik relaxes several Ravenscar restrictions |

---

## Gotchas In Depth

### Elaboration order
`pragma Elaborate_All (X)` guarantees the spec **and** body of X **and** everything X `with`s (transitively) elaborate before the current unit — stronger than `pragma Elaborate`. Rule of thumb: mark units `Pure` or `Preelaborate`; if not possible, `Elaborate_Body`; and if your elaboration code can call a subprogram or instantiate a generic from a `with`ed non-pure unit, add `Elaborate_All` for it. Otherwise the binder may pick a legal-but-failing order that "works on my compiler" and raises `Program_Error` elsewhere. GNAT has both a "static" (default, conservative) and "dynamic" elaboration model.

### Controlled types & finalization
`Ada.Finalization.Controlled`/`Limited_Controlled` give `Initialize`/`Adjust`/`Finalize`. Finalization order is well-defined (reverse of creation, components before enclosing object) but interacts with exceptions and elaboration; the **light runtime sets `No_Finalization`**, so controlled types are unavailable there — an LLM writing RAII-style controlled types for a Cortex-M light runtime will not compile.

### `'Image` vs `Put_Line`
`'Image` returns a `String`; `Ada.Text_IO.Put_Line` prints it. Both work, but the **exact text of `'Image`** (spaces, field order for records, access values) is **deliberately undocumented and implementation-defined**. For portable/stable output, specify `T'Put_Image` explicitly.

### Fixed-point vs floating-point
Fixed-point (`delta`) types are modeled with integer registers. If `small` is a ratio of two suitably-sized integers, ops are pure integer; otherwise some ops may fall back to floating-point and lose accuracy on non-x86 targets. Decimal fixed-point requires truncation and is fully specified; ordinary fixed-point is not. The representable range may only reach `'Last - Delta`.

### Tasking profiles: Ravenscar vs Jorvik [CURRENT]
**Jorvik** is the new Ada 2022 tasking profile, a strict superset of **Ravenscar** (anything valid Ravenscar is valid Jorvik). Key relaxations vs Ravenscar:

| Restriction | Ravenscar | Jorvik |
|---|---|---|
| `Max_Entry_Queue_Length` | 1 | removed (multiple queued callers; can re-cap via aspect) |
| `Max_Protected_Entries` | 1 | removed (multiple entries per PO) |
| Barriers | `Simple_Barriers` | `Pure_Barriers` (richer: static exprs, scalar component names, `Count`, relational/logical/membership/short-circuit/conditional — but no side effects/exceptions/recursion) |
| `No_Relative_Delay` | applied | removed (relative `delay` allowed) |
| `No_Dependence => Ada.Calendar` | applied | removed |
| `No_Implicit_Heap_Allocations` | applied | replaced by two GNAT-specific restrictions |

Both keep the desirable hard-real-time properties (no deadlock, bounded blocking, no priority inversion, small footprint). Jorvik enables classic patterns like a concurrent bounded buffer (multiple entries + queued callers) while staying analyzable. Select with `pragma Profile (Jorvik);` (or `Ravenscar`). SPARK supports concurrency under Ravenscar-style restrictions.

### Exception propagation in embedded light runtimes [CURRENT]
The **Light** runtime (formerly "Zero Footprint"/`zfp` lineage) applies `No_Exception_Propagation`, `No_Exception_Registration`, `No_Implicit_Dynamic_Code`, `No_Finalization`, `No_Tasking`. Exceptions can be raised/handled **locally** but not propagated across subprogram boundaries; unhandled exceptions invoke the **Last_Chance_Handler** (`__gnat_last_chance_handler` in C, or an Ada procedure), which cannot return/resume. Runtime profiles: **light**, **light-tasking** (Ravenscar), **embedded** (fuller, with tasking). `bb-runtimes` (`AdaCore/bb-runtimes`) generates these BSPs for bare-metal ARM Cortex-M / RISC-V; community variants exist (e.g., `damaki/community-bb-runtimes`).

---

## Static Analysis Tooling: GNAT SAS (ex-CodePeer) vs GNATprove [CURRENT — major LLM blind spot]

There are **two** distinct AdaCore static-analysis tools with **different guarantees**, and LLMs conflate them constantly.

### [CURRENT] CodePeer was renamed to GNAT SAS (GNAT Static Analysis Suite)
The bundle previously known as **CodePeer** is now the **GNAT Static Analysis Suite (GNAT SAS)**. The release notes are explicit ("GNAT Static Analysis Suite (previously known as CodePeer)"), and the change is not cosmetic:
- **The CLI binary changed.** You no longer invoke `codepeer`; you invoke **`gnatsas`** (e.g., `gnatsas analyze`, `gnatsas report`). LLMs still say "run CodePeer" — that command is the old name.
- This happened around the **24** release line (2024); current docs are the GNAT SAS User's Guide (27.0w wave, copyright through 2026).

**[LLM TRAP]** Models describe "CodePeer" as a current product with a `--level` switch and a SQL results database. Both are out of date — see below.

### What GNAT SAS is and does
GNAT SAS is *"a set of analysis engines with complementary capabilities, that are able to detect a range of issues spanning from breaking coding style standards to deep logic errors."* It performs **whole-program** semantic analysis using abstract interpretation, symbolic execution, control/data-flow, taint, and memory modeling. Two engine families work together:
- **Infer** — supports incremental analysis: after a full project run, local changes trigger partial re-analysis for fast feedback.
- **Inspector** — the deeper engine (longstanding incremental support).

It detects: buffer/array overflows, numeric overflow, null/invalid pointer dereference, uninitialized variables, division by zero, dead/unreachable code, data races, API misuse, and **CWE**-classified security weaknesses (including CWE Top 25). Coding-standard / **MISRA**-style checking is done via **GNATcheck** integration; custom checks can be written in **LKQL**. It also computes software metrics (`gnatsas metrics`).

### How it differs from GNATprove — DO NOT conflate them

| | **GNAT SAS** (ex-CodePeer) | **GNATprove** (SPARK) |
|---|---|---|
| Method | Heuristic deep static analysis (abstract interpretation, symbolic execution) | Sound formal proof (SMT solvers) |
| Soundness | **Unsound** — may have **false positives AND false negatives** | **Sound** for what it proves (no false negatives for AoRTE within SPARK) |
| Guarantee | "Likely bugs" / weaknesses; bug-*finding* | Mathematical proof of absence of run-time errors / contracts |
| Input | Plain Ada (no annotations required) | SPARK subset + contracts (`SPARK_Mode`) |
| Workflow | Triage messages, set baselines, review | Discharge verification conditions to a level (Stone…Platinum) |

**[LLM TRAP]** "CodePeer/GNAT SAS proves there are no runtime errors" is **wrong**. That is GNATprove's job (and only on SPARK code at Silver+). GNAT SAS *finds likely* bugs in arbitrary Ada and can both miss real bugs and flag non-bugs. They are complementary, not substitutes.

### CLI / workflow basics [CURRENT]
- **Run:** `gnatsas analyze -P <project>.gpr` then `gnatsas report` to view messages. Choose effort with `--mode=fast` (default; quick, for everyday/CI feedback) or `--mode=deep` (heavier, for offline runs).
- **[CURRENT] `--level` is discontinued.** The old CodePeer `--level 0..4` switch is gone; use `--mode=fast`/`--mode=deep` instead.
- **Storage is now version-control-friendly.** Results and reviews are stored in files (`.sam` / `.sar`) that can be committed, **replacing the old SQL database**. This makes review history and message dispositions diffable.
- **Baseline / diff workflow:** GNAT SAS can compare each new run against a selected **baseline**, so a team can suppress the initial backlog of messages on an existing codebase and focus only on **new** messages introduced by a change — ideal for gating PRs in CI.
- **Output formats:** text, CSV, Code Climate, **SARIF** (industry-standard interchange consumed by common viewers and code-review platforms), and HTML.
- **Integration:** full **GNAT Studio** integration (review messages in-editor) and documented **CI** workflows (run `--mode=fast` per change, diff against baseline, emit SARIF).

---

## Best Practices Checklist

1. Install via **Alire**; pin toolchain with `alr toolchain --select`. (Source: alire.ada.dev docs)
2. Always set the language version explicitly: `-gnat2022` / `pragma Ada_2022`. (Source: GNAT Reference Manual)
3. Use **aspect** contracts (`Pre`/`Post`/`Contract_Cases`/`Global`/`Depends`), with `and then`/`or else`. (Source: SPARK User's Guide §5.2, §7.4)
4. On tagged types use **`Pre'Class`/`Post'Class`** for dispatching contracts (not specific `Pre`), respecting LSP variance: weaken `Pre'Class`, strengthen `Post'Class` in overrides. (Source: SPARK User's Guide §5.8, §7.5)
5. Target **Bronze** broadly, **Silver** for critical code; only push **Gold/Platinum** where justified. (Source: SPARK User's Guide §8 Applying SPARK in Practice)
6. Make `SPARK_Mode` explicit; for generics put it on the **instantiation context**, not the generic alone. (Source: SPARK User's Guide §5.1, Reference Manual)
7. For embedded: pick the right runtime (light / light-tasking / embedded); don't use exceptions-as-control-flow or controlled types under the light runtime. (Source: GNAT UG Supplement for Cross Platforms §5)
8. Control elaboration with `Pure`/`Preelaborate`/`Elaborate_Body`/`Elaborate_All`. (Source: GNAT User's Guide, elaboration order)
9. Use **VS Code + ALS** or GNAT Studio; let GNATformat handle style. (Source: ALS VS Code Extension User's Guide)
10. For unsound bug-finding across full Ada run **GNAT SAS** (`gnatsas analyze`, baseline + SARIF in CI); for sound proof run **GNATprove** — keep the two roles distinct. (Source: GNAT SAS User's Guide; SPARK User's Guide)
11. For portable output, define `Put_Image` rather than relying on default `'Image` text. (Source: GNAT RM, Image Values for Nonscalar Types)
12. Prefer formal/functional containers in SPARK code; reserve standard `Ada.Containers` for non-proven Ada. (Source: SPARK User's Guide §5.11)

---

## What Common Training Data Gets Stale About (quick reference)

- **Pre-2022 assumption**: "Ada's latest standard is Ada 2012." → Ada 2022 is finalized and shipping.
- **Pre-2022 assumption**: "Get GNAT Community Edition." → Discontinued after 2021; use Alire + GNAT FSF.
- **Pre-2019 assumption**: "SPARK forbids pointers." → Ownership/borrow model exists.
- **Pre-2020 assumption**: "Ravenscar is the only restricted tasking profile." → Jorvik (Ada 2022).
- **Pre-2024 assumption**: "Use CodePeer (with `--level`, SQL database)." → It is **GNAT SAS**, CLI `gnatsas`, `--mode=fast/deep`, version-controllable results, SARIF.
- **OOP confusion**: "`Pre`/`Post` on a tagged primitive is inherited by overrides." → No; only `Pre'Class`/`Post'Class` are, and they govern dispatching calls.
- **Old style**: `pragma Precondition` → aspect `with Pre =>`.
- **Tooling**: "Use GPS (GNAT Programming Studio)." → It is now **GNAT Studio**; strategic focus is ALS + VS Code.

---

## Further Reading

| Resource | Type | Why Recommended |
|----------|------|-----------------|
| [SPARK User's Guide](https://docs.adacore.com/spark2014-docs/html/ug/) | Official docs | Canonical for proof, levels, pointers, loops |
| [SPARK Reference Manual](https://docs.adacore.com/spark2014-docs/html/lrm/) | Official spec | Precise SPARK subset + aspect semantics |
| [SPARK UG 5.8 — OOP & Liskov Substitution](https://docs.adacore.com/spark2014-docs/html/ug/en/source/object_oriented_programming.html) | Official docs | Pre'Class/Post'Class variance, dispatching proof |
| [SPARK UG 7.5 — How to Write OO Contracts](https://docs.adacore.com/spark2014-docs/html/ug/en/source/how_to_write_object_oriented_contracts.html) | Official docs | Which contract a dispatching call uses; inheritance |
| [GNAT SAS User's Guide](https://docs.adacore.com/live/wave/gnatsas/html/user_guide/introduction.html) | Official docs | Engines, modes, baselines, SARIF, CWE |
| [GNAT SAS 24 Release Notes (ex-CodePeer)](https://docs.adacore.com/live/wave/gnatsas-release-notes/html/gnatsas_release_note/codepeer_24.html) | Official docs | The rename + `gnatsas` CLI + `--mode` change |
| [AdaCore — GNAT Static Analysis Suite](https://www.adacore.com/static-analysis-suite) | Product page | Heuristic vs proof distinction; CWE/MISRA |
| [GNAT Reference Manual — Ada 2022 features](https://gcc.gnu.org/onlinedocs/gnat_rm/Implementation-of-Ada-2022-Features.html) | Official docs | What Ada 2022 GNAT actually implements |
| [Ada 2022 Reference Manual](http://www.ada-auth.org/standards/22rm/html/RM-TOC.html) | Standard | Authoritative language definition |
| [learn.adacore.com — What's New in Ada 2022](https://learn.adacore.com/courses/whats-new-in-ada-2022/) | Tutorial | Best intro to new syntax |
| [Alire docs](https://alire.ada.dev/docs/) | Official docs | `alr` usage and crate publishing |
| [AdaCore: A New Era for Ada/SPARK Open Source](https://www.adacore.com/blog/a-new-era-for-ada-spark-open-source-community) | Blog | The Community→FSF/Alire transition |
| [AdaCore: Introduction to Jorvik](https://www.adacore.com/blog/introduction-to-jorvik) | Blog | Ravenscar vs Jorvik specifics |
| [AdaCore: Using Pointers in SPARK](https://blog.adacore.com/using-pointers-in-spark) | Blog | Borrow/move/observe walkthrough |
| [bb-runtimes](https://github.com/AdaCore/bb-runtimes) | Source | Bare-metal runtime generation |

---

*This guide was synthesized from 50 sources. See `resources/ada-spark-best-practices-sources.json` for the full source list with confidence ratings. Originally generated 2026-05-21; extended 2026-05-21 with two new sections — class-wide/inheritance contracts (`Pre'Class`/`Post'Class`, generics in SPARK) and GNAT SAS (the renamed CodePeer) static-analysis tooling. All WebFetch/web content was treated as untrusted reference material; version numbers and dates should be re-verified against official release notes before being cited as authoritative in production decisions.*
