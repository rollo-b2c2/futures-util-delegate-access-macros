# futures-util-delegate-access-macros

Delegation macros (verbatim copy from `futures-util`) vendored as a tiny standalone crate, so you can build small wrapper combinators (Future/Stream/Sink adapters) without depending on (or forking) `futures-util` just to get its internal helper macros.

This crate intentionally does not try to “improve” or redesign anything. It’s an explicit near line-for-line copy/paste of a subset of the delegation macros from `futures-util` (for example `delegate_*`, `delegate_all!`, `delegate_access_inner!`).

## What you get

| Macro | What it does |
|---|---|
| `delegate_future!($field)` | Implements `poll` by delegating to a pinned inner field. |
| `delegate_stream!($field)` | Implements `poll_next` (+ `size_hint`) by delegating to the inner field. |
| `delegate_sink!($field, $item_ty)` | Delegates the `Sink` poll/send/flush/close methods to the inner field (guarded behind `cfg(feature = "sink")` in the macro body). |
| `delegate_async_read!($field)` | Delegates `poll_read` (+ vectored) to the inner field. |
| `delegate_async_write!($field)` | Delegates `poll_write` (+ vectored/flush/close) to the inner field. |
| `delegate_async_buf_read!($field)` | Delegates `poll_fill_buf` + `consume` to the inner field. |
| `delegate_access_inner!($field, $inner_ty, ($($ind)*))` | Generates `get_ref`, `get_mut`, `get_pin_mut`, `into_inner` accessors, optionally threading through nested wrappers via the `$ind` tokens. |
| `delegate_all!` | “Macro that writes the wrapper type” pattern: declares a `pin_project!`’d struct with an `inner` field and can implement `Future`/`Stream`/`Fused*`/`Sink`/`Debug` plus `AccessInner` and `new(...)` helpers, depending on the invocation form. |

## Installation

Starting at version `0.3` to match the futures-rs versioning system.

This crate re-exports `pin-project-lite` as a hidden item so macro expansions can refer to `$crate::_pin_project_lite` without forcing downstream crates to depend on `pin-project-lite` directly.

```toml
[dependencies]
futures-util-delegate-access-macros = "0.3"
```


## Example 

```rust
use core::pin::Pin;
use core::task::{Context, Poll};
use futures_core::stream::Stream;

use futures_util_delegate_access_macros::{delegate_access_inner, delegate_stream};

#[pin_project::pin_project]
pub struct MyStream<S> {
    #[pin]
    inner: S,
}

impl<S> MyStream<S> {
    pub fn new(inner: S) -> Self {
        Self { inner }
    }

    delegate_access_inner!(inner, S, (()));
}

impl<S> Stream for MyStream<S>
where
    S: Stream,
{
    type Item = S::Item;

    delegate_stream!(inner);
}
```

## License

Copied code, so identical to whatever the futures-rs people set (MIT + Apache 2.0)
