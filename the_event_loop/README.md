# The Event Loop

The core of `mio` is the `EventLoop` API.

The lifecycle of an event loop involves:

* [Creating](creating.md) a new event loop (`EventLoop::new` or `EventLoop::configured`)
* [Registering](registering.md) interest in events (`EventLoop::register`)
  * An `IOHandle` is [Readable or Writable](readable_and_writable_notifications.md)
  * A [timer expired](timers.md)
  * A [signal](signals.md) was delivered
  * A [message](sending_messages.md) was sent to the event loop
* Running the event loop with a [Handler](handlers.md), waiting for events (`EventLoop::run` or `EventLoop::run_once`)
* [Sending a message](sending_messages.md) to the event loop (`EventLoop::channel().send()`)
* Shutting down the event loop (`EventLoop::shutdown`)

When the event loop is shut down, it will finish sending any pending messages to the registered handler, and return `Ok(handler)` from the original call to `EventLoop::run()`.
