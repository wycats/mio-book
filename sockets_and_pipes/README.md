# Sockets and Pipes

`mio` provides convenience APIs for working with sockets and pipes that are designed to work with the `EventLoop`.

This includes wrapper structs (`TcpSocket`, `TcpAcceptor`, `UnixSocket`, `PipeReader`, `PipeWriter`) and methods on the `EventLoop` itself for connecting to a socket and listening for new connections on an acceptor.

## Event Loop Socket APIs

The `EventLoop` comes with APIs for connecting and listening to sockets.

```rs
/// Socket can be TcpSocket, TcpAcceptor, UnixSocket, etc. The Handler's
/// `writable` method will be notified with `Token` when the socket has
/// connected.
pub fn connect<S: Socket>(&mut self, io: &S, addr: &SockAddr, token: T) -> MioResult<()>;

/// IoAcceptor is implemented for TcpAcceptor. The Handler's `readable` method
/// will be notified with `Token` when a connection is accepted.
pub fn listen<T, A: IoAcceptor<T>>(&mut self, io: &A, backlog: uint, token: T) -> MioResult<()>;
```

An example of using `listen` to listen for new connections:

```rs
use mio::{TcpSocket, EventLoop, SockAddre};

fn start_server(event_loop: &mut EventLoop, addr: &SockAddr) -> MioResult<()> {
    // Create a socket for the server
    let srv = try!(TcpSocket::v4());

    // Bind it to the supplied address
    let srv = try!(srv.bind(addr));

    // tell the event loop to notify us when a connection occurs with token 0u
    event_loop.listen(&srv, 256, 0u)
}
```

An example of using `connect` to connect a client socket to an address:

```rs
fn start_client(event_loop: &mut EventLoop, addr: &SockAddr) -> MioResult<()> {
    // Create a socket for the client
    let sock = try!(TcpSocket::v4());

    // tell the the event loop to connect to the address, and notify us when the
    // connection succeeds with token 1u
    event_loop.connect(&sock, addr, 1u)
}
```

A fully working example of an echo server using these APIs is included as a `mio` test.

> TODO: Link to the test

---

> The rationale for including these APIs on the `EventLoop` itself is that it is
> possible for `connect(2)` and `listen(2)` to return, having synchronously
> connected. This means that there is a possible race between connecting a
> socket and registering interest in it. To work around this, every consumer of
> a mio event loop would need to check whether `connect` succeeded synchronously,
> and do exactly the same work as it would do in the handler. To simplify this
> pattern, you can `connect` through the `EventLoop`, and mio will guarantee
> that your handler is notified even if the call succeeded synchronously.
>
> **TL;DR** these APIs allow you to reliably handle connections in your Handler
> and not have to worry about races.
