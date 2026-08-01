# RT-Thread Software Review

Load this reference for RT-Thread, RTT, threads, scheduler, IPC, device framework, components, init exports, FinSH/MSH, workqueue, timers, memory heap, memory pool, interrupt APIs, packages, Kconfig, or SCons changes. Here RTT means RT-Thread unless repository context explicitly means SEGGER RTT logging.

## Configuration And Initialization

- Identify RT-Thread version, BSP, architecture, toolchain, `rtconfig.h`, Kconfig/menuconfig options, SCons configuration, packages, components, heap mode, tick width/rate, priorities, and debug assertions.
- Cross-build affected BSP/configurations already supported by the repository. Check conditional compilation, component dependencies, init-export levels, symbol conflicts, package versions, and debug/release differences.
- Verify `INIT_*_EXPORT` ordering, device/component registration, dependencies, duplicate registration, partial initialization cleanup, and repeated initialization assumptions.

## Threads, IPC, And Time

- Trace `rt_thread_create/init/startup`, entry parameters, stack ownership, priority, time slice, suspend/resume, deletion/detach, and owned-object cleanup.
- Review semaphores, mutexes, events, mailboxes, message queues, signals, completions, workqueues, and timers for ownership, flags/options, timeout, full/empty behavior, deletion races, lost wakeups, and priority inversion. For semaphores, verify initial/maximum count, duplicate or overflowing release, wait result, and whether mutex ownership and inheritance semantics are required instead.
- For mutexes, verify owner-only release, recursive depth and pairing, consistent lock order, protected-condition revalidation after take or wake, no unintended blocking while held, and release on every error/timeout exit. Prove IPC objects are detached/deleted only after owners, waiters, callbacks, interrupt paths, and in-flight users are quiescent.
- Check `rt_tick_t` conversions and wraparound, `RT_WAITING_FOREVER`, zero timeout, timer callback context, blocking restrictions, interrupt enter/leave pairing, and APIs permitted in interrupt context.
- Verify atomic and critical-section code against the selected architecture and SMP configuration: supported width/alignment, interrupt-mask scope, nesting and prior-state restore, read-modify-write behavior, barriers/visibility, counter wrap, and lifetime. Do not treat `volatile` as synchronization.

## Device Framework And Memory

- Trace device register/find/open/read/write/control/close/unregister operations, open flags, reference counts, callbacks, user data, error codes, and partial-transfer semantics.
- Check `rt_malloc`/`rt_calloc`/`rt_realloc`/`rt_free`, memheap/mempool/slab usage, allocation failure, object init/detach versus create/delete pairing, stack sizes, buffer bounds, and cleanup on every exit.
- Review FinSH/MSH commands for argument count, parsing, bounds, permissions where applicable, reentrancy, blocking duration, and stale global state.
- Cover zero/one/maximum/over-limit IPC counts and message sizes, empty/full queues, no-wait/finite/forever waits, timeout-versus-release, allocation failure, partial initialization, repeated init/detach or create/delete, and teardown with in-flight work using repository-supported evidence.

## Software Evidence

Run repository-supported SCons/build, warnings, static analysis, and pure-logic tests only. Report BSP/configuration, commands, results, resource deltas, and unexecuted scheduler/device paths. Do not claim real scheduling latency, interrupt delivery, device behavior, or physical I/O.
