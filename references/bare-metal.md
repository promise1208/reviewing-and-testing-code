# Bare-Metal Software Review

Load this reference for bare-metal, no-OS, startup assembly, reset handler, vector table, linker script, CMSIS, superloop, polling, register-level driver, interrupt, exception, or boot code.

## Startup And Linkage

- Identify MCU/SoC, core, architecture, ABI, toolchain, startup file, linker script, memory map, vector-table location, entry point, and compile-time feature configuration.
- Review reset flow, stack pointer, vector entries, weak/default handlers, `.data` copy, `.bss` zeroing, constructors when applicable, clock-independent early code, relocation, bootloader/application boundaries, and reset-cause state handling in software.
- Inspect map/size output for Flash/RAM regions, section placement, alignment, `NOLOAD`, retained/no-init memory, stack/heap reservations, orphan sections, overflow, duplicate symbols, and dead-code/LTO effects.

## Superloop, Interrupts, And Time

- Trace initialization order, partial-failure handling, main-loop state machines, cooperative scheduling, polling, sleep/wake calls represented in code, shutdown/reset requests, and repeated peripheral init/deinit.
- Ensure loops and polling have bounded exit or explicit permanent-loop intent. Check timeout arithmetic, counter rollover, zero and maximum delays, nonblocking progress, starvation, and watchdog service placement in source.
- Review vector names, ISR prototypes, shared data, `volatile`, atomic access, read-modify-write races, critical-section save/restore, nesting, pending-flag handling code, callback lifetime, and foreground/ISR buffer ownership. Prove the exact interrupt levels and cores excluded, bound masked duration, and restore the saved prior mask state rather than unconditionally enabling interrupts.
- Verify atomic access width and alignment against the architecture and generated instructions. Check compound check-then-act sequences, exclusive-access or compare/exchange retry behavior, compiler/CPU barriers, multi-core visibility, counter wrap, and interaction with DMA or other bus masters. `volatile` and single-core interrupt masking are not universal synchronization.

## Drivers, Memory, And Errors

- Check register addresses, masks, shifts, reserved-bit preservation, access width, write-one-to-clear handling, ordering, barriers, and configuration constants against repository headers and available specifications.
- Trace driver init/start/transfer/complete/error/abort/stop/deinit state, partial completion, invalid arguments, busy state, timeout, retry bounds, cancellation, and cleanup.
- Prove array, buffer, ring, queue, descriptor, and length bounds before pointer or size arithmetic; check zero/exact-fit/over-limit values, signedness, addition/multiplication overflow, index wrap, alignment, packing, aliasing, stack use, static allocation, and ownership.
- Test repository-supported pure algorithms, parsers, state machines, timeout/wrap calculations, and error handling without adding fake registers, fake buses, emulators, or simulators.

## Software Evidence

Report exact cross-build, link/map/size, warning, static-analysis, and pure-logic test commands and results. Do not claim reset timing, interrupt arrival, register side effects, peripheral traffic, clock accuracy, or electrical behavior.
