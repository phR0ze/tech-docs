# Gestures
The gesture system in Flutter has two separate layers. The first layer has raw pointer events that 
describe the location and movement of pointers across the screen. The second layer has gestures that 
describe the semantic actions that consist of one or more pointer movements.

* [Flutter taps, drags and other gestures](https://docs.flutter.dev/ui/interactivity/gestures)

### Quick links
* [Dismissible](#dismissible)
* [InteractiveViewer](#interactiveviewer)
* [PageView](#PageView)

## Pointers
Pointers represent the raw data about the user's interaction with the screen. There are four types of 
pointer events.

* `PointerDownEvent` - the pointer has contacted the screen at a particular location
* `PointerMoveEvent` - the pointer has moved from one location on the screen to another
* `PointerUpEvent` - the pointer has stopped contacting the screen
* `PointerCancelEvent` - input from this pointer is no longer directed toward this app

You can directly listen for pointer events with a `Listener` widget. Listeners can be setup to listen 
to completely raw unprocessed event information from the system. You can subscribe to pointer events 
like `onPointerDown`, `onPointerUp` and `onPointerMove`.

### Which widget gets the touch
On pointer down, the framework does a `hit test` on your app to determine which widget exists at the 
location where the pointer contacted the screen. The pointer down event and subsequent events are 
then dispatched to the innermost wiget found by the hit test. There is no mechanism for canceling or 
stopping pointer events from being dispatched further.

## Gesture Arena
At a given location on the screen, there might be multiple gesture detectors. All of the gesture 
detectors listen to the stream of pointer events as they flow past and attempt to recognize specific 
gestures. The framework disambiguates which gesture the user intends by having each recognizer join 
the gesture arena. The gesture arena determines which gesture wins using the following rules.

* At any time a recognizer can leave the arena. The single remaining recognizer wins.
* At any time a recognier can declare itself the winner, causing all of the other recognizers to win.

* [Decoding Flutter - Gesture Arena](https://www.youtube.com/watch?v=Q85LBtBdi0U)

* Max time in the arena is 500ms i.e. double tap
* Child's recognizers will always win as they get the first shot

## Gesture Detectors
Gesture detectors detect gestures using gesture recognizers then maps them to callbacks. Think of the 
detector as the controller and the recognizer as the specific pointer events to gesture interpreter.
`GestureDetector` is a widget that does just this. It wraps whatever widget you give it as its child 
and intercepts touch for that widget. GestureDetector's with invisible children ignores touch, but 
can be overriddent with the `behavior` property.

* 95% of everything you'll ever want to do is covered by the GestureDetector widget

## Raw Gesture Detector
The `RawGestureDetector` is what the GestureDetector uses and is what you'll use for the other 5% of 
gesturess that aren't supported by the GestureDetector and you'd like to implement.

* [Understanding Flutter's Gestures - Youtube](https://www.youtube.com/watch?v=zEoASR7DTIw)

All gestures can be decomposed to a sequence of pointer events.

## Gesture Recognizers
Gesture recognizers know how to tell if raw pointer event information is a gesture it recognizes.
`GestureRecognizer` is the base class that all gesture recognizers inherit from. It provides a basic 
API that can be used by classes that work with gesture recognizers but don't care about the specific 
details of the gesture recognizers themselves.

* [flutter docs](https://api.flutter.dev/flutter/gestures/gestures-library.html)
![Flutter Gestures](../../../../data/images/flutter-gestures-mind-map.png)

When building your own gesture recognizer you can build off from the closest existing recognizer.

* Base Tap And Drag
* Base Tap
* Delayed Multi Drag
* Double Tap
  * After single tap appears it sets a 300ms timer
    * If no second tap appears it admits defeat
    * If it sees a tap within the window it declars victory
* Drag
* Eager
* Horizontal Drag
* Horizontal Multi Drag
* Immediate Multi Drag
* Long Press
  * After single tap appears it sets a 500ms timer
    * If user is still holding then it declars victory
    * If pointer up event occurs before elapse time it admits defeat
* Multi Drag
* Multi Tap
* One Sequence
* Pan Gesture
* Primary Pointer
* Scale Gesture
* Serial Tap
* Tap and Horizontal drag
* Tap and pan
* Tap
  * Never declares victory
  * Only admits defeat if the pointer strays too far from the down location before a pointer up event 
  is received
* Vertical Drag
* Vertical Multi Drag

## Gesture aware widgets
Working with raw gestures is awesome but quite complicated to get right. The easiest and more 
reliable way to use gestures is to use the built in widgets that provide specific gestures for you. 
This is usually must more reliable and gives you nice animations for free.

**References**
* [Choose the right gesture widgets](https://blog.logrocket.com/choosing-the-right-gestures-for-your-flutter-project/)
* [Vertical Drag](https://blog.logrocket.com/handling-gestures-flutter-gesturedetector/)

## Dismissible
Provides an animated slide out of view via the swipe or flinging gesture. Its really nice and 
convenient however it doesn't seem to work well with the InteractiveViewer's Zoom

* Works well with PageView's swipe scrolling
* Breaks InteractiveViewer's pinch to zoom

## InteractiveViewer
[InteractiveViewer](https://api.flutter.dev/flutter/widgets/InteractiveViewer-class.html)
is a widget that enables pan and zoom interactions with its child. It works greate as a standalone 
but as soon as you wrap it in a PageView or Dismissible you get problems with pinch to zoom

## PageView
PageView provides a nice way to swipe to load the next page.

* Works well with Dismissible's swipe scrolling
* Breaks InteractiveViewer's pinch to zoom


<!-- 
vim: ts=2:sw=2:sts=2
-->
