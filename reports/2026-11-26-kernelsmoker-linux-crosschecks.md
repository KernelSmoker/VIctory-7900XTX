# KernelSmoker Linux cross-checks — 26 November 2026

## Scope

CPU-only Linux shader-suite runs against the exact heads of [PR #4](https://github.com/avatarmanz/VIctory-7900XTX/pull/4) and [PR #10](https://github.com/avatarmanz/VIctory-7900XTX/pull/10). Both were tested in separate Git worktrees. No source changes were made.

| PR | Tested head | Command | Result |
| --- | --- | --- | --- |
| #4, wave32 LDS flag regions | `e29bfe273470e1c5067d5b2c5c998b84ff3cb04d` | `timeout --signal=TERM --kill-after=15s 900s cargo test -p victory-shader --lib --locked -- --test-threads=1` | 202 passed, 0 failed, 2 ignored; 2.92 s |
| #10, wide indirect dispatch split | `b390945556ab74a933afaf625e9f35700b7d9eb7` | `timeout --signal=TERM --kill-after=10s 300s cargo test -p victory-shader --lib --locked --offline -- --test-threads=1` | 204 passed, 0 failed, 2 ignored; 2.92 s |

The first offline dependency resolution for PR #4 could not find `zlib-rs` in the local Cargo cache. The network-enabled run above downloaded `zlib-rs 0.6.8` and completed successfully. Each run reported six existing Rust warnings; neither had test failures.

## Limits

These results establish only that the Linux `victory-shader` library test suites passed. They do not establish the 14 hardware LDS dispatches in PR #4, Windows DXIL validation, WARP behavior, NVIDIA D3D12 pipeline creation, PR #10's hardware dispatch behavior, gameplay, or frame rate. No game files were read or modified. No GPU workload was run; repeated correctable PCIe AER events were present during the session, so hardware and in-game validation were withheld.

## Follow-up

Repeat the hardware dispatch and gameplay A/B checks on a stable Windows or Proton host after confirming that the AvatarManz build can be produced and that the host has no continuing PCIe errors. For PR #10, compare the same saved checkpoint with `VICTORY_DISPATCH_SPLIT=0` and `=1`, using separate shader caches.

## PR #18: wave32 compute thread IDs

Tested PR head `7eda5e3637825d375507f108f9424c7cfdf252e9` in its own worktree. The focused CPU-side selector

```text
timeout --signal=TERM --kill-after=10s 300s cargo test -p victory-shader --lib wave_ticket_translation_respects_the_group_shape_and_lds_budget --locked --offline -- --test-threads=1 --nocapture
```

matched **0 tests** (204 filtered). The new `lane_order_tests` module is guarded by `#[cfg(all(test, windows))]`, so this Linux run cannot execute those cases. The complete portable suite

```text
timeout --signal=TERM --kill-after=10s 300s cargo test -p victory-shader --lib --locked --offline -- --test-threads=1
```

passed **202 tests, 0 failed, 2 ignored** in 2.63 s, with six Rust warnings. This is a Linux regression check only; the 11-shape dispatch on a 32-lane adapter and gameplay/frame-time checks remain untested here. Repeated correctable PCIe AER events were present, so no GPU workload was run.
