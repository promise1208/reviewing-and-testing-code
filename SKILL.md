---
name: reviewing-and-testing-code
description: Use when reviewing, auditing, or completing embedded software or firmware changes involving embedded Linux, FreeRTOS, RT-Thread, other RTOS, bare-metal, BSP, HAL, drivers, ISR, DMA, cache, semaphores, mutexes, atomics, shared memory, memory boundaries, integer overflow, resource exhaustion, limit testing, protocols, timeouts, or recovery; especially for GPIO, UART/USART, I2C/IIC, SPI/QSPI/OSPI, CAN/FDCAN, USB, SDIO/SDMMC/eMMC, Ethernet/LwIP, MQTT/MQTT-SN, HTTP/HTTPS, WebSocket, CoAP/LwM2M, IoT, TLS/DTLS, Wi-Fi, Bluetooth/BLE, Matter/Thread/Zigbee, LoRaWAN, NB-IoT/LTE-M, ADC, DAC, PWM, timers, RTC, watchdog, Flash/NVM, display, touch, sensors, and MIPI.
---

# Reviewing And Testing Embedded Software

## Overview

Treat post-change review as a software completion gate. Prove embedded code behavior with bidirectional reasoning, repository-supported builds and tests, static evidence, and failure-path analysis. Synchronization, atomicity, memory bounds, and resource limits are mandatory whenever applicable.

This skill does not include physical hardware, HIL, QEMU, Renode, vendor simulators, or host HAL fake testing. Do not create, require, execute, or claim results from those environments. Physical signals, electrical behavior, real peripheral responses, throughput, and power are outside the conclusion.

## Mandatory Workflow

1. Reconstruct the intended behavior from the request, specifications, repository conventions, and final diff. State assumptions.
2. Map the affected end-to-end flow: inputs, outputs, components, interfaces, state, ownership, synchronization, timing, errors, and recovery.
3. Review **top-down**: system goal -> architecture -> module contracts -> functions -> branches and expressions.
4. Review **bottom-up**: each changed expression/function -> callers and callees -> shared state -> module contracts -> system behavior.
5. Always load the common embedded reference, then load every matching operating-system or framework reference below. When domains overlap, load all matching references.
6. Build a risk matrix from the code's actual invariants. Include applicable normal, boundary, malformed, partial, repeated, concurrent, timeout, exhaustion, recovery, and long-running cases. Apply the mandatory focus below; do not replace it with a generic concurrency or memory checklist.
7. Run the repository's relevant cross-builds, link checks, warnings, static analysis, and existing software tests. Add pure-logic behavioral tests when feasible without introducing hardware, simulator, emulator, or HAL-fake infrastructure.
8. Re-read the final diff after results. Subject to the bounded loop below, fix issues, then repeat affected review passes and checks. Evidence from before the last edit is stale.
9. Report findings, evidence, limitations, and residual risks.

## Mandatory Cross-Cutting Focus

For changed or reachable code that uses shared state, synchronization objects, atomics, buffers, lengths, counts, indices, or resource pools, record the applicable invariant and how it was proved.

