# State Management
Bloc and Provider are the two most populate state management systems in the Flutter ecosystem. I'll 
be examining the Bloc system here.

### Quick links
* [Comparison](#comparison)
* [Riverpod](#riverpod)
* [Bloc](#bloc)
  * [VSCode Extension](#vscode-extension)
  * [Core Concepts](#core-concepts)

## Overview
Applications frequently need to fetch data from disk, backend servers or the network asynchronously. 
Caching that async data locally to avoid re-executing the call again and handling all the complexity 
around this such as infinite lists, i.e. fetching while scrolling, debouncing async requests, offline 
mode, pull to refresh, search as we type etc... is what state management systems like Bloc and 
Riverpod were created to solve.

## Comparison
I'm going to implement a simple todo in both to sort out my take on the subject.

**References**
* [Riverpod or Bloc](https://www.youtube.com/watch?v=EPVKdverFuw)
* [River pod or Bloc - site](https://mobileappcircular.com/bloc-vs-riverpod-making-the-right-choice-for-your-flutter-app-5feb4486ac4)
* [Bloc vs RiverPod](https://www.xavor.com/blog/bloc-vs-riverpod/)

**General**
* Bloc
  * Flutter favorite and MIT license
  * Last published 6 months ago, 2542 likes, 140 pub points, `11.1k stars`, 3.3k forks
  * Has a VSCode extension to assist in creation
  * Focus on scalability and efficiency
    * well suited for large scale apps
    * async UI event handling with state updates in real-time
  * Separates out the business logic from the UI allowing for better testing
  * Naming conventions are not intuitive
  * Plethora of boiler plate code
  * Complex with a steep learning curve
  * future flutter compatibility could be affected because of its strict architecture
* Riverpod
  * Flutter favorite and MIT license
  * Same creator as the Provider package i.e. 2.0
  * Last published 15 days ago, 2928 likes, 140 pub points, `5.3k stars`, 824 forks
  * state management is achieved through `Provider - a Flutter integrated state managment solution`.
    * seems a little more integrated with Flutter than Bloc
    * more granularity over rebuilds
  * less overhead with event definitions
  * more flexible and intuitive with simplified syntax and naming
  * aligns more closely with flutter making it easier to update and maintain
  * a more modern take on state managment than bloc

**Dependency Injection**
* Bloc
  * uses the `BlocProvider` to wrap your whole app
* Riverpod
  * uses the `ProviderScope` to wrap your whole app

**State Notifications**  
* Bloc
  * uses the Cubit to handle state notifications
* Riverpod
  * uses the StateNotifier which is basically identical to the Cubit
* They both take an identical amount of code to configure
* They both are based on the Stream Controller concept

**Consuming state**  
* Rebuilding widget on state change
  * Bloc
    * Uses the `BlocBuilder` to wrap the `builder`
  * Riverpod
    * Uses the `Consumer` to wrap the builder
    * Consumer seems to have an extra line of code over bloc
    * Supports extending the `ConsumerWidget` and `ConsumerStatefulWidget` allowing for custom user 
    widgets without wrapping the build method
* Rebuilding widget on partial state change
  * Bloc
    * Uses the `BlocSelector` with a different pattern
  * Riverpod
    * Continues to use the `Consumer` with little change
* Reacting to state without rebuilding
  * Bloc
    * Uses the `BlocListener` widget to listen for state changes and perform an action without 
    rebuilding e.g. showing a snackbar
  * Riverpod
    * Provides similar functionality with `ref.listen<TYPE?>`
    * The ref pattern provides more flexibility than Bloc's pattern

**UI State Change**
* Bloc
  * Uses events to signal state objects to apply business logic and make changes to the state
  * Developer needs to create a number of event objets to define this behavior
* Riverpod
  * Access the state object from the UI using the `ref.read`
  * No need to create all the event files

## Riverpod
Riverpod is built on top of the `provider` package and came out in Aug 2020. Its key pattern is the 
idea that state should be managed in a centralized location, rather than being scattered throughout 
the application. Riverpod also allows for managing dependencies between different parts of the state 
which allows Riverpod to manage complex state in efficient scalable ways. Additionally Riverpod 
provides a set of tools for debuggiung and testing.

The primary wins with Riverpod are that it:
* state management is achieved through `Provider - a Flutter integrated state managment solution`.
* less overhead with event definitions
* has more intuitive naming
* AsyncValue is freakin awesome

**References**
* [Riverpod #1 Introduction](https://codewithandrea.com/articles/flutter-app-architecture-riverpod-introduction/)
* [Riverpod #2 Data mutation](https://codewithandrea.com/articles/data-mutations-riverpod/)
* [Riverpod - Code with Andrea](https://codewithandrea.com/tags/riverpod/)
* [Riverpod - codewithandrea](https://codewithandrea.com/articles/flutter-state-management-riverpod/)
* [FutureProvider fixes FutureBuilder](https://pub.dev/documentation/riverpod/latest/riverpod/FutureProvider-class.html)
* [Riverpod 2 changes](https://medium.com/@ahmedtahaelelemy/riverpod-2-a-comprehensive-overview-and-comparison-with-legacy-providers-e413e61fe53b)
* [Riverpod examples](https://dev.to/nikki_eke/master-riverpod-even-if-you-are-a-flutter-newbie-2m34)
* [AsyncValue is Awesome](https://codewithandrea.com/articles/flutter-use-async-value-not-future-stream-builder/)
* [Riverpod Caching and Providers](https://codewithandrea.com/articles/flutter-riverpod-data-caching-providers-lifecycle/)

**General notes**
* All providers are lazy loaded during the `ref.watch` call.
* Use `ref.watch()` to get values and rebuild when changes occur
* Use `ref.read()` for one time reads/writes where you don't want rebuilds
* Use `ref.listen()` to get a callback that is executed when the provider changes

### Todo app example
1. Create a new project
   * `flutter create --platform=linux,android riverpod_todo`
2. Add the river pod package
   * `flutter pub add flutter_riverpod`
3. Add the river pod import and wrap the app with a `ProvideScope`
   ```dart
   import 'package:flutter_riverpod/flutter_riverpod.dart';
   
   void main() {
     runApp(const ProviderScope(
       child: MyApp(),
     ));
   }
   ```
4. Define the global package providers e.g.
   ```dart
   final itemStreamProvider = StreamProvider<Item>((ref) {
     // return a Stream<Item>
   });
   
   final itemFutureProvider = FutureProvider<Item>((ref) {
     // return a Future<Item>
   });
   ```
5. Create a consumer widget and watch the providers
   ```dart
   class SomeWidget extends ConsumerWidget {
     @override
     Widget build(BuildContext context, WidgetRef ref) {

       // Watch provider for changes
       final streamAsyncValue = ref.watch(itemStreamProvider);

       return streamAsyncValue.when(...);
     }
   }
   ```

### Consumer
The `Consumer` class wraps portions of the application as desired to provide a performant way to 
get access to data and trigger rebuilds smaller more granular portions of your application.

**Example** below shows how the `MyHomePage` is not being rebuilt in its entirety but rather only the 
small piece wrapped by the `Consumer` class.
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final state = StateProvider((ref) => 7);
class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text('Riverpod example'),
      ),
      body: Center(
        child: Consumer(builder: (context, ref, child) {
          final count = ref.watch(state).toString();
          return Text(count, style: Theme.of(context).textTheme.headlineLarge);
        }),
      ),
    );
  }
}
```

### ConsumerWidget
`ConsumerWidget` is a good replacement for `StatelessWidget` with convenient access to state from 
providers.

Allows for custom user widgets with built in provider support for injecting data into that widget. 
This is less performant as it rebuilds the entire widget. Using the `Consumer` pattern around just 
the piece that you want rebuilt is more performant although less maybe less aesthetically pleasing. 
However best practice in the Flutter world is to create granular widgets which would mean that 
`ConsumerWidget` would be a perfect fit and just as performant.

**Example** below will rebuild the full `MyHomePage` widget as it is wrapped with the `ConsumerWidget`.
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final state = StateProvider((_) => 7);
class MyHomePage extends ConsumerWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context, ref) {
    final count = ref.watch(state).toString();

    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text('Riverpod example'),
      ),
      body: Center(child: Text(count, style: Theme.of(context).textTheme.headlineLarge)),
    );
  }
}
```

### ConsumerStatefulWidget
`ConsumerStatefulWidget` is a good replacement for `StatefullWidget` with convenient access to state from 
providers.


### Providers
**Providers**
* not widgets, but rather plain Dart objects
* defined outside of the widget tree as global final variables
* buisness logic is placed inside your providers which are super-powered functions
  * they behave like normal functions but have caching
  * offer default error/loading handling
  * are listenable
  * automatically re-execute when data changes
* stores all state allowing easy access to data from multiple locations
* replaces design patterns such as singletons, service locators, dependency injection and inherited widgets
* allows you to optimize your wiget rebuilding through granular triggers
* provides a caching layer for async data
* makes your code more testable

**Summary when to use**
1. `Provider` - immutable state you never change but just want access to
2. `StateProvider` - legacy
3. `StateNotifierProvider` - legacy
4. `FutureProvider` - perform and cache async API calls; used with `AsyncValue`
5. `StreamProvider` - like future but for a stream
6. `ChangeNotifierProvider` - legacy
7. `NotifierProvider` - 
8. `AsyncNotifierProvider` - like future provider but for updates as well

### Provider
This is great for accessing immutable objects

### FutureProvider
`FutureProvider` can be considered a combination of `FutureBuilder` and `Provider` and solves the 
limitations of `FutureBuilder`.

**FutureBuilder - flutter core**
* is more cumbersome to work i.e. connectionState, data and error cases
* doesn't have the same ability to provide data down the widget tree or cache the data as effectively.

**AsyncValue - Riverpod**
* uses factory constructors to define three mutually exclusive states: data, loading and error
* more concise syntax
* proper async result caching

**Example: reading a configuration file**  
`FutureProvider` can be a great way to handle reading in a json configuration file for your 
application. 

Creating the configuration would be done with your typical async/await syntax, but inside the 
provider.
```dart
final configProvider = FutureProvider<Configuration>((ref) async {
  final content = json.decode(
    await rootBundle.loadString('assets/configurations.json'),
  ) as Map<String, Object?>;

  return Configuration.fromJson(content);
});
```

Then the UI can listen for configuration like this:
```dart
// This will automatically rebuild this widget when the Future completes
Widget build(BuildContext context, WidgetRef ref) {
  AsyncValue<Configuration> config = ref.watch(configProvider);

  return config.when(
    loading: () => const CircularProgressIndicator(),
    error: (err, stack) => Text('Error: $err'),
    data: (config) {
      return Text(config.host);
    },
  );
}
```

**Another example**
```dart
// Load the todos from persistent storage
final todosProvider = FutureProvider<List<Todo>>((ref) async {
  return Future.delayed(const Duration(seconds: 2), () {
    return const [
      Todo(
        id: '1',
        title: 'Example Todo 1',
        description: 'Description 1',
      ),
      Todo(
        id: '2',
        title: 'Example Todo 2',
        description: 'Description 2',
      ),
      Todo(
        id: '3',
        title: 'Example Todo 3',
        description: 'Description 3',
      ),
    ];
  });
});
```

### StreamProvider

**Example**
```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

final itemStreamProvider = StreamProvider<Item>((ref) {
  return getItem(); // returns a Stream<Item>
});

class ItemWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final itemValue = ref.watch(itemStreamProvider);

    return itemValue.when(
      data: (item) => ListTile(
        title: Text(item.title),
        subtitle: Text(item.description),
      ),
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (e, st) => Center(child: Text(e.toString())),
    );
  }
);
```

### ScopedProvider

### NotifierProvider
This is the replacement for the now deprecated `StateProvider`. The new provider:
* allows for encapsulation of logic for updating the state in a more advanced way
* only rebuilds the widgets that are listening to the specific state change i.e. more performant



### AsyncNotifierProvider
This is the replacment for the now deprecated `StateNotifierProvider`:
* when doing async work inside a notifier we can set the state more than once to cover all cases
* new version provides async initialization
* `build` method override is the initialization method

```dart
// 1. add the necessary imports
import 'dart:async';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 2. extend [AsyncNotifier]
class AuthController extends AsyncNotifier<void> {
  // 3. override the [build] method to return a [FutureOr]
  @override
  FutureOr<void> build() {
    // 4. return a value (or do nothing if the return type is void)
  }

  Future<void> signInAnonymously() async {
    // 5. read the repository using ref
    final authRepository = ref.read(authRepositoryProvider);
    // 6. set the loading state
    state = const AsyncLoading();
    // 7. sign in and update the state (data or error)
    state = await AsyncValue.guard(authRepository.signInAnonymously);
  }
}
```

### Riverpod annotation
The `@riverpod` annotation will convert the annotated function into a provider generating the 
equivalent with the `Provider` suffix.

## Bloc
* know what state your application is in at any point in time
* easily test every case to make sure your app is responding appropriately
* record every single user interaction in your application for data-driven decisions
* develop fast and reactive apps
* MIT license
* `bloc` - core bloc library
* `flutter_bloc` - package of flutter widgets designed to work with bloc
* `hydrated_bloc` - extension allowing for persisting bloc state
* `replay_bloc` - extension for adding undo and redo

**References**
* [bloc website](https://bloclibrary.dev/#/)
* [bloc dart package](https://pub.dev/packages/bloc)
* [bloc - Flutteris](https://www.flutteris.com/blog/en/reactive-programming---streams---bloc)
* [Flutter Bloc - Flutterly youtube](https://www.youtube.com/watch?v=THCkkQ-V1-8)

### VSCode Extension
Bloc has its own VSCode extension called `bloc` from `Felix Angelov`

### Core Concepts
Bloc is composed of three pieces: the UI, Bloc and the Data. The UI or other external soruces that 
trigger events which the Block or business logic receives and builds a new state that the UI can 
consume. You can think of Bloc as the bridge between your data sources and your widgets. Bloc may 
even need to go fetch data from an API to satify the UI request.

In order to get started with this package you want to also use the `equateable` package to reduce 
boiler plate code for comparing objects. Bloc compares and only emits a new state if the prior state 
changed.

1. Use VSCode to add `Bloc: New Bloc` and give it a name e.g. `todo` which will create 3 new bloc files
   1. `bloc/todo_bloc.dart`
   2. `bloc/todo_event.dart`
   3. `bloc/todo_state.dart`

2. Define your `state` part of the bloc

3. Define your `event` part of the bloc

4. Define your `bloc` part of the bloc

5. Wrap your `MaterialApp` in a `MultiBlocProvider`

6. Wrap the home page's `body` with a `BlocBuilder` to then determine which parts of your UI to rebuild

7. Wrap todo page `body` with a `BlocListener`

<!-- 
vim: ts=2:sw=2:sts=2
-->
