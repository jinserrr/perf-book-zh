# Standard Library Types

It is worth reading through the documentation for common standard library
types—such as [`Vec`], [`Option`], [`Result`], and [`Rc`]/[`Arc`]—to find interesting
functions that can sometimes be used to improve performance.

[`Vec`]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[`Option`]: https://doc.rust-lang.org/std/option/enum.Option.html
[`Result`]: https://doc.rust-lang.org/std/result/enum.Result.html
[`Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

It is also worth knowing about high-performance alternatives to standard
library types, such as [`Mutex`], [`RwLock`], [`Condvar`], and
[`Once`].

[`Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html
[`RwLock`]: https://doc.rust-lang.org/std/sync/struct.RwLock.html
[`Condvar`]: https://doc.rust-lang.org/std/sync/struct.Condvar.html
[`Once`]: https://doc.rust-lang.org/std/sync/struct.Once.html

## `Vec`

[`Vec::remove`] removes an element at a particular index and shifts all
subsequent elements one to the left, which makes it O(n). [`Vec::swap_remove`]
replaces an element at a particular index with the final element, which does
not preserve ordering, but is O(1).

[`Vec::retain`] efficiently removes multiple items from a `Vec`. There is an
equivalent method for other collection types such as `String`, `HashSet`, and
`HashMap`.

[`Vec::remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.remove
[`Vec::swap_remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.swap_remove
[`Vec::retain`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.retain

## `Option` and `Result`

[`Option::ok_or`] converts an `Option` into a `Result`, and is passed an `err`
parameter that is used if the `Option` value is `None`. `err` is computed
eagerly. If its computation is expensive, you should instead use
[`Option::ok_or_else`], which computes the error value lazily via a closure.
For example, this:
```rust
# fn expensive() {}
# let o: Option<u32> = None;
let r = o.ok_or(expensive()); // always evaluates `expensive()`
```
should be changed to this:
```rust
# fn expensive() {}
# let o: Option<u32> = None;
let r = o.ok_or_else(|| expensive()); // evaluates `expensive()` only when needed
```
[**Example**](https://github.com/rust-lang/rust/pull/50051/commits/5070dea2366104fb0b5c344ce7f2a5cf8af176b0).

[`Option::ok_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or
[`Option::ok_or_else`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or_else

There are similar alternatives for [`Option::map_or`], [`Option::unwrap_or`],
[`Result::or`], [`Result::map_or`], and [`Result::unwrap_or`].

[`Option::map_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map_or
[`Option::unwrap_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or
[`Result::or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.or
[`Result::map_or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_or
[`Result::unwrap_or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap_or

## `Rc`/`Arc`

[`Rc::make_mut`]/[`Arc::make_mut`] provide clone-on-write semantics. They make
a mutable reference to an `Rc`/`Arc`. If the refcount is greater than one, they
will `clone` the inner value to ensure unique ownership; otherwise, they will
modify the original value. They are not needed often, but they can be extremely
useful on occasion.
[**Example 1**](https://github.com/rust-lang/rust/pull/65198/commits/3832a634d3aa6a7c60448906e6656a22f7e35628),
[**Example 2**](https://github.com/rust-lang/rust/pull/65198/commits/75e0078a1703448a19e25eac85daaa5a4e6e68ac).

[`Rc::make_mut`]: https://doc.rust-lang.org/std/rc/struct.Rc.html#method.make_mut
[`Arc::make_mut`]: https://doc.rust-lang.org/std/sync/struct.Arc.html#method.make_mut

## `Mutex`, `RwLock`, `Condvar`, and `Once`

The [`parking_lot`] crate provides alternative implementations of these
synchronization types that are smaller, faster, and more flexible than those in
the standard library. The APIs and semantics of the `parking_lot` types are
similar but not identical to those of the equivalent types in the standard
library.

[`parking_lot`]: https://crates.io/crates/parking_lot
