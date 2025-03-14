# Kotlin <img align="left" width="48" height="48" src="../../data/images/logo_256x256.png">

### Quick links
- [.. up dir](../README.md)
* [Overview](#overview)
* [Primitive Types](#primitive-types)
  * [Boolean](#boolean)
  * [Character](#character)
  * [Literals](#literals)
  * [Numbers](#numbers)
  * [Nullable types](#nullable-types)
* [Control flow](#control-flow)

## Overview
* `val test: Any` type `Any` allows for expressions to return different types
* `kotlin.Unit` is what is returned when there is no value
* Kotlin doesn't allow for declaring multiple variables in a single statement
* Kotlin prefers expressions over statements. This is a modern technique shared by other languages 
like Rust and allows for returning values out of a conditional block.
```kotlin
val bigger = 1000
val smaller = 1
val max = if (bigger > smaller) bigger else smaller
```

## Primitive Types

### Boolean
```kotlin
var exists = false
```

Boolean logic:
```kotlin
if (first || second) {
  println("first or second is true")
}

if (first && !second) {
  println("first is true and second is false")
}
```

### Character
* Char e.g. `var letter = 'A'`
* ASCII `var tab = '\t'`
* Unicode `var infinity = '\u221E'`

### Literals
* Hex e.g. `0xFEDC`
* Binary e.g. `0b101010`

### Numbers
Types can be inferred based on the size and type of number used e.g. `val foo = 1_230.1F`

Oddities:
* Long's require the `L` after the number
* Float requires the `F` after the number else Double is used
* Underscores in numbers are ignored and work well for place separators

* `Byte` 8 bits
* `Short` 16 bits
* `Int` 32 bits
* `Long` 64 bits e.g. 1234L

* `Double` 64 bits
* `Float` 32 bits e.g. 1234.5F

### Nullable types
Kotlin made nullable types special, use `?` suffix to turn any type into nullable. You can also use 
the `?` to make the operation safe.

```kotlin
val person: Person? = null
val greeting: String? = "hello world"
prinln("message: ${greeting?.length}")

// Default to 0 if greeting is null
val len = greeting?.length ?: 0
```

### Strings
```kotlin
val owe = 50
val janet = "I owe Janet \$$owe dollars"
println(janet)
```

## Immutable Variables
Using the `val` declaration makes the value immutable
```kotlin
val foo: Int = 0
```

## Control Flow

### loop
Doesn't have the standard 3 part for loop. Supports modern iteration and standard `break` and 
`contiue`.

```kotlin
for (i in 1..10) {
  println("i = $i")
}

val students = listOf("foo1", "foo2")
for (student in students) {
  println("Current student: $student")
}

for ((i, student) in students.withIndex()) {
  println("Current student: $student, index: $student")
}
```

### when expression
Seems to be basically the same thing as a Rust `match`

* Must be exhaustive
* `else` catch all can be used for non exhaustive lists

## Collections
Fundamental collection types are Lists, Sets and Maps. All come in `Read-only` or `Mutable`. Neither 
has anything to do with `val` or `var`.

### Arrays
Fixed length on declaration

```kotlin
// can be of multiple types
val things = arrayOf(1, 2, 3, "one", "two")

// Must be the same type
val things = arrayOf<Int>(1, 2, 3)
```

### Lists
Mutable length

```kotlin
val names = listOf("foo1", "foo2")
```


