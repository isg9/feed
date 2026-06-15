---
title: Considering Strictly Monotonic Time
url: https://matklad.github.io/2026/01/23/strictly-monotonic-time.html
published: "2026-01-23T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2026/01/23/strictly-monotonic-time.html
---

# Considering Strictly Monotonic Time

Jan 23, 2026

Monotonic time is a frequently used, load bearing abstraction. Monotonicity is often enforced using the following code:

```
fn now(clock: *Clock) Instant {
    const t_raw = os_time_monotonic();

    const t = @max(t_raw, clock.guard);
    assert(t >= clock.guard);
    assert(t >= t_raw);

    clock.guard = t;
    return t;
}
```

That is, ask the OS about the current monotonic time, but don’t trust the result too much and clamp it using an in-process guard. Under normal scenarios, you can trust the OS promise of monotonicity, but, empirically, there’s a long tail of different scenarios where the promise isn’t upheld: [https://github.com/rust-lang/rust/pull/56988](https://github.com/rust-lang/rust/pull/56988)

Today I realized that, if you are doing the above, you might as well force the time to be *strictly* monotonic:

```
const t = @max(t_raw, clock.guard + 1ns);
assert(t > clock.guard);
```

The benefit of strict monotonicity is that you can tighten asserts, `assert(past <= present)` can become `assert(past < present)` and that *additionally* catches the bug where you pass in *exactly* the same instance. In other words, the `<=` version explicitly allows either query-ing the time again, or using the old value directly.

Conversely, with strictly monotonic time, you know that if you see two numerically identical time instances, they must have been ultimately derived from the exact same call to `now()`. Time becomes fundamentally less ambiguous.

The constraint here is that the resolution of the time value ( *not* the clock resolution) needs to be high enough, to make sure that repeated `+1` don’t move you into the future, but nanosecond precision seems fine for that.