| Area | Required proof |
|---|---|
| Semaphore, mutex, event, condition, critical section | Justify the primitive and mode; check creation, initial and maximum state, ownership, permitted task/ISR/callback contexts, acquire/release or wait/signal pairing, return values, zero/finite/infinite waits, priority behavior, lock order, blocking while held, cancellation, deletion, and partial-init unwind. Treat a semaphore used as a mutex as a design decision requiring explicit justification. |
| Protected predicates and state | Name the predicate and every field that forms it. Evaluate them under the same protection as the dependent transition; revalidate any pre-lock fast-path check after acquisition, and reevaluate condition-wait predicates in a loop after every wake. Check every early return, error, timeout, retry, and callback path for state rollback and lock release. |
| Atomic operations and visibility | Identify the exact object and operation that must be atomic. Do not infer compound atomicity from `volatile` or from individually atomic accesses. Compiler/CPU barriers order accesses but do not make conflicting non-atomic C/C++ accesses data-race-free. Check read-modify-write, compare/exchange retry logic, memory order or barriers, ISR and multi-core visibility, DMA/cache interaction, counter wrap or saturation, reference lifetime, and ABA exposure. |
| Memory and arithmetic bounds | Prove ranges using the actual integer types before pointer arithmetic, allocation, indexing, copying, terminator addition, alignment rounding, or descriptor calculation. Check signed conversion, truncation, multiplication/addition overflow, underflow, one-past-end access, overlap, aliasing, alignment, and lifetime. |
| Limits and interleavings | Exercise or statically prove `0`, `1`, minimum, maximum, and just-over-maximum values; empty/full and count saturation; timeout edges and wraparound; first/repeated/concurrent operations; competing success/failure/cancellation; partial initialization; teardown with in-flight work; exhaustion; recovery; and long-running stability. |

Use repository-supported evidence only. Do not introduce scheduler mocks, HAL fakes, simulated devices, or host models of unavailable RTOS/hardware behavior to manufacture coverage. Repository-supported host tests of target-independent logic or native embedded Linux services remain valid. When an interleaving or hardware-visible property cannot be executed within the skill's evidence boundary, provide the source-level proof and state the unexecuted behavior and residual risk explicitly.

## Bounded Test And Repair Loop

Apply this limit separately to each functional change:

1. Track a retry key consisting of the functional change, target or MCU configuration, toolchain, build mode, test or command, and logical failure location. Record the error signature separately for diagnosis. A moved line number or changed error message alone does not create a new retry key.
2. After a failed check, compare its retry key with the immediately preceding failure for that functional change. Start at one for a different logical location; increment for the same retry key even when the error text or symptom changes.
3. On the first or second consecutive failure, diagnose the cause and state the repair method. Apply an in-scope, evidence-based fix when safe, then run the relevant check again.
4. On the third consecutive failure with the same retry key, stop testing and auto-fixing that functional change immediately. Do not run an equivalent check again, apply another speculative fix, reset the counter through superficial edits, or start a fresh retry cycle.
5. Report the exact failing command, logical location and error signature, all three results, likely root cause, fixes attempted, and the recommended repair or manual next step. Mark the functional change as incomplete.

If no safe, evidence-based fix is available after either of the first two failures, stop earlier and report the same information. A different logical failure location starts its own count, but never use changing symptoms as justification for an unbounded loop.

## Domain Routing

| Changed area | Required reference |
|---|---|
| Any embedded software, firmware, BSP, HAL, driver, ISR, DMA, cache, protocol, peripheral, or constrained-memory change | [embedded.md](references/embedded.md) |
| Embedded Linux, kernel module, Kconfig, device tree, platform driver, character device, sysfs, ioctl, workqueue, or user/kernel boundary | [embedded-linux.md](references/embedded-linux.md) |
| FreeRTOS tasks, queues, semaphores, mutexes, event groups, notifications, timers, heap, or `FromISR` APIs | [freertos.md](references/freertos.md) |
| RT-Thread threads, IPC, device framework, components, FinSH, workqueue, timers, heap, or interrupt APIs | [rt-thread.md](references/rt-thread.md) |
| Bare-metal startup, vector table, linker script, superloop, polling, interrupt, or register-level code | [bare-metal.md](references/bare-metal.md) |
| MQTT, MQTT-SN, HTTP, HTTPS, WebSocket, CoAP, LwM2M, TLS/DTLS, TCP/UDP, sockets, DNS, DHCP, Wi-Fi, Bluetooth Classic, BLE, GAP, GATT, ATT, HCI, Matter, Thread, Zigbee, LoRaWAN, NB-IoT, LTE-M, cellular modem, AT commands, PPP, or IoT connectivity | [connectivity.md](references/connectivity.md), plus the matching OS reference |
| LVGL, displays, screens, event-driven GUI, input, or navigation | [gui.md](references/gui.md), plus the matching OS reference |
| Embedded Linux user-space APIs, persistence, filesystem, network, concurrency, or security | [services.md](references/services.md), plus [embedded-linux.md](references/embedded-linux.md) |

