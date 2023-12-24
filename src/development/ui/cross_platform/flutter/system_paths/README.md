# System Paths
Cross platform system paths is important to get right to make sure you data is ending up in the right 
place for the right reasons. Some locations like the temp directory are not durable and will get 
cleaned out on reboot. Other locations have specific meanings on different systems.

## Existing packages

* [`path_provider`](https://pub.dev/packages/path_provider)
  * Cross platform and mostly usable
* [`xdg_directories`](https://pub.dev/packages/xdg_directories)
  * Linux only, but defines all XDG directories

## Linux XDG specification
The XDG base directory specification documents a common usage recommendation across linux to unify 
the linux world more. The [Arch Linux docs describe it as](https://wiki.archlinux.org/title/XDG_Base_Directory)

* ***XDG_CONFIG_HOME***
  * Where user specific configuration should be written (analogous to `/etc`)
  * Defaults to `~/.config`
* ***XDG_CACHE_HOME***
  * Where user specific non-essential (cached) data should be written (analogous to `/var/cache`)
  * Defaults to `~/.cache`
* ***XDG_DATA_HOME***
  * Where user specific data files should be written (analogous to `/usr/share`)
  * Default to `~/.local/share`
* ***XDG_STATE_HOME***
  * Where user specific state files should be written (analogous to `/var/lib`)
  * Defaults to `~/.local/state`
* ***XDG_RUNTIME_DIR***
  * Used for non-essential, user specific data files such as sockets, named pipes
  * May be subject to periodic cleanup
  * Only exists for the duration of the user login
  * Should not store large files

## Path provider package
The [`path_provider`](https://pub.dev/packages/path_provider) package makes some cross platform 
directories available for use. Its a good example of a fully federated plugin implementation. The 
linux implementation seems to be lacking some notables imo though; namely the user configuration 
directory e.g. `~/.config/<project_name>`

That said `getApplicationSupportDirectory` is probably the best candidate for persisting 
configuration and data for your application.

### Path provider linux
These paths are influenced by the XDG specification so might be different on your flavor of linux.

| function                             | linux                            |
| ------------------------------------ | -------------------------------- | 
| `getApplicationCacheDirectory()`     | `~/.cache/<project_name>`        |
| `getApplicationDocumentsDirectory()` | `~/Documents`                    |
| `getApplicationSupportDirectory()`   | `~/.local/share/<project_name>`  |
| `getDownloadsDirectory()`            | `~/Downloads`                    |
| `getLibraryDirectory()`              | `Not Implemernted`               |
| `getTemporaryDirectory()`            | `/tmp`                           |
 
## Project ID
Many path packages use the project id as a means of differentiating where your data files are stored. 
However by default this identifier, coming from the Java world, is confusing and complicated e.g. 
`com.example.riverpod_movies`. It can be changed in various locations for the different system 
platforms your supporting.

### Change linux project id
1. Edit `linux/CMakeLists.txt`
2. Change `set(APPLICATION_ID "com.example.riverpod_movies")`
3. to `set(APPLICATION_ID "riverpod_movies")`
4. This makes `getApplicationSupportDirectory` resolve to `/home/<user>/.local/share/riverpod_movies`


<!-- 
vim: ts=2:sw=2:sts=2
-->
