# State Management <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Applications frequently need to fetch data from disk, backend servers or the network asynchronously. 
Caching that async data locally to avoid re-executing the call again and handling all the complexity 
around this such as infinite lists, i.e. fetching while scrolling, debouncing async requests, offline 
mode, pull to refresh, search as we type etc... is what state management systems were created to solve.

Some of the most common in Flutter are:
* BLoC
* GetIt
* MobX
* Riverpod

## Overview
Flutter uses `StatefulWidgets` to generate `State` objects, which are then used to hold state. 
Widgets and State objects have different life cycles. Widgets are used for presentation and are 
frequently destroyed and recreated with changes while State is persisted between calls to the 
`build()` method.

**References**
* [State Management - Flutter docs](https://docs.flutter.dev/data-and-backend/state-mgmt/intro)
* [Provider vs BLoC](https://www.miquido.com/blog/flutter-architecture-provider-vs-bloc/)
* [Provider vs BLoC - Medium](https://medium.com/@dihsar/bloc-vs-provider-in-flutter-a-comprehensive-comparison-fbd0f6c41e50)

* ***ChangeNotifier*** creates state and dependencies can watch for notifications and rebuild

**Provider vs BLoC**  
Both are popular state management libraries. ***Provider*** provides the current data model to the 
place where we currently need it, it contains some data and notifies observers when a change occurs. 
In the Flutter SDK, this type is called a ***ChangeNotifier***. For this notifier to work we need the 
***ChangeNotifierProvider*** which provides observed objects for all its descendants. ***BLoC*** 
a.k.a. Business Logic Components is a Flutter architecture similar which tries more closely to mimic 
the common Android paradigm of MVP or MVVM. It provides separation of the presentation layer from the 
business logic rules.

Provider's pattern is to create a new class extending the ChangeNotifier and capture your data as 
class fields and allows for class methods to interact with the data in a custom way. BLoC's pattern 
is based on events. You define the events that your screens will produce then you define the BLoC 
objects that take the events. Because of BLoC's granular nature you can more granularly control the 
updates to your app wiget tree thus speeding up your whole app. BLoC offers more control but is more 
complex to use. Industry concensus is to use Provider unless you need more complexity and performance 
and then use BLoC which is more verbose and requires more boiler plate code.

## Package Comparison
I'm going to implement a simple todo in both to sort out my take on the subject.

**References**
* [Riverpod or Bloc](https://www.youtube.com/watch?v=EPVKdverFuw)
* [River pod or Bloc - site](https://mobileappcircular.com/bloc-vs-riverpod-making-the-right-choice-for-your-flutter-app-5feb4486ac4)
* [Bloc vs RiverPod](https://www.xavor.com/blog/bloc-vs-riverpod/)

### General
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
  * less boiler plate overhead with code generation
  * more flexible and intuitive with simplified syntax and naming
  * aligns more closely with flutter making it easier to update and maintain
  * a more modern take on state managment than bloc
  * allows for Consumer widget giving a more natural fit with Flutter
  * simpler more intuitive `ref.read(), ref.watch(), ref.listen()` API for changes

### Dependency Injection
* Bloc
  * uses the `BlocProvider` to wrap your whole app

* Riverpod
  * uses the `ProviderScope` to wrap your whole app

### State Notifications
* Bloc
  * uses the Cubit to handle state notifications

* Riverpod
  * uses the StateNotifier which is basically identical to the Cubit

* They both take an identical amount of code to configure
* They both are based on the Stream Controller concept

### Rebuilding widget on state change
* Bloc
  * Uses the `BlocBuilder` to wrap the `builder`
* Riverpod
  * Uses the `Consumer` to wrap the builder
  * Consumer seems to have an extra line of code over bloc
  * Supports extending the `ConsumerWidget` and `ConsumerStatefulWidget` allowing for custom user 
  widgets without wrapping the build method

### Rebuilding widget on partial state change
* Bloc
  * Uses the `BlocSelector` with a different pattern

* Riverpod
  * Continues to use the `Consumer` with little change

### Reacting to state without rebuilding
* Bloc
  * Uses the `BlocListener` widget to listen for state changes and perform an action without 
  rebuilding e.g. showing a snackbar

* Riverpod
  * Provides similar functionality with `ref.listen<TYPE?>`
  * The ref pattern provides more flexibility than Bloc's pattern

### UI State Change
* Bloc
  * Uses events to signal state objects to apply business logic and make changes to the state
  * Developer needs to create a number of event objets to define this behavior

* Riverpod
  * Access the state object from the UI using the `ref.read() ref.watch(), ref.listen()`
  * No need to create all the event files

<!-- 
vim: ts=2:sw=2:sts=2
-->
