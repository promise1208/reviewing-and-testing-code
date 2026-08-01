# FreeRTOS Software Review

Load this reference for FreeRTOS tasks, scheduler, queues, queue sets, semaphores, mutexes, recursive mutexes, event groups, direct-to-task notifications, stream/message buffers, software timers, heap schemes, or `FromISR` APIs.

## Configuration And Build

- Identify FreeRTOS version and relevant `FreeRTOSConfig.h` values: tick width/rate, priorities, preemption, time slicing, static/dynamic allocation, timers, mutexes, notifications, stack overflow checks, malloc-failed hook, trace, and `configASSERT`.
- Cross-build every affected target/configuration already supported by the repository. Check feature guards, unavailable APIs, tick-type width, port-specific types, interrupt-priority macros, and debug/release assertion differences.

## Tasks And Time

- Trace task creation, startup ordering, handles, parameters, stack lifetime, priority, blocking, suspend/resume, deletion, restart, and cleanup of owned IPC/resources.
- Check `vTaskDelay` versus `vTaskDelayUntil`, zero/infinite waits, `portMAX_DELAY` semantics, `TickType_t` conversions, tick wraparound, priority inversion, starvation, and scheduler-state assumptions.
- Verify task functions do not return unexpectedly and that deletion paths cannot leave callbacks, timers, queues, locks, or shared pointers referencing dead state.

## ISR And Synchronization

- Use only `...FromISR` APIs in ISR context where required. Check the higher-priority-task-woken flag, `portYIELD_FROM_ISR`/port equivalent, interrupt priority limits, nesting, and critical-section pairing.
- Review queue/semaphore/mutex/event-group/notification ownership, send/receive direction, item size, full/empty behavior, timeout, deletion races, lost wakeups, duplicate signals, and every API return. For semaphores, verify binary versus counting intent, initial and maximum count, duplicate/overflowing give, and whether a mutex API is required for ownership and priority inheritance.
- Check that only the mutex owner releases it, recursive take/give APIs are paired, protected conditions are revalidated after take or notification wake, lock ordering is consistent, and no error path leaks the lock. Review blocking while holding locks and APIs forbidden before scheduler start, from ISR context, or from timer callbacks.
- Prove queues, semaphores, mutexes, and task handles are not reset or deleted until all owners, waiters, callbacks, ISR paths, and in-flight users are quiescent. Cover partial object creation and repeated start/stop.
- Treat `volatile`, task suspension, and critical sections as context-limited tools, not general atomicity. Verify port-supported atomic width/alignment, read-modify-write or compare/exchange behavior, required compiler/CPU barriers, multi-core port semantics, counter saturation/wrap, and object lifetime.

## Memory And Software Tests

- Review heap scheme selection, allocation failure, static object backing storage, stack depth units, stack high-water interpretation in code, queue storage sizing, integer overflow, and cleanup on partial initialization.
- Exercise repository-supported pure logic and RTOS-independent state machines for malformed data, timeouts, queue-full/error returns, cancellation, repeated init/deinit, and tick-wrap calculations without adding scheduler simulation or HAL fakes.
- Include zero/one/maximum/over-maximum semaphore counts, empty/full queues, zero/finite/infinite waits, timeout-versus-give races, exact-fit/over-limit storage, and creation-failure unwind where repository-supported evidence can exercise them. State scheduler interleavings that remain unexecuted.
- Report exact build/test/static-analysis commands, affected configuration values, and unexecuted scheduling behavior. Do not claim real-time timing, ISR delivery, context-switch behavior, or peripheral operation.
