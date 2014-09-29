# Creating an Event Loop

When you create an event loop, remember that running it will block the thread that is waiting for events, so make sure you have a thread dedicated to the event loop you are creating.

The simplest way to create a new event loop is with the `new` method.

```rs
let event_loop = EventLoop::new();
```

This creates an event loop that:

* unblocks every 1s to check for shutdown requests
* supports up to 1024 pending notifications
* delivers up to 64 notifications per tick

Each of these options is configurable using `EventLoop::configured`:

```rs
let event_loop = EventLoop::configured(ReactorConfig {
    io_poll_timeout_ms: 100,
    notify_capacity: 4096,
    messages_per_tick: 128
})
```

In general, the default settings should be acceptable, and support messaging the event loop with zero runtime allocations.
