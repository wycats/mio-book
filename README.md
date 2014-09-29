# What is mio?

`mio`, or "metal IO" is a fast, low-level IO library.

# Goals and Requirements

* Fast
* Zero allocations
* A scalable readiness-based API, similar to `epoll` on Linux
* Designed for stack-allocated buffers when possible
* Can multiplex all operations on the minimal number of threads
* A maximal subset of all operations can work in a single-threaded application
* Interoperability with the Rust Standard Library
* Works on all platforms supported by Rust*

When we say "fast", we mean that our goal is to avoid regressing on the performance capabilities of the low-level platform, even while providing an abstraction.

Among other things, this means providing a readiness-based API, which can be used to build high-performance proxies on the highest performing platforms. It also means avoiding allocations that would not be necessary when working with the low-level platform capabilities.

> *`mio` plans to support Windows, but does not yet. The current abstractions are designed with an eye towards Windows support.

The `mio` library is intended to be interoperable with the Rust standard library. It should always be able to run together with `libstd`.

Currently, not all of the standard Rust IO provides lowering operations (from `File` to `Fd` for instance), so it may not always be possible *today* to construct a high-level IO using `libstd` and register it with the `mio` event loop.

The Rust core team is actively working on exposing across-the-board lowering operations for Rust 1.0, and generally exposing the low-level IO underpinnings of the standard library, which should further improve interoperability with `mio`.

# The `mio` API

From a high-level, `mio` provides a way to register interest in certain events, and a way to wait until an event has occurred.

```rs
use mio::{EventLoop, io, buf};

fn main() {
    start().assert("The event loop could not be started");
}

fn start() -> MioResult<EventLoop> {
    // Create a new event loop. This can fail if the underlying OS cannot
    // provide a selector.
    let event_loop = try!(EventLoop::new());

    // Create a two-way pipe.
    let (mut reader, mut writer) = try!(io::pipe());

    // the second parameter here is a 64-bit, copyable value that will be sent
    // to the Handler when there is activity on `reader`
    try!(event_loop.register(&reader, 1u64));

    // kick things off by writing to the writer side of the pipe
    try!(writer.write(buf::wrap("hello".as_bytes())));

    event_loop.run(MyHandler)
}

struct MyHandler;

impl Handler for MyHandler {
    fn readable(&mut self, _loop: &mut EventLoop, token: u64) {
        println!("The pipe is readable: {}", token);
    }
}
```

One thing you should notice right away is that the API is pretty low-level. It uses a 64-bit token to communicate with the handler, and does not use a separate Handler (or a closure) for each callback.

This is intentional. The `mio` API is designed to allow such APIs to be built on top, but to offer a simple, fast primitive that is agnostic to the specifics of higher-level APIs.

We expect other libraries, including ones written by the authors of `mio`, to provide higher-level, more opinionated APIs on top of this abstraction. If carefully written, higher-level libraries will likely be able to maintain very good performance with better ergonomics, thanks to Rust's philosophy of zero-cost abstractions.

With `mio`, our goal is to start with a high-performance, low-level API that is a minimal, portable shim on top of platform capabilities.

By keeping the core primitive small and aiming for a modular ecosystem of high-level abstractions, we hope to avoid the all-too-common fragmentation that occurs when ergonomic considerations compete with the requirements of performance-critical applications. Instead, we hope that both kinds of applications can share a common implementation of the protocol layer.

# What Does `mio` Provide?

The base `mio` layer will provide APIs needed to register interest in kernel-level operations. In general, this means that `mio` provides all of the facilities provided by modern `epoll`, implemented efficiently and portably.

You can register interest in all of the following, blocking only a single thread when waiting for any combination of these:

* There is readable data on a file descriptor
* A file descriptor is ready for writing
* A general-purpose message is available
* A registered timer has expired
* A Unix signal has occurred

On modern Linux, Darwin or BSD, only a single thread is necessary to wait for any combination of these events. On older Linux (without `signalfd` or `eventfd`) `mio` will use an additional background thread to wait for signals.

## What about Windows?

The approach we plan to take for Windows is to emulate the Unix readiness API using a pre-allocated slab of memory, and to use IOCP (`GetQueuedCompletionStatusEx` and `PostQueuedCompletionStatus`) for multiplexing. This will allow for an allocation-free, stack-allocated-buffer approach on Windows, which should get reasonable performance.

Most importantly, it allows `mio` to expose the low-level readiness API available on Linux in a portable fashion. This should allow `mio` to be used for a very broad class of applications, even when maximum performance is a requirement.

This is in contrast to the two approaches normally taken by portable abstractions:

* One approach, taken by `libuv`, is to provide an abstraction over the completion model provided by Windows. This provides reasonable performance in most cases, but is not suitable for applications that require maximum performance, like high-performance, low-memory proxies. This is a reasonable approach, but it does not meet our performance goals, nor our goal of allowing stack-allocated buffers in performance-critical cases.
* Another approach, taken by `libev`, is to provide an abstraction over the readiness model on Linux, and to use older APIs, like `select`, on Windows. This approach is fundamentally un-scalable on Windows, and is indeed what triggered the creation of `libuv` in the first place.

Our goal is to provide an API that is maximally efficient on Unix, and still very high performance on Windows, by using Windows' high-performance IOCP API to emulate the Unix API in user-space.

# Further Work and Proposals

We understand that many people will not want to use the low-level primitives provided by `mio` directly. We have designed `mio` to serve as a low-level primitive for higher-level abstractions that people can build outside of the core `mio` repository, and we intend to experiment with such APIs ourselves.

To be more specific, the scope of the core `mio` library is restricted to providing facilities that are available in the modern Linux kernel, but in a portable and efficient fashion. This is why it includes facilities like high-performance timers (like `timerfd`), a message queue (like `eventfd`) and signal support (like `signalfd`).

If you wish to propose changes to the API, we ask that you consider whether the API proposal you have in mind:

* could be implemented as a higher level of abstraction that depends on `mio`. If you discover that you cannot implement a higher-level API on top of `mio` efficiently, we want to know. Proposals that help people build higher-level APIs more efficiently are highly desired at this point, and it may be harder to change `mio` later.
* satisfies all of the goals listed at the beginning of this document. For example, an API that requires allocations, or that introduces `Rc` into the core, do not meet the zero-allocation requirement.

If you have a proposal for an improvement, please feel free to open up an issue with a design proposal, or a scenario you are having trouble using `mio` for.