For unfamiliar embedded software, derive a checklist from its compile-time configuration, API contracts, state machine, ownership, context rules, lifecycle, failure modes, and resource limits. Do not add hardware or simulation requirements.

## Completion Gate

Do not claim completion when:

- a required test, build, or analysis command fails or is still running;
- a high-severity correctness, safety, corruption, deadlock, or memory issue remains;
- an applicable critical software invariant has neither repository-supported executable evidence nor an explicit source-level proof;
- only the happy path was exercised;
- the final edit has not been re-reviewed and re-tested;
- the same retry key reached the three-attempt limit;
- an applicable synchronization primitive, protected predicate, atomic operation, memory-order requirement, or object-lifecycle transition has not been reviewed explicitly;
- a buffer, index, count, allocation, or pointer range is justified only by typical inputs rather than its actual type and configured limits;
- applicable zero, maximum, over-limit, saturation, wraparound, concurrent, or teardown cases lack executable evidence or explicit source-level proof. Reporting an environmental limitation does not waive this requirement.

Hardware and simulation evidence is out of scope, not a software-test failure and not evidence of a pass. State that boundary explicitly. Never turn it into a bench, HIL, emulator, simulator, or HAL-fake task.

## Required Report

Include:

- review scope and affected end-to-end flow;
- target or MCU configuration, toolchain, build mode, and relevant compile-time options;
- findings and fixes ordered by severity;
- commands run and concise actual results;
- covered normal/failure/recovery/resource scenarios;
- synchronization and memory invariants, their protection or ownership, and covered limit/interleaving cases;
- software checks not run, out-of-scope hardware behavior, and residual software risk.

## Common Mistakes

| Shortcut | Correction |
|---|---|
| "The build passes" | A build proves syntax/linkage, not behavior or recovery. |
| "The change is small" | Trace its callers, shared state, and lifecycle anyway. |
| "Static review is enough" | Run executable checks whenever feasible. |
| "Memory probably fits" | Measure budgets, peaks, stack, fragmentation, and repeated-cycle stability. |
| "There is a lock" | Prove it protects every predicate field, revalidate any pre-lock check after acquisition, and reevaluate condition-wait predicates in a loop. |
| "The access is atomic" | Prove the complete invariant, visibility ordering, counter limits, and object lifetime, not only one load or store. |
| "The length was checked" | Prove the arithmetic cannot overflow before the comparison and cover zero, exact-fit, and just-over-limit values. |
| "The stress test passed" | Stress is supporting evidence; require invariant-based checks and explicit unexecuted interleavings. |
| "Hardware is unavailable" | Record hardware behavior as out of scope; do not prescribe or perform bench, HIL, emulator, simulator, or HAL-fake testing. |
| "One more retry might work" | Stop after the third consecutive failure at the same logical location and issue the required error report. |

## Red Flags

- No end-to-end flow map
- Only one review direction
- No malformed, timeout, exhaustion, or recovery scenario
- Test evidence predates the final change
- "Logically tested" without commands or explicit limitations
- A fourth attempt at the same logical failure location
- Hardware, HIL, emulator, simulator, or HAL-fake work presented as software-test evidence
- A condition checked outside its protecting lock without revalidation
- A synchronization object deleted while waiters, owners, callbacks, or in-flight users may remain
- `volatile`, a single atomic access, or interrupt masking asserted to protect a larger invariant without proof
- Bounds checked only after overflow-prone arithmetic or without exact-fit and over-limit cases

Any red flag means the review is incomplete.
