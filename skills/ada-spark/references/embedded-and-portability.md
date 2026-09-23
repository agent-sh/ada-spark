# Embedded runtimes and portability

Read this for bare-metal targets, tasking profiles, and behavior that differs between compilers or targets. Checked against AdaCore `bb-runtimes` sources and the Ada 2022 RM (D.13).

## Runtimes

- Light runtimes (for example the ARM and RISC-V ones in `bb-runtimes`) set `No_Exception_Propagation`, `No_Finalization`, `No_Tasking`, `No_Protected_Types` and `No_Streams`. Exceptions can be raised and handled in the same subprogram but not propagated (an unhandled one goes to `Last_Chance_Handler`), controlled types are unavailable, and there is no tasking.
- Light-tasking and embedded runtimes add Ravenscar or Jorvik tasking; the embedded runtime adds more of the standard library.

## Ravenscar and Jorvik

`pragma Profile (Ravenscar);` or `pragma Profile (Jorvik);`. Jorvik (Ada 2022) keeps Ravenscar's analyzable scheduling but relaxes several restrictions, among them one entry per protected object and queue length one (`Max_Protected_Entries`, `Max_Entry_Queue_Length`), relative `delay`, implicit heap allocation, `Ada.Calendar`, and simple barriers (it allows `Pure_Barriers`). RM D.13 lists the exact pragma sets.

## Portability

- `'Image` text for composite and access values is implementation-defined; define `Put_Image` (Ada 2022) where output must be stable.
- Ordinary fixed point with a `Small` that is not a power of ten or two may not map to pure integer arithmetic, and some operations are implementation-defined; decimal fixed point is exact for money.
- Representation clauses depend on bit order and storage unit; use them for real hardware or wire layouts and state `Bit_Order` and `Scalar_Storage_Order` where it matters.
- Elaboration: mark units `Pure` or `Preelaborate` where possible. When elaboration code calls into another unit, `pragma Elaborate_All` on it keeps the binder from choosing an order that fails at startup.
