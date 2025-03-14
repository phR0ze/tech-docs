# Datastore <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Storing user data locally in our applications is an essential part of delivering a faster seamless 
user experience. There are a number of good options available for local databasse management.

**References**
* [Local Databse comparison](https://medium.com/@tahavoncelik/flutter-local-databases-and-comparisons-hive-sembast-sqflite-and-more-5d402390d1f8)

## Sembast
Sembast is a simple and lightweight local database solution.

**Key points**
* NoSQL JSON document store
* Geared for reactive apps as you can listen and trigger actions when docs change
* Async operations and simple init `final db = await databaseFactoryIo.openDatabase(dbPath);`
* CRUD is a little more convoluted than Hive

**Target**
* Small to medium sized apps
* Apps with simple storage needs

**Pros**
* Cross platform
* Simple structure
* Fast and efficient

**Cons**
* No complex query support
* Updating existing databases can be challenging

### Add Sebast to your project

### Opening a database
```dart
// File path to a file in the current directory
String dbPath = 'sample.db';

// We use the database factory to open the database
DatabaseFactory dbFactory = databaseFactoryIo;
Database db = await dbFactory.openDatabase(dbPath);
```

## Hive
Hive is a highly popular and fast local database solution.

**Key points**
* Key-value db written in pure Dart with no native dependencies
* Initializing the database is done immediately with `Hive.init(dbPath);`
* Async operations e.g. `final box = await Hive.openBox('users');`
* CRUD is simple
* Martial and unmartial objects from maps

**Target**
* Small to medium sized apps
* Apps with simple storage needs

**Pros**
* Cross platform
* Fast and efficient
* Easy integration
* Object oriented

**Cons**
* No support for complex queries
* Updating existing databases can be challenging

## Sqflite
Sqflite is a popular local database solution built on top of SQLite.

**Key points**
* SQL db
* Raw async SQL commands

**Target**
* Medium to large apps
* Apps with complex storage needs

**Pros**
* Fast and efficient
* Supports complex queries

**Cons**
* Cross platform on desktop is a bit wonky with FFI to system lib
* Performance might be a concern with complex queries

<!-- 
vim: ts=2:sw=2:sts=2
-->
