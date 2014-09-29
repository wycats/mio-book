# Registering Interest

Once you have a reactor, you can register interest in events that happen on specific IOs. In `mio`, all supported IOs are represented as `IoHandle`s, which wrap the low-level file descriptor concept in each supported platform.

When you register interest, you also provide a `Token` that `mio` will send to the handler when an event occurs on the `IoHandle`. Since the low-level event loop sends events to a single `Handler`, you use this token to determine which `register` call was responsible for the event. The description of [handlers](handlers.md) shows how the `Token` is used.

To register interest, use `event_loop.register()`:

```rs
let (mut reader, mut writer) = io::pipe();

// when an event occurs, the handler will be passed `1`
event_loop.register(&reader, 1u);
```

You can use this facility to build a higher-level callback API based on functions or closures.

> TODO: Unregister interest

## Tokens

The `Token` API is a thin shim over the APIs provided by the low-level kernel APIs that `mio` is abstracting.
 `event_loop.register()` allows you to pass any Rust value that implements `Token` and `Copy` as the token. Under the hood, the low-level selector APIs can portably support sending back a pointer-sized value (`uint`) when an event occurs.

You can implement `Token` on any newtype of `uint` (which is pointer sized) to more clearly express the intent of the token value, and to add additional convenience methods for use in your handler.

The `Copy` restriction is necessary because we cannot reliably learn when an `IoHandle` is closed, so we cannot guarantee that destructors ever get run. Abstractions built on `mio` could require users to close `IoHandles` through their API, and associate a destructible object with an `IoHandle`.
