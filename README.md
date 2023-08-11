# `wherr` Crate Documentation

The `wherr` crate provides utilities to embed where errors originate from by enhancing them with additional file and line number information.

## Table of Contents

- [Installation](#installation)
- [Usage](#usage)
  - [Using the `wherr` procedural macro](#using-the-wherr-procedural-macro)
- [API Reference](#api-reference)
  - [`Wherr`](#wherr)
  - [`wherrapper`](#wherrapper)
  - [`wherr` procedural macro](#wherr-procedural-macro)
- [Contributing](#contributing)
- [License](#license)

## Installation

Add the `wherr` crate to your `Cargo.toml`:

```toml
[dependencies]
wherr = "0.1"
```

## Usage

To understand the benefits of the `wherr` crate, let's first observe the problem it aims to solve:

### Without `#[wherr]`:

```rust
fn add_two(s1: &str, s2: &str) -> Result<i64, Box<dyn std::error::Error>> {
    let radix = 10;
    let i1 = i64::from_str_radix(s1, radix)?;
    let i2 = i64::from_str_radix(s2, radix)?;
    Ok(i1 + i2)
}

fn main() {
    let sum1 = add_two("10", "20").unwrap();
    println!("sum1 = {}", sum1);

    let sum2 = add_two("123", "not a number").unwrap();
    println!("sum2 = {}", sum2);
}
```

Running this code would produce:

```sh
sum1 = 30
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: ParseIntError { kind: InvalidDigit }', wherr/examples/macro.rs:12:47
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

The error message tells us where the `.unwrap()` call happened, but it doesn't pinpoint where the actual error occurred.

### Using the `wherr` procedural macro

By adding `#[wherr]`, the location of the error becomes visible in the error message:

```rust
use wherr::wherr;

#[wherr]
fn add_two(s1: &str, s2: &str) -> Result<i64, Box<dyn std::error::Error>> {
    let radix = 10;
    let i1 = i64::from_str_radix(s1, radix)?;
    let i2 = i64::from_str_radix(s2, radix)?;
    Ok(i1 + i2)
}

fn main() {
    let sum1 = add_two("10", "20").unwrap();
    println!("sum1 = {}", sum1);

    let sum2 = add_two("123", "not a number").unwrap();
    println!("sum2 = {}", sum2);
}
```

The resulting error is:

```sh
sum1 = 30
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error at wherr/examples/macro.rs:7. Original error: ParseIntError { kind: InvalidDigit }', wherr/examples/macro.rs:15:47
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Now we see that the error originates from **wherr/examples/macro.rs:7**.

The `file` and `line` info can also be extracted from the `Wherr` struct,
that wraps the original `Err`.

```rust
    match add_two("123", "not a number") {
        Ok(sum) => {
            println!("sum2 = {}", sum);
        }
        Err(e) => {
            if let Some(wherr) = e.downcast_ref::<wherr::Wherr>() {
                println!(
                    "Error at file: '{}', line: {}. Original error: {}",
                    wherr.file, wherr.line, wherr.inner
                );
            } else {
                println!("Unexpected error: {}", e);
            }
        }
    }
```

```sh
Error at file: 'wherr/examples/macro.rs', line: 7. Original error: invalid digit found in string
```

## API Reference

### `Wherr`

Represents an error that includes file and line number information.

```rust
pub struct Wherr {
    inner: Box<dyn std::error::Error>,
    file: &'static str,
    line: u32,
}
```

Methods:

- `new(err: Box<dyn std::error::Error>, file: &'static str, line: u32) -> Self`: Creates a new `Wherr` error that wraps another error, providing additional context.

### `wherrapper`

This internal utility function is used by the procedural macro to wrap errors with file and line information.

```rust
pub fn wherrapper<T, E>(
    result: Result<T, E>,
    file: &'static str,
    line: u32,
) -> Result<T, Box<dyn std::error::Error>>
```

### `wherr` procedural macro

A procedural macro that auto-wraps errors (using the `?` operator) inside a function with file and line number details.

```rust
#[wherr]
fn some_function() -> Result<(), Box<dyn std::error::Error>> { /* ... */ }
```

## Contributing

If you're interested in contributing to `wherr`, please follow standard Rust community guidelines and submit a PR on our repository.

## License

Please refer to the `LICENSE` file in the root directory of the crate for license details.
