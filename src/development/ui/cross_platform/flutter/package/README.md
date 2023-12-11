# Package
[Build your own Flutter package](https://docs.flutter.dev/packages-and-plugins/developing-packages) 
to share code or make something resuable across projects.

## Getting started
1. Create a `pubspec.yaml`
2. Create a `lib` folder


## Example project
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
