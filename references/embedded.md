# Embedded Software Common Review

Always load this reference for embedded software, firmware, BSP, HAL, drivers, peripherals, ISR, DMA, protocols, or constrained-memory code. Review software only: do not require physical hardware, HIL, QEMU, Renode, vendor simulators, or host HAL fakes.

## Build And Configuration

- Identify the MCU/SoC or target profile, architecture, endianness, ABI, toolchain and version, build mode, linker script, feature macros, Kconfig/menuconfig options, generated code, CMSIS, BSP, HAL, middleware, and library versions affected by the change.
- Run repository-supported clean cross-builds for affected configurations. Inspect errors, warnings, undefined or duplicate symbols, section overflow, orphan sections, ABI mismatch, conditional-compilation gaps, and debug/release differences.
- Check generated-code ownership and regeneration rules. Keep application changes outside generated regions unless the repository explicitly owns those files.
- Trace startup, initialization, partial initialization, normal operation, suspend/resume when implemented, error unwind, deinitialization, repeated initialization, reset handling, and shutdown paths in code.

## Context, Concurrency, And Time

- Classify every entry point as reset/startup, thread/task, ISR, callback, timer, workqueue, polling loop, or asynchronous completion context.
- Check ISR-safe APIs, interrupt nesting and masking, critical sections, lock ordering, priority inversion, deadlock, starvation, reentrancy, atomicity, `volatile`, compiler barriers, memory barriers, and shared-state ownership.
- Verify timeout units, signed/unsigned conversions, zero and maximum values, tick or counter wraparound, deadline comparison, retry bounds, cancellation, and stale completion handling.
- For DMA and cache code, verify buffer lifetime, address range, alignment, length, direction, descriptor ownership, completion and error paths, clean/invalidate ordering, barriers, abort, and teardown races from source and API contracts only.

## Synchronization And Atomicity

| Mechanism | Required review |
|---|---|
| Counting/binary semaphore | Confirm that signaling or resource counting is intended; verify initial and maximum count, take/give direction, overflow or duplicate give, lost signal, timeout, fairness/priority behavior, return values, ISR variants, and quiescence before deletion. A semaphore does not imply mutex ownership or priority inheritance. |
| Mutex/recursive mutex | Name the protected predicate and fields; keep check-and-update under the same lock; revalidate a pre-lock fast path after acquire and reevaluate a condition-wait predicate in a loop after wake; verify owner-only release, recursive API pairing, lock rank/order, priority inversion handling, no unintended blocking while held, and release on every exit. |
| Event/condition/notification | Treat wakeup as a reason to re-evaluate state, not proof that the condition remains true. Check stale, duplicate, coalesced, lost, and out-of-order signals plus clear/consume semantics. |
| Critical section/interrupt mask | Prove the protected execution contexts, maximum masked duration, nesting and prior-state restore. Do not assume single-core interrupt masking excludes another core, DMA, or bus master. |
| Atomic/lock-free state | Verify supported width and alignment, exact read-modify-write scope, compare/exchange retry and expected-value handling, required memory order or barriers, wrap/saturation, reference lifetime, and ABA risk. `volatile` provides neither mutual exclusion nor inter-context ordering; barriers alone do not make conflicting non-atomic C/C++ accesses data-race-free. |
| Lifecycle | Prove publication only after initialization, handle validity at every use, rollback after partial creation, and quiescence before reset/delete/free. Include waiters, owners, callbacks, timers, work items, ISR paths, and in-flight operations. |

Express each shared-state rule as an invariant, for example: "queue indices and occupancy change together while protected" or "payload writes happen-before the ready flag is observed." Review every participant against the same invariant instead of checking lock calls in isolation.

## Memory And Resource Budgets

Use numeric evidence where repository tools expose it:

| Area | Software checks |
|---|---|
| Flash/ROM and static RAM | Compare map/size output with limits and baseline; inspect sections, globals, constants, alignment, linker regions, and configuration-dependent growth. |
| Stack | Inspect task/ISR stacks, call depth, recursion, large locals, compiler stack-usage output, static estimates, overflow hooks, and configured limits. |
| Heap/pools | Check allocation failure, ownership transfer, cleanup on every exit, leak, double free, use-after-free, fragmentation assumptions, and repeated lifecycle paths. |
| Buffers/rings/queues | Prove bounds, full/empty distinction, index wrap, length arithmetic, integer overflow, producer/consumer ownership, backpressure, and reset behavior. |
| Handles/descriptors/timers | Bound counts and verify acquire/register/start pairs with release/unregister/stop on success, failure, cancellation, and teardown. |

