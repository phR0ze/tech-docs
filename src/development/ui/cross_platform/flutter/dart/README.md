# Dart

## extends vs implements vs with

* ***extends*** - is a typical OOP class inheritance relationship i.e. if class `b` extends `a` then 
all properties, variables, functions implemented in the base class `a` are also available in the 
derived class `b` and you can override these inherited pieces as per usual in the OOP world.

* ***implements*** - is used to create an implementation of an interface where class `b` implements 
all the functions defined in class `a`. In this case there is no inheritance and instead you are 
implementing a specification.

* ***with*** - is a `mixin` type paradigm. class `b` is mixed with `a` such that `b` is all of `a` 
plus all of `b`.

## Mixins
[Mixins](https://dart.dev/language/mixins) are a way of defining code that can be reused in multiple 
class hierarchies. They are intended to provide member implementations en masse. Mixins can be used 
as an add on to a single class or in an extends situation as well.

**Example direct**
```dart
class Musician with Musical {
  // ···
}
```

**Example with extends**
```dart
class Musician extends Performer with Musical {
  // ···
}
```

**Example with multiple**
```dart
class Musician extends Performer with Musical, Aggressive, Demented {
  // ···
}
```

### Mixin definition
```dart
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

## Tooling

### Create a new project
```bash
$ dart create -t TEMPLATE PROJECT_NAME
```

  
<!-- 
vim: ts=2:sw=2:sts=2
-->
