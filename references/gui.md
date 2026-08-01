# LVGL And Event-Driven GUI Review

## Navigation And State

- Draw the reachable screen/page transition graph, including back, cancel, timeout, error, sleep/wake, and reset paths.
- Check initial route, invalid and duplicate transitions, rapid repeated input, reentrant navigation, history/back-stack behavior, focus/input group, and state restoration.
- Verify model and displayed state remain consistent across asynchronous updates, partial initialization, communication failure, and page recreation.

## Lifetime And Threading

- Identify owners of screens, child objects, styles, fonts, images, timers, animations, event descriptors, user data, and frame/draw buffers.
- Check create/load/unload/delete order and every early exit. Prevent callbacks, timers, animations, DMA, or async work from touching deleted objects or freed data.
- Remove or cancel event sources before destruction. Validate deletion from within callbacks and transitions queued during teardown.
- Call LVGL only from the permitted thread/context or use the project's synchronization mechanism. Check locks across callbacks and blocking driver operations.

## Display And Input

- Verify resolution, rotation, color format, stride, clipping, partial refresh, flush completion, buffer count, cache coherency, tearing sync, and display sleep/wake ordering.
- Test touch/key/encoder press, release, long-press, repeat, debounce, calibration, out-of-range coordinates, missing input, and input during transitions.

## Memory And Recovery Tests

- Record memory before and after repeated enter/exit cycles; include heap/pool usage, largest free block, object count, timer/event count, and framebuffer/draw-buffer ownership.
- Exercise allocation failure, partially built pages, image/font load failure, rapid navigation, repeated sleep/wake, display-driver failure, and communication timeout.
- Require cleanup to a stable baseline, a usable error/fallback screen, and no callback-after-delete, leak, double delete, stale focus, or permanently busy display.
