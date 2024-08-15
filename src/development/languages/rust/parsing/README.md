# Parsing

### Quick links
* [nom](#nom)

## nom
[nom](https://crates.io/crates/nom) is a byte-oriented, zero-copy, parser coombinators library. It 
enables the creation of safe parsers without hogging memory or compromising performance. It relies on 
Rust's strong typing and memory safety to produce parsers that ar both correcdt and performant.

**References**
* [Nom 7 tutorial](https://tfpk.github.io/nominomicon/)

nom's premise is that you can combine simple parsers with combinators to build more complex parsers. 
Parsers take an input value and return a result. 

### tag
A tag in nom is a collection of bytes that can be recognized. There are multiple different 
definitions for tags. tags can be checked with our without case e.g. `tag` vs `tag_no_case`.

### character classes
* `alpha0`: Recognizes zero or more lowercase and uppercase alphabetic characters: /[a-zA-Z]/.
* `alpha1`: Does the same as `alpha0` but returns at least one character
* `alphanumeric0`: Recognizes zero or more numerical and alphabetic characters: /[0-9a-zA-Z]/.
* `alphanumeric1`: Does the same as `alphanumeric0` but returns at least one character
* `digit0`: Recognizes zero or more numerical characters: /[0-9]/.
* `digit1`: Does the same as `digig0` but returns at least one character
* `multispace0`: Recognizes zero or more spaces, tabs, carriage returns and line feeds.
* `multispace1`: Does the same as `multispace0` but returns at least one character
* `space0`: Recognizes zero or more spaces and tabs.
* `space1`: Does the same as `space0` but returns at least one character
* `line_ending`: Recognizes an end of line (both \n and \r\n)
* `newline`: Matches a newline character \n
* `tab`: Matches a tab character \t
