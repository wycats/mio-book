# Timers

`mio` provides a high-performance timer implementation that can be used to wake up the event loop after a certain amount of time. It is optimized for relatively low-resolution timers (in the hundreds of milliseconds).

To register a timer, use `EventLoop::timer`. Like `EventLoop::register`, `timer` takes a `Token`, which will be passed to the handler when the timer expires. [More on Tokens](registering.md#tokens)

```rs
event_loop.timer(1000, 10u);
```

> Question for Carl: High resolution timers?
