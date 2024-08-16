# Errors

### Quick links
* [Overview](#overview)
* [Create a custom error type](#create-a-custom-error-type)
* [Libs](#libs)
  * [thiserror](#thiserror)

## Overview
Rust allows you to create your own `Err` types using the `Error trait`.

**References**
* [Andrew Gallant's error idioms](https://blog.burntsushi.net/rust-error-handling/)
* [Guidelines for Good Errors](https://sabrinajewson.org/blog/errors#guidelines-for-good-errors)

## Create a custom error type
**Guidelines**
* Error types should be located near to their unit of fallibility
  * `create::Error` or `errors.rs` is an antipattern
* Error types should be modular i.e. don't use single enum to lump them all together
  * Error types should be nested as needed and leverage the `.source()` method to chain
* Error types should include a kind if there are multiple ways the single error can occur
* Error types should be extensible i.e. attributed with `#[non_exhaustive]`
  * This allows you to add new cases without breaking downstream

**Anyhow example error output**
```
Error: error reading `Blocks.txt`

Caused by:
	0: invalid Blocks.txt data on line 223
	1: one end of range is not a valid hexidecimal integer
	2: invalid digit found in string
```


## Libs

### thiserror
[thiserror](https://crates.io/crates/thiserror) is a library providing a derive macro to facilitate 
simpler error creation in Rust.
