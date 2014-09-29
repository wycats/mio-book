# Interoperability with Rust `std`

The philosophy of `mio` is to support interoperating with objects supplied by Rust standard library APIs as much as possible.

We intend to use elements of the Rust standard library that are compatible with `mio`'s requirements, such as Rust's `mpmc_bounded_queue`. Users of `mio` are intended to run it alongside the Rust standard library, not as a replacement for it. In particular, `mio` is focused on providing a high-performance IO library, and Rust's standard library is much broader in scope.

The Rust team is actively working on factoring its IO libraries into lower-level APIs that expose the system capabilities more directly, and higher level blocking APIs. As they continue this work, we expect to be able to use more of the low-level facilities provided by Rust.

---

At a high level, our goal is to enable more code sharing between applications with very high-performance requirements and applications that are willing to sacrifice some performance for better ergonomics.

Historically, standard IO libraries have been written with a high-level orientation, and sub-ecosystems interested in high performance have been forced to reinvent everything from the ground up, including the protocol layer, to get the performance they need. This results in very little code sharing between the sub-ecosystems, as well as frustrating incompatibilities.

> For examples, see the Twisted (Python) and EventMachine (Ruby) ecosystems.
> Interestingly, because EventMachine was so large in scope, there have been
> multiple subsequent (and incompatible) attempts to try alternate high-level
> approaches that required new rewrites of the low-level protocol libraries.

Our goal is to create a low-level abstraction that can be used to implement protocol-level details, which can then be used by higher-level APIs that make different tradeoffs. We hope that this will allow for more casual experimentation on the tradeoffs, and increased cooperation among people building higher level APIs.

This goal also means that the scope of the core `mio` will remain small and constrained: a "portable epoll", not something more ambitious like `libuv`.

---

Ultimately, we hope that this approach will be considered for inclusion in the Rust standard library if it proves itself.
