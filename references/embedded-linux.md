# Embedded Linux Software Review

Load this reference for embedded Linux, Linux kernel, kernel module, Kconfig, Makefile/Kbuild, device tree, platform driver, character device, sysfs, ioctl, netlink, procfs, debugfs, workqueue, threaded IRQ, or user/kernel boundary changes. Do not require a target board, VM, emulator, simulator, or fake device.

## Build And Integration

- Identify kernel version, architecture, cross-compiler, defconfig, Kconfig symbols, module/built-in mode, compiler options, and affected device-tree sources or overlays.
- Run repository-supported kernel, module, DTB, and user-space cross-builds. Inspect compiler warnings, `modpost`, exported symbols, section mismatches, license/namespace constraints, and configuration-dependent code.
- Use available project commands such as `W=1`, `sparse`, `smatch`, Coccinelle, `dt_binding_check`, or `dtbs_check` only when already supported by the repository and installed toolchain.

## Driver Lifecycle

- Trace module init/exit and `probe`, deferred probe, remove, shutdown, suspend, resume, runtime PM, and every error-unwind label.
- Verify `devm_*` and manual ownership are not mixed incorrectly. Pair register/request/map/get/open operations with unregister/free/unmap/put/close across partial failure and removal.
- Check device-tree `compatible`, `reg`, `interrupts`, `clocks`, `resets`, `gpios`, `pinctrl`, `dmas`, cell counts, endianness, defaults, optional properties, and malformed or missing property handling in code and bindings.

## Context And Concurrency

- Distinguish process, atomic, hard IRQ, threaded IRQ, softirq, tasklet, timer, workqueue, completion, and callback context. Do not call sleeping APIs from atomic context.
- Review spinlock/mutex/semaphore/completion/waitqueue/RCU/refcount/kref usage, lock ordering, IRQ save/restore pairs, work and timer cancellation, remove races, use-after-free, and module unload safety. Use semaphores for resource counts only when their ownership and priority semantics fit; verify initial/count limits, down/up results and balance, and teardown with waiters.
- Verify every waitqueue or condition predicate while holding the required lock or through its documented atomic access, and recheck it after wake. Check mutex owner/release paths, nested locks, sleeping while locked, interruptible-wait exits, and lock coverage of every field in compound state.
- For `atomic_t`, bit operations, refcounts, lock-free publication, and RCU state, prove the entire invariant rather than one access. Check operation-specific memory ordering/barriers, compare/exchange loops, overflow or saturation, ABA/lifetime, and whether `refcount_t`, a lock, or another primitive better matches the contract.
- Check `READ_ONCE`/`WRITE_ONCE` and acquire/release helpers against their actual access and ordering contract; they do not make multi-access invariants atomic. For RCU, verify reader coverage, pointer publication, grace-period or callback completion before reclamation, and teardown barriers. Never use a barrier alone to justify conflicting plain C accesses.
- Check MMIO accessors, bit operations, barriers, polling timeout helpers, jiffies wraparound, DMA mapping API pairing, scatter-gather bounds, and cache/IOMMU ownership contracts from source.

## Memory And User Interfaces

- Check `kmalloc`/`kzalloc`/`vmalloc`/`devm_*`, error pointers, `IS_ERR`/`PTR_ERR`, reference ownership, allocation flags, size overflow, cleanup, and sensitive-data exposure. Prove addition, multiplication, flexible-array, alignment, page-count, user-length, and end-pointer arithmetic before allocation or access; use repository/kernel overflow helpers where applicable.
- Validate `copy_from_user`/`copy_to_user`, ioctl command/size/direction, `compat_ioctl`, sysfs parsing and formatting, netlink attributes, procfs/debugfs lifetime, poll/select semantics, and partial read/write behavior.
- Treat UAPI, device-tree bindings, sysfs ABI, ioctl layout, and netlink schema as compatibility contracts. Check validation, permissions, capabilities, bounds, and error codes.

## Software Evidence

Report affected configurations, exact build/static-analysis commands, warnings, module or image size changes, pure-logic test results, and unexecuted kernel paths. Do not claim probe success, IRQ delivery, DMA operation, bus traffic, or peripheral behavior without hardware; those claims are outside this skill.
