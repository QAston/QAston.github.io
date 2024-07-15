---
layout: post
title:  "Simplify your rust error types with Error-Stack"
date:   2024-06-16 20:07:56 +0000
categories: rust error
---

[Error-stack][error-stack] is a rust library for user defined error types. I stumbled upon this library while looking for ways to improve observability in a project I was working on, and in this post I'd like to outline why I believe error-stack's approach is superior compared to other libraries out there, at least for application purposes.

### What does an error type do in rust?

Rust's primary error handling mechanism is the [Result](https://doc.rust-lang.org/std/result/) type, which is an enum with either the success value, or the error value. The type of the error has surprisingly few constraints, for the most part the only type you need to implement to have a usable error is to implement [fmt::Debug](https://doc.rust-lang.org/std/fmt/trait.Debug.html) as it is required by many methods of `Result`. Notably, implementing [std::error::Error](https://doc.rust-lang.org/std/error/trait.Error.html) is not needed to be a valid `Result` error type. At first, this disconnect between `Result` and `std::error::Error` was confusing and disappointing, but in the end this flexibility does open more design opportunities that libraries like error-stack can use. More on that later. To prevent confusion, whenever I'm talking about rust error types, I'm refering to the Result's variant, not the `std::error::Error` trait.

The job of the error types in rust is to propagate all of the information necessary to both handle and debug an error at an appropriate layer of the application. This turns out to be a long list of requirements to implement in practice, among which are:

* provide a distinct type for the function return type
* provide a distinct enum variant for functions with many errors
* store the data needed for handling the error case
* store the data for debugging purposes: line/file location
* merge/nest errors for debugging purposes if the cause of the problem is in another layer
* provide convenience code for constructions and conversions
* implement `fmt::Debug` for panic logging purposes
* implement `std::error::Error` for interop with other error types

The job of an error library is to help you implement all of these, for all your errors, without losing your mind.

### The error library landscape

This [library survey from 2019](https://blog.yoshuawuyts.com/error-handling-survey/) is a decent starting point for choosing a library, but some of the points are no longer relevant thanks to the improvements to the language and library updates. Notably, the survey doesn't include error-stack (it was created later). Fortunately for me the survey covers the currently biggest libraries in the rust's error ecosystem: [anyhow](https://crates.io/crates/anyhow), [thiserror](https://crates.io/crates/thiserror) and [snafu](https://crates.io/crates/snafu), so here I can just focus on the differences between these approaches.

### Issues with error-stack

Error-stack has some issues that fortunately can be worked around with a few snippets. These aren't fundamental issues with the library, but are definitely something to be aware of.

First, the biggest issue for me is that the error stack constructors don't automatically handle nested `std::error::Error`'s, as reported [here](https://github.com/hashintel/hash/issues/2127).

This code, with request middleware returning nested errors:

```rust
struct MyContext;
fn main() {
  let wrapped_client = reqwest::Client::new();
  let client = reqwest_middleware::ClientBuilder::new(wrapped_client).build()
  let response = client
     // nonexistent server
    .post("http://localhost:9999/")
    .body("asdf")
    .send()
    .await
    .change_context(MyContext);

  println!("{:?}", response)
}
```

Should print this output
```
Error: MyContext
├╴at location
│
╰─▶ Request error: error sending request for url (http://localhost:9999/)
├╴backtrace with 28 frames (1)
├╴at location
├╴error sending request for url (http://localhost:9999/)
├╴client error (Connect)
├╴tcp connect error: Connection refused (os error 111)
╰╴Connection refused (os error 111)
```

but it prints this instead (eats the error):

```
Error: IoError
├╴at location
│
╰─▶ Request error: error sending request for url (http://localhost:9999/)
    ├╴at location
    ╰╴backtrace with 29 frames (1)
```

To me this is a major oversight, as reliable observability is very important. I made a workaround for this issue, which works without forking the library, but (embarassingly) forces you to depend on anyhow.

```rust
use error_stack::{Context, IntoReportCompat, Report};

/// Converts `Result<Ok, Err>` into `Result<Ok, Report<Err>>`
/// Workaround for <https://github.com/hashintel/hash/issues/2127>
pub trait IntoReport {
    /// Type of the [`Ok`] value in the [`Result`]
    type Ok;
    /// Converts the [`Err`] variant of the [`Result`] to a [`Report`]
    fn into_report<C: Context>(self, c: C) -> Result<Self::Ok, Report<C>>;
}

impl<T, E: std::error::Error + Send + Sync + 'static> IntoReport for Result<T, E> {
    type Ok = T;

    #[track_caller]
    fn into_report<C: Context>(self, c: C) -> Result<<Result<T, E> as IntoReport>::Ok, Report<C>> {
        IntoReportCompat::into_report(self.map_err(|e| anyhow::anyhow!(e)))
            .map_err(|e| e.change_context(c))
    }
}
```

Second, smaller issue is that error stack doesn't provide any annotations for quickly implementing the Context and Display traits. To automate the tedious bit you can use a simple macro like this:

```rust
/// Derive the `Context` and `Display` trait for a type to be used as error_stack::Context
/// Make sure to add #[derive(Debug)] to the type
/// Example usage:
/// ```no_run
/// #[derive(Debug)]
/// struct ApplicationError;
///
/// c1_std::derive_context!(ApplicationError);
/// ```
#[macro_export]
macro_rules! derive_context {
    ($type_to_derive_impls_for:ty) => {
        impl error_stack::Context for $type_to_derive_impls_for {}
        impl std::fmt::Display for $type_to_derive_impls_for {
            fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
                std::fmt::Debug::fmt(self, f)
            }
        }
    };
}
```

### Other promising approaches I found

```rust

```


### Conclusion

[Error-stack][error-stack] is a huge improvement on the status quo of error handling in rust because of the separation of concerns it introduces. The library implements all of the repetitive error machinery and policy configuration for you, while you just need to provide the custom error functionality specific to the function you're implementing. You get most of the conenience of anyhow combined with most of the control of thiserror while avoiding the bad parts of both (untyped errors and nested error trees).

[error-stack]: https://crates.io/crates/error-stack