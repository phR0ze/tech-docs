# BLoC

BLoC benefits
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

