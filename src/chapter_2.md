# Chapter 2

In Chapter1, we created a Rust package using Cargo. The package is comprised of a binary crate and a library crate. Please, refer to Chapter1 if you have not crated the bare-metal-rust package.

The astute learner may have already noticed the package is not actually suitable to run in bare metal, since it is printing to the standard output. This chapter will take you to the preliminary steps to be able to build your package in a hosted environment and a bare metal environment. This is 
part of an embedded test driven design (TDD) strategy to run hardware independent code on your host deveopment system. 

This chapter will introduce you to the concept of features. Cargo “features” provide a mechanism to express conditional compilation and optional dependencies. A package defines a set of named features in the [features] table of Cargo.toml, and each feature can either be enabled or disabled. Features for the package being built can be enabled on the command-line with flags such as --features. Features for dependencies can be enabled in the dependency declaration in Cargo.toml. 

The [features] section
Features are defined in the [features] table in Cargo.toml. Each feature specifies an array of other features or optional dependencies that it enables.
Let's  illustrate how features could be used to conditionally compile code that runs only on hosted environment.

Add a features section to the Cargo.toml.

```toml
[features]
# Defines a feature named `std` that does not enable any other features.
std = []
# Defines a feature named `mbed` that does not enable any other features.
mbed = []
```

And then run, to make sure we did not introduce any errors:
```console
$ cargo run
Welcome, new Rustacean!
```
## Hosted versus bare metal environment
Rust comes with a standard library called std, which includes code for a wide variety of common tasks, from standard data structures to networking, from multi-threading support to file I/O. For convenience, many of the items from std are automatically imported into your program, via the prelude: a set of common use statements.

Rust also supports building code for environments where it's not possible to provide this full standard library, such as bootloaders, firmware, or embedded platforms in general. Crates indicate that they should be built in this way by including the #![no_std] crate-level attribute at the top of src/lib.rs and/or src/main.rs.

Add the code below to src/lib.rs
```rust
//! My `no_std` compatible crate.
#![no_std]
```

And see the build fail.
```console
$ cargo build
Compiling bare-metal-rust v0.1.0 (/home/rusty1968/rust/bare-metal-rust-guide/bare-metal-rust)
error: cannot find macro `println` in this scope
 --> src/main.rs:3:5
  |
3 |     println!("Welcome, new Rustacean!");
  |     ^^^^^^^

error: could not compile `bare-metal-rust` due to previous error
```

Let's try to fix it. Remove the offending code and run cargo build again.


```console
$ cargo build 
   Compiling bare-metal-rust v0.1.0 (/home/rusty1968/rust/bare-metal-rust-guide/bare-metal-rust)
error: `#[panic_handler]` function required, but not found

error: language item required, but not found: `eh_personality`
  |
  = note: this can occur when a binary crate with `#![no_std]` is compiled for a target where `eh_personality` is defined in the standard library
  = help: you may be able to compile for a target that doesn't need `eh_personality`, specify a target with `--target` or in `.cargo/config`

error: could not compile `bare-metal-rust` due to 2 previous errors
```

The compiler is telling you that it is missing a #[panic_handler] function and a language item.

```
error: `#[panic_handler]` function required, but not found
error: language item required, but not found: `eh_personality`
```

A panic handler is the function invoked when your Rust program encounters an unrecoverable error. As a bare metal Rustacean you should write panic-free code. You will learn more about how Rust handles recoverable errors  later. 

Rust panic mechanism is not exactly the same as exceptions in C++, since C++ exceptions are used for both recoverable and unrecoverable errors.

The #[panic_handler] attribute is used to define the behavior of panic! in #![no_std] applications. The #[panic_handler] attribute must be applied to a function with signature fn(&PanicInfo) -> !

The trivial panic handler function below just enter an infinite loop when the progoram encounters an unrecoverable error. 
Enter the code snippet below and run "cargo build" again. You should see that only the eh_personality error remains.

```rust
#[panic_handler]
fn panic(_: &PanicInfo) -> ! {
    loop {}
}
```

We wil take care of the remaining error but first let's introduce the concept of panic strategy. In Rust, there are two panic strategies :abort and unwind. 

By default, when a panic occurs, the program starts unwinding, which means Rust walks back up the stack and cleans up the data from each function it encounters. If in your project you need to make the resulting binary as small as possible, you can switch from unwinding to aborting upon a panic by adding panic = 'abort' to the appropriate [profile] sections in your Cargo.toml file.

```toml
[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```