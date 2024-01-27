# Package & Plugins
[Build your own Flutter package](https://docs.flutter.dev/packages-and-plugins/developing-packages) 
to share code or make something resuable across projects.

### Quick links
* [Getting started](#getting-started)
  * [Create a new package](#create-a-new-package)
* [Packages](#packages)
  * [Package basics](#package-basics)

## Getting started
Packages are the fundamental unit of reusability in Dart/Flutter. A package can contain platform 
specific plugins.

### Create a new package
The template example below will create a new package project at your current path. This can be nested 
in other projects or whatever. Packages are just a set of files.

1. Create your package
   ```bash
   $ flutter create --template=package pull_close
   ```

2. Now create an example project for your package
   ```bash
   $ cd pull_close
   $ flutter create --platforms=linux,android pull_close_example
   $ mv pull_close_example example
   ```

## Packages

### Package basics
* `pubspec.yaml` declares the package `name`, `version`, `author` and so on
* `lib` contains the public code in the package, minimally just a single `<package-name>.dart` 

## XDG paths example project
To learn how to create packages I'm going to create a cross platform configuration package for 
persisting basic application configs. 

**Requirements**
* Linux - use the XDG convention configuration and store as `~/.config/<app>/<app>.yaml`
* Android - use the typical Shared Preferences patttern e.g. `shared_preferences` package
  * Accessible with teh `getDefaultSharedPreferences`

1. Use `xdg_directories` and/or `path_provider` to handle pathing
2. Use `global_configuation` singleton pattern
3. Use `cli_config` yaml support concept

<!-- 
vim: ts=2:sw=2:sts=2
-->
