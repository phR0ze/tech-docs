# Gestures <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

The gesture system in Flutter has two separate layers. The first layer has raw pointer events that 
describe the location and movement of pointers across the screen. The second layer has gestures that 
describe the semantic actions that consist of one or more pointer movements.

* [Flutter taps, drags and other gestures](https://docs.flutter.dev/ui/interactivity/gestures)

### Quick links
* [Pointers](#pointers)
  * [Pointer Events](#pointer-events)
  * [Which widget gets the touch](#which-widget-gets-the-touch)
* [Gesture Arena](#gesture-arena)
* [Gesture Detectors](#gesture-detectors)
* [Raw Gesture Detector](#raw-gesture-detector)
* [Gesture Recognizer](#gesture-recognizer)
* [Dismissible](#dismissible)
* [InteractiveViewer](#interactiveviewer)
* [PageView](#PageView)

## Pointers

### Pointer Events
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

### Nesting of gesture detectors
The gesture arena explanation seems simply enough; however when you start to nest gesture detectors 
such as wrapping an `InteractiveViewer` inside a `PageViewer` inside a `Dismissible` things start to 
break down. By default the `InteractiveViewer` stops responding to pinch to zoom about 80% of the 
time when inside a `PageView`. There seems to be multiple problems with nesting of detectors of this 
sort. They have to be carefully crafted with the correct behaviors set which PageView doesn't have 
and what's worse the base GestureDetector is unable to process scale, drags and pans at the same time 
without some non-intuitive convolutions since the scale is a super set of pan and pan is a super set 
of drag horizontal + drag vertical which means you can't have any at the same time unless you 
carefully and manually track the number of pointers in flight and manually disable different gesture 
callbacks under different circumstances to orchestrate your desired behavior. Although this can be 
done for complex nested gestues it might be better just to build your own gesture recognizer and 
detector.

* [PageView is not click through issue](https://github.com/flutter/flutter/issues/47119)
* [Allow multiple gestures example](https://gist.github.com/Nash0x7E2/08acca529096d93f3df0f60f9c034056)
* [Raw Gesture Detectors seems to be the solution](https://stackoverflow.com/questions/58138114/receive-onverticaldragupdate-on-nested-gesturedetectors-in-flutter)
* [pointerCount=2 for trackpad gestures](https://github.com/flutter/flutter/pull/140745)
* [us scale recognizer instead of pan](https://github.com/flutter/flutter/issues/115061)
* [interactiveViewer blocks gestures](https://github.com/flutter/flutter/issues/136622)
* [onTap is ignored after scale](https://github.com/flutter/flutter/issues/140730)
* [onScale conflicts with PageView](https://github.com/flutter/flutter/issues/68960)
* [conflict between scale and drag gestures](https://github.com/flutter/flutter/issues/13101)
* [recognizers should bid when they win](https://github.com/flutter/flutter/issues/11384)

Its a common enough problem that multiple different packages have solved it in different ways.
Usually using the low level event [Listener] directly or as a wrapper around [GestureDetector].
Others have used the [RawGestureDetector] which is a lower level version of the [GestureDetector].
* [Gesture X Detector](https://pub.dev/packages/gesture_x_detector) uses raw events from Listener
* [Extended Image](https://pub.dev/packages/extended_image) uses custom gesture recognizers

The official solution from Flutter seems to be a combination of the following:
* Use the [pointerCount] added to the [ScaleStartDetails] in this PR
  https://github.com/flutter/flutter/pull/73474 to track how many pointers were involved
* Since the scale gesture is a super set of the pan gesture and the pan gesture is just
  the horizontal drag + the vertical drag we can use the scale callbacks for all three.
  https://github.com/flutter/flutter/blob/fb57da5f945d02ef4f98dfd9409a72b7cce74268/packages/flutter/lib/src/widgets/gesture_detector.dart#L167

Older solutions include using a raw [Listener] to track the number of pointers in play
directly to then decipher which gestures to enable or disable or alternatively ignoring
events in the handlers based on the pointer count.
onScaleStart: _pointers == 2 ? _onScaleStart : null,
onTapUp: _pointers == 1 ? _onTapUp: null,
https://github.com/flutter/flutter/issues/13102

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

* Works well with PageView's swipe scrolling with just the two of them
* Breaks down when used with the InteractiveViewer's pinch to zoom

## InteractiveViewer
[InteractiveViewer](https://api.flutter.dev/flutter/widgets/InteractiveViewer-class.html)
is a widget that enables pan and zoom interactions with its child. It works greate as a standalone 
but as soon as you wrap it in a PageView or Dismissible you get problems with pinch to zoom

## PageView
PageView provides a nice way to swipe to load the next page.

* Works well with Dismissible's swipe scrolling
* Breaks InteractiveViewer's pinch to zoom

