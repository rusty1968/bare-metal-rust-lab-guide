# Chapter 1

In Rust, as you may know, a library or executable program is called a crate. Crates are compiled using the Rust compiler, rustc. A package is a bundle of one or more crates that provides a set of functionality.

Cargo is the Rust package manager. It is a tool that allows Rust packages to declare their various dependencies and ensure that you’ll always get a repeatable build.

Check whether Cargo is installed by entering the
following in your terminal:

```console
$ cargo --version
```

If you see a version number, you have it! If you see an error, such as `command
not found`, look at the documentation for your method of installation to
determine how to install Cargo separately.


Run the following:
```console
$ cargo new --lib bare-metal-rust
```

We’re passing --lib because we’re making  making a library. This also initializes a new git repository by default. 

```console
$ cd bare-metal-rust
$ tree .
.
├── Cargo.toml
└── src
    └── lib.rs
1 directory, 2 files1 directory, 2 files
```

When we create a lib project using Cargo, we get a src folder, with a lib.rs, and a Cargo.toml. One might then add other files and folder inside src, consisting of the library's modules and functions.

Let’s take a closer look at Cargo.toml:
```console
[package]
name = "bare-metal-rust"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

This is called a manifest, and it contains all of the metadata that Cargo needs to compile your project. 

At this point we have a package which has one library crate. We want to have two crates, a binary and a library.

Create a main.rs file under src

```console
.
├── Cargo.lock
├── Cargo.toml
└── src
    ├── lib.rs
    └── main.rs
```


If a package contains src/main.rs and src/lib.rs, it has two crates: a binary and a library, both with the same name as the package.

Add some code to the main function.
```rust
pub fn main() {
    println!("Welcome, new Rustacean");
}
```

And then run it:

```console
$ cargo run
Welcome, new Rustacean!
```
