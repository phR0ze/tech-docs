# Code Generation <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Code generation tooling has started to become a norm in development. Often best practice would 
required more boiler plate code than is useful to write by hand. So many packages have started 
including annotations that will trigger code generation to create additional boiler plate code for 
your projects to avoid this.

In Dart and thus Flutter the fundamental tool for code generation is [Build](https://github.com/dart-lang/build)

### Quick links
* [Freezed](#freezed)
  * [Get started with Freezed](#get-started-with-freezed)
  * [What just happened](#what-just-happened)
* [Build](#build)
  * [Build runner](#build-runner)
  * [Builder](#builder)

## Freezed
Freezed is a code-generation package that helps you to create data classes quickly and avoid the 
tedious boiler plate necessary for best practices.

**References**
* [Data modeling with Freezed](https://dev.to/carlomigueldy/data-modeling-with-flutter-using-freezed-package-4p69)

### Get started with Freezed
Note: don't make the mistake of naming your test project `freezed` as this will cause the flutter 
dependency resolution to break as it thinks freezed is dependent on itself.

1. Create a new flutter project test with
   ```bash
   $ flutter create --platform=android,linux pkg_freezed
   ```

2. Add the `freezed_annotation` dependency
   ```bash
   $ flutter pub add freezed_annotation
   $ flutter pub add json_annotation
   ```

3. Add the `freezed` and `build_runner` dev dependencies
   ```bash
   $ flutter pub add --dev build_runner
   $ flutter pub add --dev freezed
   $ flutter pub add --dev json_serializable
   ```

4. Create an example data class called `model/person.dart`
   ```dart
   import 'package:freezed_annotation/freezed_annotation.dart';

   part 'person.freezed.dart';

   // Provides JSON code generation for toMap and fromMap methods
   part 'person.g.dart';
   
   @freezed
   class Person with _$Person {
     const factory Person({
       required String firstName,
       required String lastName,
       required int age,
     }) = _Person;
   
     // Provides JSON code generation for toMap and fromMap methods
     factory Person.fromJson(Map<String, Object?> json) => _$PersonFromJson(json);
   }
   ```

5. Run the builder
   ```bash
   $ dart run build_runner build --delete-conflicting-outputs
   ```

### What what just happened
1. We needed to add the line `import 'package:freezed_annotation/freezed_annotation.dart';` to make 
   the `@freezed` annotation be recognized by the system.
2. The `part 'person.freezed.dart'` told freezed
3. The `part 'person.g.dart'` told freezed
4. The `@freezed` annotation told freezed to generate code for your class
5. The `class Person with _$Person {` defines the name of our class Person as per usual then 
   defines a mixin class to include as part of the Person class. This allows freezed to generate 
   code for the Person class that we can then use in our application.

Freezed will then generate
* `toString()` method
* `==` operator
* `hashCode` getter
* `copyWith()` method for immutable creation
* `toJson()` method

### Add a default value in the constructor
```dart
const factory MyClass({
  required String name,
  @Default(false) bool isPremium,
}) = _MyClass
```

Rebuild after additions with
```bash
# i.e delete the existing files first then generate new
$ dart run build_runner build --delete-conflicting-outputs
```

## Build
There are a number of other packages associated with Dart's build tooling.

* ***build*** which defines the interface for creating a `builder` which is a way for other build 
ecosystems or packages to generate code.

* ***build_config*** which provides support for parsing `build.yaml` used by `build_runner`

* ***build_modules*** provides support for discovering sub-modules within packages

* ***build_resolvers*** implements teh `Resolver` interface to use the analyzer during build steps

* ***build_runner*** provides utilities to enact builds and a way to automatically run builds based 
on configuration

* ***build_test*** stub implementation for classes in `Build` and some utilities for running 
instances of builds and checking their outputs.

* ***build_web_compilers*** provides the `dart3js` and `dartdevc` packages

### Build runner
The `build_runner` is a Dart written and supported development package providing a common way for 
other packages to have the code they need generated. Other languages use reflection (which has 
performance concerns) or macros (which Dart doesn't support), but Dart uses the build runner pattern 
to fill this need.

**References**
* [Dart build runner](https://dart.dev/tools/build_runner)
* [build runner package](https://pub.dev/packages/build_runner)

`build_runner` is based on the `builders` concept and uses the Dart build system to generate output 
as needed. This generation of output can be as arbitrarily complex as is needed and is leveraged to 
generation, transformations and much more.

To use the build runner system add a dev dependency on it in your project:
```bash
$ dar pub add dev:build_runner
```

Invoking the build runner is done as needed to generate your project code.
```bash
$ dart run build_runner build
```

### Create a builder
A `Builder` is where you'd write your logic that will generate the code you want. Build config and 
build runner are then used to read your builder and run it in the end user's project.

**References**
* [Conquering code generation](https://creativebracket.com/conquering-code-generation-in-dart-1/)

1. Add dependencies
   ```bash
   $ dart pub add build
   $ dart pub add dev:build_config
   $ dart pub add dev:build_runner
   ```

2. Create a `build.yaml` file in the root of your project e.g.
   ```yaml
   targets:
     $default:
       builders:
         code_generators|copyBuilder:
           generate_for:
            - lib/*
           enabled: True
   
   builders:
     copyBuilder:
       import: 'package:code_generators/code_generators.dart'
       builder_factories: ['copyBuilder']
       build_extensions:
         .txt:
           - .copy.txt
       build_to: source
       auto_apply: dependents
   ```



<!-- 
vim: ts=2:sw=2:sts=2
-->
