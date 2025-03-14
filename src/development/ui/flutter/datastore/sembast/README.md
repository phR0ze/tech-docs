# Sembast
The Simple Embedded Application Storage database a.ka. Sembast is a NoSQL persistent data store for 
single process applications. It provides very simple single file database that is loaded into memory 
when opened for speed and simplicity. It is 100% Dart and cross platform on both mobile and desktop.

**References**
* [Sembast docs](https://github.com/tekartik/sembast.dart/blob/master/sembast/doc/queries.md)

## Common use cases

### Get non empty store names
You can use the `getNonEmptyStoreNames(db)` function to get a list of names of stores in the database 
that are non-empty.

```dart
```

### Get all records for a store
The recommended way to get all records for a given store is to use the `find` functionality without a 
filter.

<!-- 
vim: ts=2:sw=2:sts=2
-->
