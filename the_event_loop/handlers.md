# Handlers

Every time you run the event loop, you can supply it with a `Handler` to call when an event occurs that the consumer registered interest in.

The `Handler` has a method for every kind of event that `mio` supports:

```rs
pub trait Handler<T: Token, M: Send> {
    /// A registered IoHandle has available data to read
    fn readable(&mut self, reactor: &mut EventLoop<T, M>, hint: ReadHint, token: T);

    /// A registered IoHandle is available to write to
    fn writable(&mut self, reactor: &mut EventLoop<T, M>, token: T);

    /// A registered timer has expired
    fn timer(&mut self, reactor: &mut EventLoop<T, M>, token: T);

    /// A message has been delivered
    fn notify(&mut self, reactor: &mut EventLoop<T, M>, msg: M);

    /// A signal has been delivered to the process
    fn signal(&mut self, reactor: &mut EventLoop<T, M>, info: mio::SigInfo);
}
```

When registering interest in an `IoHandle` or when registering a timer, the consumer [provides a `Token`](registering.md) (which wraps a copyable pointer-sized value). This same token is provided to the `Handle`.
