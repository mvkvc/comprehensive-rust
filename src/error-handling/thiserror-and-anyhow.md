---
minutes: 5
---

# `thiserror`  and `anyhow`

The [`thiserror`](https://docs.rs/thiserror/)  and [`anyhow`](https://docs.rs/anyhow/)
crates are widley used to simplify error handling. `thiserror` helps
create custom error types that implement `From<T>`. `anyhow` helps with error
handling in functions, including adding contextual information to your errors.

```rust,editable,compile_fail
use anyhow::{bail, Context, Result};
use std::{fs, io::Read};
use thiserror::Error;

#[derive(Clone, Debug, Eq, Error, PartialEq)]
#[error("Found no username in {0}")]
struct EmptyUsernameError(String);

fn read_username(path: &str) -> Result<String> {
    let mut username = String::with_capacity(100);
    fs::File::open(path)
        .with_context(|| format!("Failed to open {path}"))?
        .read_to_string(&mut username)
        .context("Failed to read")?;
    if username.is_empty() {
        bail!(EmptyUsernameError(path.to_string()));
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    match read_username("config.dat") {
        Ok(username) => println!("Username: {username}"),
        Err(err)     => println!("Error: {err:?}"),
    }
}
```

<details>

* The `Error` derive macro is provided by `thiserror`, and has lots of useful
  attributes like `#[error]` to help define a useful error type.
* `anyhow::Result<V>` is a type alias for `Result<V, anyhow::Error>`.
* `anyhow::Error` is essentially a wrapper around `Box<dyn Error>`. As such it's again generally not
  a good choice for the public API of a library, but is widely used in applications.
* Actual error type inside of it can be extracted for examination if necessary.
* Functionality provided by `anyhow::Result<T>` may be familiar to Go developers, as it provides
  similar usage patterns and ergonomics to `(T, error)` from Go.

</details>
