# Errors

### Quick links
* [Overview](#overview)
* [Create a custom error type](#create-a-custom-error-type)
* [Libs](#libs)
  * [thiserror](#thiserror)

## Overview
Rust allows you to create your own `Err` types using the `Error trait`. The Error trait allows for 
chaining errors together for more context via the `source` function.

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

**Desired Anyhow example error output**
```
Error: error reading `Blocks.txt`

Caused by:
	0: invalid Blocks.txt data on line 223
	1: one end of range is not a valid hexidecimal integer
	2: invalid digit found in string
```

### Use a struct type for your base error
I'd argue that a struct is the most useful error base type as it allows for inner fields that will 
allow us to capture useful information about the error.

**Example error**
```rust
#[derive(Debug)]
#[non_exhaustive]                         // allow for future error field additions
pub struct JpegParseError {
    data: Box<[u8]>,                      // error specific data of a type useful for your error
    kind: JpegParseErrorKind,             // extensible kind messaging
    source: Option<JpegParseErrorSource>, // optional extensible source error
}
impl fmt::Display for JpegParseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match &self.kind {
            JpegParseErrorKind::JpegSegmentMarkerInvalid => write!(f, "JPEG segment marker invalid")?,
            JpegParseErrorKind::JpegSegmentLengthInvalid => write!(f, "JPEG segment length invalid")?,
            JpegParseErrorKind::JpegSegmentDataInvalid => write!(f, "JPEG segment data invalid")?,
        };

        // Display additional error data if available
        if self.data.len() > 0 {
            write!(f, " {:x?}", self.data)?;
        };
        Ok(())
    }
}
impl Error for JpegParseError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match &self.source {
            Some(JpegParseErrorSource::Read(e)) => Some(e),
            _ => None,
        }
    }
}

#[derive(Debug)]
#[non_exhaustive]
pub enum JpegParseErrorKind {
    JpegSegmentMarkerInvalid,
    JpegSegmentLengthInvalid,
    JpegSegmentDataInvalid,
}

#[derive(Debug)]
#[non_exhaustive]
pub enum JpegParseErrorSource {
    Read(std::io::Error),
}
```

* Per our guidelines use `#[non_exhaustive]` to allow for adding additional fields later without 
breaking downstream API users.
* `data` can be any number of error values, named anything you'd like that are useful for context
* `kind` is most useful as a enum of different error kinds that then have specific messaging
* `source` is an optional enum of different underlying error types that may have caused the issue

This pattern allows for a very extensible way to add new error messaging and additional underlying 
error types.


## Libs

### thiserror
[thiserror](https://crates.io/crates/thiserror) is a library providing a derive macro to facilitate 
simpler error creation in Rust.
