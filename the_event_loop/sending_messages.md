# Sending Messages

You can send a message to a `mio` event loop, and it will wake up and deliver the message to the handler.

When you create a Reactor, you decide on a kind of event that it can receive. The only restriction on event types is that they must be `Send`. You can use an enum to support multiple kinds of messages. Applications that are willing to accept additional allocations can event use `Box<MyTrait + Send>` for more flexibility.

```rs
/// The implementation of mio's Reactor with its constraints
impl<T: Token, M: Send> EventLoop<T, M> {
    // ...
}
```

To send a message to an `EventLoop`, first call its `channel` method, which will give you a sender to write messages to.

```rs
// An EventLoopSender<M>, using the M from EventLoop<T, M>
let sender = event_loop.channel();

// later...
sender.send(message);
```

When a message is sent to the event loop, it unblocks the loop and delivers the message to the handler.

Under the hood, the notification mechanism tries to avoid system calls where possible; the event loop checks for pending messages before going back to sleep. Among other things, this allows handlers to efficiently deliver messages to the event loop without incurring the cost of a system call.

## Size Limits

Because `mio` avoids runtime allocations, the event loop is configured with a maximum number of messages that can be pending at any time. By default, this number is 1024, which will result in preallocating a vector whose size is 1024 × `size_of::<M>()`.

If a sender attempts to push more messages than this limit, `mio` will return an error, and the sender can try again later.

It is possible to build abstractions on top of `EventLoopSender` that:

* cause the sender to block if the queue is full
* add buffering for messages that exceed the limits, at the cost of possible runtime allocations
