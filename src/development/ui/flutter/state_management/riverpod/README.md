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
  * [Getting Started](#getting-started)
* [Use Cases](#use-cases)
  * [Request data without side effects](#request-data-without-side-effects)
  * [Request data with side effects](#request-data-with-side-effects)
  * [Handle Stream or Future directly](#handle-stream-or-future-directly)
  * [Combining providers](#combining-providers)
  * [Consume statelessly](#consume-statelessly)
  * [Consume statefully](#consume-statefully)
  * [Consume with arguments](#consume-with-arguments)
* [Repository design pattern](#repository-design-pattern)
  * [Repository](#repository)
  * [Wrap mutable repo functions with NotifierProvider](#wrap-mutable-repo-functions-with-notifierprovider)
  * [Combine providers](#combine-providers)
* [Code Generation](#code-generation)
  * [Riverpod annotation](#riverpod-annotation)
  * [Riverpod auto dispose](#riverpod-auto-dispose)
  * [Functional form code generation](#functional-form-code-generation)
  * [Class Based form code generation](#class-based-form-code-generation)

### References
* [Combining requests](https://riverpod.dev/docs/essentials/combining_requests)
* [Listening to a Stream](https://riverpod.dev/docs/essentials/websockets_sync#listening-to-a-stream)
* [onDispose and addListener](https://riverpod.dev/docs/essentials/websockets_sync#listenable-objects-considerations)
* [Riverpod examples](https://dev.to/nikki_eke/master-riverpod-even-if-you-are-a-flutter-newbie-2m34)
* [Code With Andrea examples](https://github.com/bizz84/flutter_example_apps)

## Overview

### General points
* Providers
  * provide global access and caching
    * subsequent reads simply returns the cached response
  * are plain Dart objects defined outside the widget tree
  * replace other design patterns like
    * Singletons, Service locators, dependency injection and inherited widgets
    * Flutter's FutureBuilder
  * allow you to optimize your wiget rebuilding through granular triggers
  * are lazy loaded during the `ref.watch` call.
  * should contain your business logic
  * provide access to all provides via the `ref` param
  * ability to override UI droping with `@Riverpod(keepAlive: true)`
  * handle errors making them available to the UI in the `errors`
* Use `ref.watch()` to get values and rebuild when changes occur
* Use `ref.read()` for one time reads/writes where you don't want rebuilds
* Use `ref.listen()` to get a callback that is executed when the provider changes

### Getting Started
1. Create a new project
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
     runApp(const ProviderScope(child: MyApp()));
   }
   ```

5. Convert your top level `StatelessWidget` into a `ConsumerWidget`
   ```dart
   class MyApp extends ConsumerWidget {
     const MyApp({super.key});

     @override
     Widget build(BuildContext context, WidgetRef ref) {
       return MaterialApp(
         title: Const.appName,
         home: Layout(),
       );
     }
   }
   ```

## Use Cases
I've found Riverpod's documentation to be confusing due to the multiple different versions and styles 
of the code and spotty documentation for each. The author's original versions had users writing and 
chosing specific provider types for different use cases, but then later converted to a code 
generation style that doesn't require a user to know the provider types. I've found it is much 
more intuitive to just use the code generation to define a provider that handles your use case and 
not worry about what the specific provider is called internally.

### Request data without side effects
Use the functional provider form for static data or asynchronous operations that don't need to modify 
the state or handle other side effects and instead only need to cache the state. The provider will 
not be executed until the first UI pass calling a `ref.watch` or other ref function. Subsequent reads 
will use the cached response. This gives you `global access` and `caching` to avoid costly IO bound 
requests.

**References**
* [Riverpod cache network response](https://riverpod.dev/docs/essentials/first_request)

**Example use cases**
* GET an async non-pagignated network resource 
  * e.g. the current weather for a given city

***Perform async request and cache response***  
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

***Perform sync request and cache response***  
Key difference here is you just leave off the `async` notation and return a non future.
```dart
@riverpod
int synchronousExample(SynchronousExampleRef ref) {
  return 0;
}
```

***Handle provider arguments***  
Caching now happens per argument used like a key for the cached result. Requires that the parameters 
have a consistent equality implementation.
```dart
// We can add arguments directly to the function
@riverpod
Future<Activity> activity(ActivityRef ref, String activityType) async {
  return fetchActivity();
}
```

### Request data with side effects
Use the class based provider form, a.k.a `Notifier`, for requesting data that needs additional 
complexity to mutate the state after the fact or perform other related operations via custom methods.

**References**
* [Riverpod side-effects - docs](https://riverpod.dev/docs/essentials/side_effects)

**Example use cases**
* GET an async pagignated network resource
  * e.g. requires mutating the internal state to add an additional page

* public methods can be consumed with `ref.read(yourProvider.notifier).yourMethod()`
  * use `ref.read` to perform mutation and `ref.watch` to respond to response
* `build` method is responsible for the inital state of your provider
  * class based providers should not have a constructor
* ensure your local cache matches the API response by updating locally
  * the UI will have the most up-to-date state possible in this way
* `ref.invalidateSelf();` will cause the `build` to be called again and notify listeners

***Update local cache with API response***
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

    // We can then manually update the local cache. For this, we'll need to obtain the previous state.
    // Caution: The previous state may still be loading or in error state.
    // To gracefully handle this use `this.future` instead `this.state`, which would enable awaiting 
    // the loading state, and throw an error if the state is in error state.
    final previousState = await future;

    // We can then update the state, by creating a new state object.
    // This will notify all listeners.
    state = AsyncData([...previousState, todo]);
  }
}
```

***Handle provider arguments***  
Caching now happens per argument used like a key for the cached result. Requires that the parameters 
have a consistent equality implementation.
```dart
@riverpod
class ActivityNotifier2 extends _$ActivityNotifier2 {
  /// Notifier arguments are specified on the build method.
  @override
  Future<Activity> build(String activityType) async {
    // Arguments are also available with "this.<argumentName>"
    print(this.activityType);

    // TODO: perform a network request to fetch an activity
    return fetchActivity();
  }
}
```

### Handle Stream or Future directly
AsyncValue is nice if you want to replace your entire page with a loading indicator or an error 
message. However if you'd like to handle the errors inline and allow for displaying the existing 
cached data instead one way is to simply bypass the AsyncValue and handle the Future or Stream 
directly with the Flutter FutureBuilder. Alternately you can bypass AsyncValue by watching the 
provider's future directly which is arguably more useful as you can then use either behavior as 
needed.

***Bypass the AsyncValue and listen directly to the future***
```dart
@riverpod
Future<List<String>> restaurantsNearMe(RestaurantsNearMeRef ref) async {
  final location = await ref.watch(locationProvider.future);
  ...
}
```

***Have the provider return the raw value***
```dart
@riverpod
Raw<Stream<int>> rawStream(RawStreamRef ref) {
  // "Raw" is a typedef. No need to wrap the return
  // value in a "Raw" constructor.
  return const Stream<int>.empty();
}

class Consumer extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    Stream<int> stream = ref.watch(rawStreamProvider);
    return StreamBuilder<int>(
      stream: stream,
      builder: (context, snapshot) {
        return Text('${snapshot.data}');
      },
    );
  }
}
```

### Combining providers
By using a `ref.watch` inside our provider's `build` method we can cause it to be invalidated and 
rebuild in the next frame or read. `ref.watch` is the preferred way to do this. When doing so you'll 
want to await the actual future of the provider rather than deal with the AsyncValue.

When the listened provider changes and our request recomputes, the previous state is kept until the 
new request completes. It is possible to emit state updates during the re-computation to signal the 
UI to show incremental updates.

* Note the `ref.watch(otherProvider.future)` which uses's the provider's future directly
* Its bad practice to call `ref.watch` inside non `build` method

***Functional variant***
```dart
part 'example.g.dart'

@riverpod
Future<Activity> example(ExampleRef ref) async {
  final otherValue = await ref.watch(otherProvider.future);
  return 0;
}
```

***Class variant***
```dart
part 'example.g.dart'

@riverpod
class Example extends _$Example {
  @override
  Future<Activity> build() async {
    final otherValue = await ref.watch(otherProvider.future);

    return 0;
  }
}
```

### Listen to provider changes manually
Sometimes `ref.watch` doesn't offer enough flexibility. That is where `ref.listen` might be of help.

* It is safe to use `ref.listen` in the build phase

***Listen to a provider***
```dart
@riverpod
int example(ExampleRef ref) {
  ref.listen(otherProvider, (previous, next) {
    print('Changed from: $previous, next: $next');
  });
  return 0;
}
```

### Consume directly 
AsyncValue is nice if you want to replace your entire page with a loading indicator or an error 
message. However if you'd like to handle the errors inline and allow for displaying the existing 
cached data instead one way is to simply bypass the AsyncValue and handle the Future or Stream 
directly. 

```dart
class Home extends ConsumerWidget {
  const Home({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final AsyncValue<Activity> activity = ref.watch(activityProvider);
    return Center(
      child: if 
        AsyncData(:final value) => Text('Activity: ${value.activity}'),
        AsyncError() => const Text('Oops, something unexpected happened'),
        _ => const CircularProgressIndicator(),
      },
    );
  }
}
```

### Consume statelessly
You can consume an async response in the UI by watching, i.e. `ref.watch`, a provider then handling 
the `AsyncValue` response which has 3 different states.

**ConsumerWidget example**  
`ConsumerWidget` allows for a cleaner simpler way to build in provider access to your widgets. If 
implemented incorrectly though it can be less performant as it rebuilds the entire widget. You could 
use the `Consumer` pattern around just the piece that you want rebuilt as is more performant to 
target just a section of your UI. However best practice in the Flutter world is to create granular 
widgets which would mean that `ConsumerWidget` would be a perfect fit and just as performant.

* Read the activityProvider. This will start the network request if it wasn't already started.
* By using `ref.watch`, this widget will rebuild whenever the the activityProvider updates.
* This can happen when:
  * The response goes from "loading" to "data/error"
  * The request was refreshed
  * The result was modified locally (such as when performing side-effects)
* We could alternatively use `if (activity.isLoading) { ... } else if (...)`

***Consume async response AsyncValue with switch***
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

***Consume sync response directly***  
One side effect of a synchronouse watch is that if the function throws and exception it won't get 
caught and handled as a AsyncValue and will be re-thrown in stead.
```dart
Consumer(
  builder: (context, ref, child) {
    // The value is not wrapped in an "AsyncValue"
    int value = ref.watch(synchronousExampleProvider);

    return Text('$value');
  },
);
```

### Consume statefully
Use the `ConsumerStatefulWidget` to provide standard Flutter StatefulWidget capabilities married with 
a Consumer for provider access. `ref` is not passed as a parameter in this form but is a property of 
`ConsumerState`.

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

### Consume with arguments
Provider becomes a function that you pass the arguments in to 

```dart
return Consumer(
  builder: (context, ref, child) {
    final recreational = ref.watch(activityProvider('recreational'));
    final cooking = ref.watch(activityProvider('cooking'));

    // We can then render both activities.
    // Both requests will happen in parallel and correctly be cached.
    return Column(
      children: [
        Text(recreational.valueOrNull?.activity ?? ''),
        Text(cooking.valueOrNull?.activity ?? ''),
      ],
    );
  },
);
```

## Repository Design Pattern
Frequently applications need external resources on local devices or over the network. To interact 
with these external resources your application needs to leverage the external resource's API. 
Whether a client library is provided or your have to write your own there is a layer of code 
dedicated to the task. This layer of code from the client to the external resource is referred to as 
the `Repository layer`. Typically the repository delegates the management of these expensive async IO 
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

<!-- 
vim: ts=2:sw=2:sts=2
-->
