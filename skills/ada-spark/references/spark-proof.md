# SPARK proof

Read this when the task involves GNATprove, contracts meant for proof, or SPARK restrictions. Checked against the SPARK User's Guide sources (AdaCore/spark2014, `docs/ug`) and the Ada 2022 RM.

## Scope and SPARK_Mode

- The aspect takes `On` or `Off` (`with SPARK_Mode` alone means `On`); `pragma SPARK_Mode (Auto | On | Off)` exists as a configuration pragma, and `Auto` is only valid there.
- Without a configuration pragma, only entities marked `On` are analyzed. With `pragma SPARK_Mode (On)` in the project's configuration file, everything is in SPARK except what is marked `Off` (or files with a configuration pragma of `Auto`).
- A spec in SPARK with its body `Off` lets callers be proved against the spec while the implementation stays unanalyzed.

## Assurance levels

Stone (valid SPARK), Bronze (initialization and data flow), Silver (absence of runtime errors: no overflow, division by zero, range or index failures), Gold (key properties as contracts), Platinum (full functional correctness). Bronze is cheap across a code base; Silver is the usual target for critical code; Gold and Platinum cost real annotation effort and belong where the property matters. "No runtime errors" is Silver, not Gold.

## Flow analysis and proof

GNATprove runs flow analysis (initialization, data dependencies, `Global` and `Depends` contracts; value-independent) and proof (verification conditions discharged by SMT solvers such as Alt-Ergo, CVC5 and Z3). Flow analysis cannot track initialization cell by cell in an array; use `Relaxed_Initialization` with the `'Initialized` attribute, which moves the check to proof.

Run it as `gnatprove -P proj.gpr --level=N` (0 fastest to 4 strongest) and read the unproved check, its location and counterexample before changing code. `--mode=flow`, `--mode=silver` or `--mode=gold` limit what is checked.

## Loops and callers

- Invariants are not inferred. Any fact the postcondition needs about what a loop did must be stated with `pragma Loop_Invariant`, usually in terms of the loop index and `'Loop_Entry`. `pragma Loop_Variant` proves termination.
- A caller can only use what a callee's contract states. When a caller fails to prove, the fix is often a stronger postcondition on the callee, not more code in the caller.
- `Contract_Cases` guards must be disjoint and cover every input; GNATprove checks both.

## Specification-only code

`with Ghost` marks code that exists for proof and is removed from the build when ghost code is disabled. `Ada.Numerics.Big_Numbers` (big integers, big reals) states arithmetic properties without overflow. `pragma Assume` records a fact the tool cannot prove; each one is an unchecked hole and should say why it is true.

## Pointers and ownership

- Assignment of an access-to-variable value moves ownership; the source can no longer be used until reassigned. Observing gives a read-only borrow; borrowing gives exclusive mutable access until the borrower goes out of scope. Access-to-constant types are not ownership-checked.
- Data structures that need sharing or cycles (doubly linked lists, graphs with back pointers) do not fit the model; use indexes into arrays or keep that part outside SPARK.

## Generics

GNATprove analyzes instantiations, not generic bodies on their own, so the same generic can prove for one instance and fail for another, and messages name the instance. `SPARK_Mode` goes on the unit that contains the instantiation.

## Tagged types and LSP

- `Pre'Class` and `Post'Class` are inherited and govern dispatching calls. A dispatching call is analyzed against the class-wide contract of the operation for the operand's static (declared) type, never against the override that runs; LSP checking on every override is what makes that sound.
- In an override, `Pre'Class` must be weaker (or equal) and `Post'Class` stronger (or equal); GNATprove checks this Liskov substitution rule.
- SPARK rejects a plain (specific) `Pre` on a dispatching subprogram (SPARK RM 6.1.1(2)); state preconditions as `Pre'Class`. A specific `Post` may sit next to `Post'Class` to give non-dispatching callers a more precise result; GNATprove checks it is stronger than the class-wide one. With only class-wide contracts given, they also serve as the specific ones.
- Converting or extending to a class-wide type requires every component, including ones added by extensions, to be initialized.
