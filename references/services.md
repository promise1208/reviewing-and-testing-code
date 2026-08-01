# Service, API, Storage, And Concurrency Review

Load this reference for embedded Linux user-space APIs, persistence, filesystem, networking, concurrency, and security. Apply it together with the embedded Linux reference. Do not introduce hardware checks.

## Contracts And State

- Trace request/event/input through validation, authorization, business rules, persistence, side effects, response, and observability.
- Check null/empty/maximum/malformed input, schema and API compatibility, default values, encoding, ordering, pagination, units, and time boundaries.
- Verify state-machine legality, transaction boundaries, idempotency, duplicate/out-of-order delivery, partial success, rollback/compensation, and retry safety.

## Failure, Concurrency, And Resources

- Exercise timeout, cancellation, disconnect, partial reads/writes, unavailable dependency, stale data, disk full, permission failure, corrupt file/record, rate limit, and restart recovery.
- For POSIX semaphores, check process-sharing mode and initial limit, `sem_wait`/timed-wait interruption and retry policy, absolute-deadline clock behavior, `sem_post` overflow, return/`errno` handling, release balance, and quiescence before destroy.
- For pthread mutexes and condition variables, check mutex type, robustness and owner-death recovery, process-sharing, owner-only unlock, cancellation cleanup, consistent lock order, and destroy preconditions. Evaluate predicates while holding the mutex and wait in a loop; verify the selected condition-variable clock and timeout conversion.
- For C/C++ atomics and lock-free state, check object lifetime, exact read-modify-write scope, compare/exchange loops, memory order, reference/generation reuse, and counter wrap. Verify shutdown with waiters or in-flight requests and whether filesystem/database operations provide the assumed transaction and durability atomicity.
- Bound memory, connections, file handles, threads/tasks, queues, caches, request bodies, retries, and logs. Check leaks and resource exhaustion under repeated or concurrent use.
- Review authentication, authorization, secret handling, injection, path traversal, data exposure, unsafe deserialization, and audit behavior where relevant.

## Verification

Run repository-supported focused unit/behavior tests plus relevant existing integration, migration, compatibility, native concurrency, sanitizer, and failure-injection tests. Use real user-space synchronization or existing production-neutral test hooks; do not add RTOS schedulers, device models, or hardware/HAL fakes. Include zero/exact-limit/over-limit resources and payloads, deterministic timeout/completion/cancellation orderings, duplicate operations, partial initialization, restart, exhaustion, and long-running counter/resource stability. Confirm metrics/logs make retries, rejection, partial failure, and recovery diagnosable without exposing secrets. State unavailable external dependencies and provide reproducible test steps rather than treating them as passed.