Before computing `len + trailer`, `count * item_size`, an aligned size, or an end pointer, prove the operation itself cannot overflow. Prefer an equivalent subtract-first comparison only after proving the subtraction is valid. Check exact-fit behavior separately from one-past-capacity behavior.

## Mandatory Limit Matrix

Apply every relevant row with repository-supported tests or explicit static proof:

| Dimension | Required cases |
|---|---|
| Length/index/count | `0`, `1`, minimum valid, exact capacity, one over capacity when representable, and the integer type's wrap/overflow boundary. When one over is unrepresentable, prove rejection before the overflowing arithmetic. |
| Queue/ring/pool/semaphore | Empty, one item/token, full/maximum count, full plus one producer/give, drain to empty, repeated wrap, and reset while empty/full. |
| Timeout/tick/deadline | No-wait, one tick/unit, just-before and at expiry, maximum finite wait, infinite-wait sentinel where supported, cancellation, and counter wrap. |
| Concurrency | Each relevant ordering of check/acquire/update/release, signal-before-wait, signal-during-wait, competing producers/consumers, timeout versus completion, and cancellation/teardown versus in-flight use. |
| Lifecycle/failure | Each allocation or creation failure, partial initialization, first and repeated start/stop, double-call rejection or idempotence, waiter/owner during teardown, restart, and stable resource baseline after recovery. |
| Long running | Counter/index wrap, semaphore or queue saturation, bounded retries/logs, stack/heap/pool trend, fragmentation assumptions, stale handles, and generation reuse. |

## Peripheral And Driver Code

| Keywords | Required software review |
|---|---|
| GPIO, EXTI, pinctrl | Mode and polarity constants, masks, read-modify-write races, atomic set/clear APIs, interrupt registration and clearing, debounce state, ownership, init/deinit, and invalid pin handling. |
| UART, USART, serial | Ring buffers, partial RX/TX, length and baud calculations, ISR/DMA interaction, overflow, timeout, cancellation, parser resynchronization, and callback lifetime. |
| I2C, IIC, SMBus, SPI, QSPI, OSPI | Address/mode/word-size configuration values, transfer state machine, busy/error/timeout paths, shared-bus locking, partial completion, DMA ownership, abort, and cleanup. |
| CAN, FDCAN, USB, SDIO, SDMMC, eMMC | Frame or request bounds, descriptor and endpoint lifecycle, queue saturation, callbacks, disconnect/reset states represented in software, retry bounds, and resource cleanup. |
| Ethernet, MAC, PHY APIs, LwIP | Buffer ownership, pbuf lifecycle, zero-copy lifetime, callbacks, queue/backpressure, link-state handling in code, timeout, retry, and thread-context rules. |
| Wi-Fi, Bluetooth, BLE, radio stacks | Driver/stack API state, buffer ownership, callbacks, queues, timeout, cancellation, reconnect, credentials, pairing data, and teardown; never infer RF or link behavior. |
| ADC, DAC, PWM, timer, RTC, watchdog | Unit and scale conversion, resolution, overflow, rollover, compare/update ordering, callback context, start/stop/restart, validation, and software fallback state. |
| Flash, EEPROM, NVM, filesystem | Address and erase/program alignment, bounds, integer overflow, versioning, CRC, interrupted-update recovery logic, wear assumptions, atomic metadata, and rollback behavior. |
| Display, MIPI, LVGL, touch, sensor | Pixel format, stride, buffer ownership, event/callback lifetime, coordinate and sample bounds, asynchronous state, allocation cleanup, and repeated create/delete or open/close paths. |

For other peripherals, derive checks from configuration values, API contracts, transaction state, context, ownership, partial/error completion, timeout, resource exhaustion, cancellation, teardown, and recovery logic.

## Protocol And Data Tests

- Exercise pure software parsers and state machines with empty, minimum, maximum, malformed, truncated, concatenated, duplicated, out-of-order, checksum-failed, unsupported-version, and timeout inputs when repository-supported tests can run without simulation infrastructure.
- Check byte order, packing, alignment, strict aliasing, signedness, length fields, integer overflow, CRC/checksum boundaries, resynchronization, idempotency, and backward compatibility.
- For every parser or copy boundary, verify that validation precedes overflow-prone arithmetic and pointer formation; cover zero-length, exact-fit, truncated, and declared-length-larger-than-storage inputs.
- Prefer tests of extracted pure logic. Do not create fake registers, fake buses, device models, or simulator-only production hooks.

## Evidence Boundary

Run only repository-supported software commands: cross-builds, link/map/size checks, compiler warnings, static analysis, linters, and existing or pure-logic tests. Report exact commands and results. Physical signals, electrical timing, real peripheral behavior, actual throughput, power, and hardware recovery are out of scope and must not be inferred from software evidence.
