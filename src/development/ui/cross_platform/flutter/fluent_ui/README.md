# Fluent UI
Flutter offers the Cuppertino and Material App widget sets for developing IOS and Android apps with 
a look and feel that mimics the native platform. The [Fluent UI](https://pub.dev/packages/fluent_ui) 
package, although 3rd party, is recommended by the Flutter team and marked as a flutter favorite with 
the express intent of building ***"an app that matches the conventions of Microsoft's Fluent design 
system"***. Microsoft even directly offers the [Fluent UI System Icons](https://pub.dev/packages/fluentui_system_icons)
for this purpose as well. Finally the [bitsdogo window](https://pub.dev/packages/bitsdojo_window) 
provides support for *owner drawn* title bars, allowing you to replace the standard Windows title bar 
with a custom one that matches the rest of your app.

Using these two packages together you'll find support for visuals and controls that are commonly 
found in modern Windows apps, including navigation views, content dialogs, flyouts, date pickers, and 
tree view widgets.

**Resources**
* [Flutter and Windows](https://docs.flutter.dev/platform-integration/windows/building)
* [Fluent UI - Medium](https://medium.com/nybles/working-with-fluent-ui-to-create-native-like-windows-app-a8e191828c2a)
* [Bits Dojo windowing video](https://www.youtube.com/watch?v=bee2AHQpGK4)
* [package - bitsdogo window](https://pub.dev/packages/bitsdojo_window) 
* [package - Fluent UI](https://pub.dev/packages/fluent_ui) 
* [package - Fluent UI System Icons](https://pub.dev/packages/fluentui_system_icons) 

## bitdojo window
1. Create a new project
   ```bash
   $ flutter create --platforms=linux bitsdojo_windowing
   $ cd bitsdojo_windowing
   $ flutter pub add bitsdojo_window
   ```
2. Remove system window decorations
   1. Edit `linux/my_application.cc`
   2. Add to the top of the file `#include <bitsdojo_window_linux/bitsdojo_window_plugin.h>`
   3. Find the `gtk_window_set_default_size(window, 1280, 720);` line and add the following before
      ```
      auto bdw = bitsdojo_window_from(window);            // <-- add this line
      bdw->setCustomFrame(true);                          // <-- add this line
      //gtk_window_set_default_size(window, 1280, 720);   // <-- comment this line
      ```
3. Make your main function in `main.dart` look like
   ```dart
   void main() {
     runApp(const MyApp());

     doWhenWindowReady(() {
       const initialSize = Size(600, 450);
       appWindow.minSize = initialSize;
       appWindow.size = initialSize;
       appWindow.alignment = Alignment.center;
       appWindow.title = "Custom title"
       appWindow.show();
     });
   }
   ```
4. Set custom window title
   ```dart
       appWindow.show();
   ```

<!-- 
vim: ts=2:sw=2:sts=2
-->
