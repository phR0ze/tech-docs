# Riverpod
Riverpod is the 2.0 version of the venerable Provider package written by the same developer. It 
solves the shortcoming of the 1.0 project providing the follwing aggregate set of benefits:

* ***Caching for business logic***
* ***Complex state management with error/loading handling and listeners***
* ***Triggers UI rebuilds when state changes***
* ***Dependency injection***

### Quick links
* [Overview](#overview)
  * [General points](#general-points)
* [Repository design pattern](#repository-design-pattern)
  * [Repository](#repository)
  * [Wrap immutable repo functions with FutureProvider](#wrap-immutable-repo-functions-with-futureprovider)
  * [Wrap mutable repo functions with NotifierProvider](#wrap-mutable-repo-functions-with-notifierprovider)
  * [Combine providers](#combine-providers)
* [Consuming a provider](#consuming-a-provider)
  * [ConsumerWidget](#consumerwidget)
  * [ConsumerStatefulWidget](#consumerstatefulwidget)
* [Code Generation](#code-generation)
  * [Riverpod annotation](#riverpod-annotation)
  * [Riverpod auto dispose](#riverpod-auto-dispose)
  * [Functional code generation](#functional-code-generation)
  * [Class Based code generation](#class-based-code-generation)

### References
* [Riverpod #1 Introduction](https://codewithandrea.com/articles/flutter-app-architecture-riverpod-introduction/)
* [Riverpod #3 Data mutation](https://codewithandrea.com/articles/data-mutations-riverpod/)
* [Riverpod - Code with Andrea](https://codewithandrea.com/tags/riverpod/)
* [Riverpod - codewithandrea](https://codewithandrea.com/articles/flutter-state-management-riverpod/)
* [FutureProvider fixes FutureBuilder](https://pub.dev/documentation/riverpod/latest/riverpod/FutureProvider-class.html)
* [Riverpod 2 changes](https://medium.com/@ahmedtahaelelemy/riverpod-2-a-comprehensive-overview-and-comparison-with-legacy-providers-e413e61fe53b)
* [Riverpod examples](https://dev.to/nikki_eke/master-riverpod-even-if-you-are-a-flutter-newbie-2m34)
* [AsyncValue is Awesome](https://codewithandrea.com/articles/flutter-use-async-value-not-future-stream-builder/)
* [Riverpod Caching and Providers](https://codewithandrea.com/articles/flutter-riverpod-data-caching-providers-lifecycle/)
* [Code With Andrea examples](https://github.com/bizz84/flutter_example_apps)

## Overview

### General points
* Providers provide global access and caching
* Providers are not widgets, but rather plain Dart objects defined outside the widget tree as globals
* Providers allow for global access from any widget
* Providers replace other design patterns such as:
  * Singletons
  * Service locators
  * Dependency injection
  * Inherited widgets
* Providers allow you to optimize your wiget rebuilding through granular triggers
* Buisness logic should be placed inside your providers
* Provider provide a caching layer for async data
* All providers are lazy loaded during the `ref.watch` call.
* Use `ref.watch()` to get values and rebuild when changes occur
* Use `ref.read()` for one time reads/writes where you don't want rebuilds
* Use `ref.listen()` to get a callback that is executed when the provider changes

### When to use which provider
* `Provider` - immutable state you never change but just want access to
* `FutureProvider` - immutable perform and cache async API calls; used with `AsyncValue`
* `StreamProvider` - like future but for a stream
* `NotifierProvider` - mutable form
* `AsyncNotifierProvider` - mutable form

**Deprecated**
* `StateProvider`
* `StateNotifierProvider`
* `ChangeNotifierProvider`

## Repository Design Pattern
Frequently applications need external resources on local devices or over the network. To interact 
with these external resources your application needs to leverage the external resource's API. 
Whether a client library is provided or your have to write your own there is a layer of code 
dedicated to task. This layer of code from the client to the external resource is referred to as the 
`Repository layer`. Typically the repository delegates the management of these expensive async IO 
calls to the application. Riverpod provides solutions to the additional responsibilities allowing you 
to skip re-implementing common functionality around handling these external resources.

**References**
* [Repository Pattern - codewithadrea](https://codewithandrea.com/articles/flutter-repository-pattern/)

For example:
* The UI should render a loading state while the request is being made
* Errors should be gracefully handled
* The request should be cached

**In the respository pattern there are three areas of concern:**
```
   Presentation        Business Logic             Data
  .------------.     .----------------.      .------------.
 /              \   /                  \    /              \

|''''''''''| ref.watch   /````````\   ref.read    .......
|          | ---------> /          \ ---------->/        \ 
|    UI    |           . Controller .          |   Repo   |
|          | rebuild    \          /  models    \        /
|__________| <---------- \________/ <----------- '------'
```

### Repository
The repository portion of our codebase encapsulates the data specific code away from the rest of the 
application and allows for switching out the repository implementation for a mock or test implementation.

***We want to wrap the repository's API with Riverpod providers for easy access and caching:***
* Wrap immutable repo functions with the `FutureProvider/StreamProvider`
* Wrap mutable repo functions with `NotifierProvider/AsyncNotifierProvider`

By wrapping the Repo API functions in a granular fashion we maximize caching and provide the ability 
to react to changes in the UI in a very targeted way with the minimal amount of costly Repo IO calls.

**References**
* [River pod basics - docs](https://riverpod.dev/docs/essentials/first_request)

### Wrap immutable repo functions with FutureProvider
By wrapping a repository function in a future provider we get `global access` to the data and 
`caching` to avoid costly IO bound duplicated calls.

* the function will not be executed until the UI reads the provider at least once
  * subsequent reads simply returns the cached response
* `ref` param passed in provides access to all other registered providers
* the name of the function determines the type of provider used
* functional form is the same as the class based form without custom methods for side effects
* if the UI navigate away from the page that is using the provider the cache will be dropped
* Riverpod handles any errors making them available to the UI in the `errors`

**Functional form for async FutureProvider**  
For an async external operation use a return type of `Future<Activity>` where `Activity` is your 
model object type and add the `async` specifier to the function which will then use the 
`FutureProvider` type under the hood. The following code creats the provider `activityProvider`.
```dart
@riverpod
Future<Activity> activity(ActivityRef ref) async {
  // Using package:http, we fetch a random activity from the Bored API.
  final response = await http.get(Uri.https('boredapi.com', '/api/activity'));

  // Using dart:convert, we then decode the JSON payload into a Map data structure.
  final json = jsonDecode(response.body) as Map<String, dynamic>;

  // Finally, we convert the Map into an Activity instance.
  return Activity.fromJson(json);
}
```

### Wrap mutable repo functions with NotifierProvider
Notifiers are the mutable providers. Use them to expose the Repo's mutable functions allowing for 
updating the local cache and the remote resource to keep them both in-sync.

* public methods can be consumed with `ref.read(yourProvider.notifier).yourMethod()`
  * use `ref.read` to perform mutation
* notifiers should not have public properties beyond the built in state
* `build` method is responsible for the inital state of your provider
  * do not put logic in the constructor of your notifier it should be in the `build` method
* ensure your local cache matches the API response by updating locally
  * the UI will have the most up-to-date state possible in this way

**References**
* [Riverpod side-effects - docs](https://riverpod.dev/docs/essentials/side_effects)

***Update local cache with API response***
```dart
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async => [/* ... */];

  Future<void> addTodo(Todo todo) async {
    // The POST request will return a List<Todo> matching the new application state
    final response = await http.post(
      Uri.https('your_api.com', '/todos'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(todo.toJson()),
    );

    // We decode the API response and convert it to a List<Todo>
    List<Todo> newTodos = (jsonDecode(response.body) as List)
        .cast<Map<String, Object?>>()
        .map(Todo.fromJson)
        .toList();

    // We update the local cache to match the new state.
    // This will notify all listeners.
    state = AsyncData(newTodos);
  }
}
```

***Force rebuild of local cache***
```dart
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async => [/* ... */];

  Future<void> addTodo(Todo todo) async {
    // We don't care about the API response
    await http.post(
      Uri.https('your_api.com', '/todos'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(todo.toJson()),
    );

    // Once the post request is done, we can mark the local cache as dirty.
    // This will cause "build" on our notifier to asynchronously be called again,
    // and will notify listeners when doing so.
    ref.invalidateSelf();

    // (Optional) We can then wait for the new state to be computed.
    // This ensures "addTodo" does not complete until the new state is available.
    await future;
  }
}
```

***Manual update of local cache***
```dart
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async => [/* ... */];

  Future<void> addTodo(Todo todo) async {
    // We don't care about the API response
    await http.post(
      Uri.https('your_api.com', '/todos'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(todo.toJson()),
    );

    // We can then manually update the local cache. For this, we'll need to
    // obtain the previous state.
    // Caution: The previous state may still be loading or in error state.
    // A graceful way of handling this would be to read `this.future` instead
    // of `this.state`, which would enable awaiting the loading state, and
    // throw an error if the state is in error state.
    final previousState = await future;

    // We can then update the state, by creating a new state object.
    // This will notify all listeners.
    state = AsyncData([...previousState, todo]);
  }
}
```

### Combine providers
By using a `ref.watch` inside our provider's `build` method we can cause it to be invalidated and 
rebuilt in the next frame or read.

**References**
* [Combining requests - docs](https://riverpod.dev/docs/essentials/combining_requests)

## Consuming a provider
To consume a provider in the UI requires handling the provider's different states. Riverpod's 
`AsyncValue` handles futures nicely providing loading, error and data states that the UI can then 
react to.

* Widgets can listen to as many providers as they want, just add a `ref.watch`
* `ConsumerWidget` allows for a cleaner simpler way to build in provider access to your widgets. If 
  implemented incorrectly though it can be less performant as it rebuilds the entire widget. You could
  use the `Consumer` pattern around just the piece that you want rebuilt as is more performant 
  to target just a section of your UI. However best practice in the Flutter world is to create 
  granular widgets which would mean that `ConsumerWidget` would be a perfect fit and just as 
  performant.

### ConsumerWidget
* Read the activityProvider. This will start the network request if it wasn't already started.
* By using `ref.watch`, this widget will rebuild whenever the the activityProvider updates.
* This can happen when:
  * The response goes from "loading" to "data/error"
  * The request was refreshed
  * The result was modified locally (such as when performing side-effects)
* We could alternatively use `if (activity.isLoading) { ... } else if (...)`

```dart
class Home extends ConsumerWidget {
  const Home({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final AsyncValue<Activity> activity = ref.watch(activityProvider);
    return Center(
      child: switch (activity) {
        AsyncData(:final value) => Text('Activity: ${value.activity}'),
        AsyncError() => const Text('Oops, something unexpected happened'),
        _ => const CircularProgressIndicator(),
      },
    );
  }
}
```

### ConsumerStatefulWidget
* `ref` is not passed as a parameter in this form but is a property of `ConsumerState`

```dart
class Home extends ConsumerStatefulWidget {
  const Home({super.key});

  @override
  ConsumerState<Home> createState() => _HomeState();
}

class _HomeState extends ConsumerState<Home> {
  @override
  void initState() {
    super.initState();

    ref.listenManual(activityProvider, (previous, next) {
      // TODO show a snackbar/dialog
    });
  }

  @override
  Widget build(BuildContext context) {
    final AsyncValue<Activity> activity = ref.watch(activityProvider);
    return Center(/* ... */);
  }
}
```

***Watch Future for state change***
```dart
class Example extends ConsumerStatefulWidget {
  const Example({super.key});

  @override
  ConsumerState<Example> createState() => _ExampleState();
}

class _ExampleState extends ConsumerState<Example> {
  // The pending addTodo operation. Or null if none is pending.
  Future<void>? _pendingAddTodo;

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      // We listen to the pending operation, to update the UI accordingly.
      future: _pendingAddTodo,
      builder: (context, snapshot) {
        // Compute whether there is an error state or not.
        // The connectionState check is here to handle when the operation is retried.
        final isErrored = snapshot.hasError &&
            snapshot.connectionState != ConnectionState.waiting;

        return Row(
          children: [
            ElevatedButton(
              style: ButtonStyle(
                // If there is an error, we show the button in red
                backgroundColor: MaterialStateProperty.all(
                  isErrored ? Colors.red : null,
                ),
              ),
              onPressed: () {
                // We keep the future returned by addTodo in a variable
                final future = ref
                    .read(todoListProvider.notifier)
                    .addTodo(Todo(description: 'This is a new todo'));

                // We store that future in the local state
                setState(() {
                  _pendingAddTodo = future;
                });
              },
              child: const Text('Add todo'),
            ),
            // The operation is pending, let's show a progress indicator
            if (snapshot.connectionState == ConnectionState.waiting) ...[
              const SizedBox(width: 8),
              const CircularProgressIndicator(),
            ]
          ],
        );
      },
    );
  }
}
```

## Code Generation
Riverpod syntax is a bit unwieldy when writing it by hand. To combat this code generation was 
introduced. There are two styles of code generation that may allow for reducing the overhead even 
further if you don't need to have public methods to mutate your state.

* Use the functional form for immutable state
* Use the class based form for mutable state

**References**
* [Code generation from Riverpod docs](https://riverpod.dev/docs/concepts/about_code_generation)

### Riverpod annotation
The `@riverpod` annotation will convert the annotated function or class into a provider generating 
the equivalent with the `Provider` suffix.

Use the `@Riverpod` version if you need to specify arguments to the code generation:
* `@Riverpod(keepAlive: true)` 

### Riverpod auto dispose
When using code generation, by default the provider state is destroyed when the provider stops being 
listened to. This happens when a listener has no active listener for a full frame. When this happens,
the state is destroyed.

This behavior can be opted out of by using the `keepAlive: true` annotation. However enabling or 
disabling this feature will not impact whether or not the state is destroyed when the provider is 
recomputed. The state will always be destroyed when the provider is recomputed.

### Functional code generation
Use the functional form for immutable state

***Sync example***
```dart
@riverpod
String example(ExampleRef ref) {
  return 'foo';
}
```

***Async example***
```dart
@riverpod
Future<String> example(ExampleRef ref) async {
  return Future.value('foo');
}
```

***Async stream example***
```dart
@riverpod
Stream<String> example(ExampleRef ref) async* {
  yield 'foo';
}
```

### Class Based code generation
Use the class based form for mutable state

***Sync example***
```dart
@riverpod
class Example extends _$Example {
  @override
  String build() {
    return 'foo';
  }

  // Add methods to mutate the state
}
```

***Async future example***
```dart
@riverpod
class Example extends _$Example {
  @override
  Future<String> build() async {
    return Future.value('foo');
  }

  // Add methods to mutate the state
}
```

***Async stream example***
```dart
@riverpod
class Example extends _$Example {
  @override
  Stream<String> build() async* {
    yield 'foo';
  }

  // Add methods to mutate the state
}
```

### Step by step
1. Setup a new project
   1. Create the new project
      ```bash
      $ flutter create --platform=linux,android riverpod_todo
      ```

   2. Add the riverpod packages
      ```bash
      $ flutter pub add flutter_riverpod
      $ flutter pub add riverpod_annotation
      $ flutter pub add dev:riverpod_generator
      $ flutter pub add dev:build_runner
      $ flutter pub add dev:custom_lint
      $ flutter pub add dev:riverpod_lint
      ```

   3. Add supporting packages
      ```bash
      $ flutter pub add json_annotation
      $ flutter pub add freezed_annotation
      $ flutter pub add dev:freezed
      $ flutter pub add dev:json_serializable
      ```

4. Add the river pod import and wrap the app with a `ProvideScope`
   ```dart
   import 'package:flutter_riverpod/flutter_riverpod.dart';
   
   void main() {
     runApp(const ProviderScope(
       child: MyApp(),
     ));
   }
   ```

5. Define your repo api interface as an abstract class
   ```dart
   abstract class MovieRepoApi {
     Future<List<TMDBMovie>> searchMovies({required int page, String query = ''});
     Future<List<TMDBMovie>> nowPlayingMovies({required int page});
     Future<TMDBMovie> movie({required int movieId});
   }
   ```

6. Implement your repo api interface
   ```dart
   class MovieRepo implements MovieRepoApi {
     MoviesRepo({required this.client, required this.apiKey});
     final Dio client;
     final String apiKey;

     Future<List<TMDBMovie>> searchMovies({required int page, String query = ''});
     Future<List<TMDBMovie>> nowPlayingMovies({required int page});
     Future<TMDBMovie> movie({required int movieId});
   }
   ```

6. Create a separate global function provider to make it the repo accessible to the full app
   ```dart
   part 'movies_repo.g.dart';
   
   @riverpod
   MoviesRepo moviesRepo(MoviesRepoRef ref) => MoviesRepo(
         client: ref.watch(dioProvider), // the provider we defined above
         apiKey: Env.tmdbApiKey, // a constant defined elsewhere
       );
   ```


## Todo app example
1. Create a new project
   * `flutter create --platform=linux,android riverpod_todo`
2. Add the river pod package
   ```bash
   $ flutter pub add flutter_riverpod
   $ flutter pub add riverpod_annotation
   $ flutter pub add dev:riverpod_generator
   $ flutter pub add dev:build_runner
   $ flutter pub add dev:custom_lint
   $ flutter pub add dev:riverpod_lint
   ```
3. Add the river pod import and wrap the app with a `ProvideScope`
   ```dart
   import 'package:flutter_riverpod/flutter_riverpod.dart';
   
   void main() {
     runApp(const ProviderScope(
       child: MyApp(),
     ));
   }
   ```
4. Add your model objects
   1. Add the supporting packages
      ```bash
      $ flutter pub add freezed_annotation
      $ flutter pub add json_annotation
      $ flutter pub add dev:build_runner
      $ flutter pub add dev:freezed
      $ flutter pub add dev:json_serializable
      ```
   2. Add `model/todo.dart`
      ```dart
      import 'package:freezed_annotation/freezed_annotation.dart';
      
      part 'todo.freezed.dart';
      part 'todo.g.dart';
      
      @freezed
      class Todo with _$Todo {
        const factory Todo({
          required String id,
          required String title,
          required String description,
          @Default(false) bool isCompleted,
          @Default(false) bool isCanceled,
        }) = _Todo;
      
        factory Todo.fromJson(Map<String, dynamic> json) => _$TodoFromJson(json);
      }
      ```
   3. Generate the model code with freezed
      ```bash
      $ dart run build_runner build --delete-conflicting-outputs
      ```
4. Create your riverpod provider e.g.
   1. Define your riverpod provider
      ```dart
      import 'package:riverpod_annotation/riverpod_annotation.dart';
      import '../model/todo.dart';
      
      // Generated riverpod code for todosProvider
      part 'todos.g.dart';
      
      // Defines todosProvider notifier to make it writable
      @riverpod
      class Todos extends _$Todos {
        // Define the initial value for the provider
        @override
        Future<List<Todo>> build() async {
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
        }
      }
      ```
   2. Now generate it
      ```bash
      $ dart run build_runner build --delete-conflicting-outputs
      ```
5. Create a consumer widget and watch the providers
   ```dart
   class SomeWidget extends ConsumerWidget {
     @override
     Widget build(BuildContext context, WidgetRef ref) {

       // Watch provider for changes
       var asyncValue = ref.watch(todosProvider);
       ...
       // Or add a new entry to the state without triggering rebuild
       ref.read(todosProvider.notifier).add(todo);
       ...
     }
   }
   ```

<!-- 
vim: ts=2:sw=2:sts=2
-->
