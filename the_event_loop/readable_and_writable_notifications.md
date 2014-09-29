# Readable and Writable Notifications

`mio` provides a readiness-based notification API. This means that you will be notified when an `IoHandle` is available for reading or writing. When you have been notified that an `IoHandle` is ready, you can synchronously read from that handle to get the data.

```rs
pub trait Handler<T: Token, M: Send> {
    /// A registered IoHandle has available data to read
    fn readable(&mut self, reactor: &mut EventLoop<T, M>, hint: ReadHint, token: T);

    /// A registered IoHandle is available to write to
    fn writable(&mut self, reactor: &mut EventLoop<T, M>, token: T);

    // ...
}
```

## Read Hint

The `ReadHint` provides additional information about the likely effects of a `read(2)` call:

```rs
pub enum ReadHint {
    /// Data is available to be read
    DataHint,

    /// The remote end of the socket closed (hung up)
    HupHint,

    /// Reading will result in an error
    ErrorHint,

    /// The IO backend does not provide hints
    UnknownHint
}
```

This information can be useful, but it does not alleviate the handler from checking the return value of `read(2)`, because a change in status can occur between the time the event was delivered and when the `read` call actually took place. It is expected to be used with optimizations that can skip expensive work if an error of hang-up has definitely occurred.

## Usage Pattern

The expected usage pattern is:

* IOs are placed into non-blocking mode when used with `mio` (using `O_NONBLOCK` or equivalent)
* When notified that data is available for reading, the handler `read(2)`s until the `read` call returns `EAGAIN` or `EOF`.
  * Note that the `ReadHint` can proactively notify handlers that `EOF` has been reached, but handlers must still handle the
    possibility that a `read` call returns `EOF`. This is why it's provided as a "hint".
* When notified that an `IoHandle` is available for writing the handle can `write(2)` until the `write` call returns `EAGAIN`.
* When an `IoHandle` is closed, it is automatically removed from the set of interested `IoHandles` (by the kernel). If you
  get an unrecoverable error when attempting an IO operation, you should probably close the file descriptor.
  * Note that if you duplicate an `IoHandle` (via `dup(2)`, for example), you will need to ensure that all duplicates are
    closed.

`mio` provides an edge-triggered, multi-shot API, so you don't need to do anything to receive events when an `IoHandle` becomes ready again. You will receive the very first notification after registration even if the `IoHandle` became ready before registration.
