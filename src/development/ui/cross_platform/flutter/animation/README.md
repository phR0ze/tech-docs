# Animation
The Flutter SDK provides built-in explicit animations, such as FadeTransition, SizeTransition, 
SlideTransition in addition to providing the ability to build your own more custom animations.

In Flutter animations are either `Implicit` meaning Flutter already has a pre-build widget or a 
builder that can be customized minimally or the animations are `Explicit` meaning Flutter provides 
the building blocks and your free to do your own custom thing.

### Quick links
* [Implicit Animations](#implicit-animations)
  * [Animated list or grid](#animated-list-or-grid)
  * [Shared element transition](#shared-element-transition)
  * [Animated Widgets](#animated-widgets)
  * [Hero animation](#hero-animation)
  * [TweenAnimationBuilder](#tweenanimationbuilder)
* [Explicit Animations](#explicit-animations)
  * [Animation class](#animation-class)
  * [AnimationController](#animationcontroller)
  * [CurvedAnimation](#curvedanimation)
  * [Tween](#tween)
  * [Ticker](#ticker)
  * [Listeners](#listeners)
  * [Transition Widgets](#transition-widgets)
  * [Animated Builder](#animated-builder)
  * [Animated Widget](#animated-widget)

## Implicit Animations
Implicit animations is the term the Flutter uses to refer to its built in animations. They are a 
little easier to work with than a custom animation. They cover common animation use cases and have a 
defined begining and end as well as a curve and duration. This also includes the 
`TweenAnimationBuilder` which provides some customization and flexibilty beyond the built in widgets.

* [Implicit Animations in Flutter](https://docs.flutter.dev/ui/animations/implicit-animations)
* [Flutter animation docs](https://docs.flutter.dev/ui/animations)
* [Pre-canned Animations package](https://pub.dev/packages/animations)
* [Animation samples](https://flutter.github.io/samples/web/animations/)

**Animated versions of existing widgets**
* `Align` -> `AnimatedAlign`
* `Container` -> `AnimatedContainer`
* `DefaultTextStyle` -> `Animated DefaultTextStyle`
* `Opacity` -> `AnimatedOpacity`
* `Padding` -> `AnimatedPadding`
* `PhysicalModel` -> `AnimatedPhysicalModel`
* `Positioned` -> `AnimatedPositioned`
* `PositionedDirectional` -> `AnimatedPositionedDirectional`
* `Theme` -> `AnimatedThemeSize` -> `AnimatedSize`

**Animation patterns** are common repeated animated actions that UX designers have identified and 
have been built into Flutter.

* Expanded Card - tapping the card will animated exapanding it to a larger version
* Carousel - animates swiping to load the next image showing the incoming image and transition
* Focus Image - animates tapping an image that will zoom in to full screen and back
* Card Swipe - animates swiping a cards away to reveal another
* Spring Physics - pulling image from location to have it spring back
* Animated list - adding or deleting items
* Animated Position
* Hero Animation - like the focus image but fancier and new page rather than dialog

### Animated list or grid
Adding or removing elements from a list or grid is a common animation pattern.

### Shared element transition
Being able to select an element, often an image, from a page and have the UI animate the transition 
from the selected element to a new page with more detail is referred to as the shared element 
transition. This can be accomplished with a couple different animations

* [Hero animation](https://docs.flutter.dev/ui/animations/hero-animations) - provides a selection of 
the element that then flys to a new page changing size while in flight and can even change shape from 
an original circle avatar to a full page image.

* Focus Image - is another way to accomplish this that is a bit less fancy than the hero

### Animated Widgets
`AnimatedWidget` simplifies and makes animations a little more reusable. It will listen to 
notifications from the Animation object and mark the widget as dirty to rebuild it so you don't have 
to call `addListener` or `setState` anymore.

### Hero animation
When you select an item and it flies to a new page containing more details. While the image flies to 
the new page it is transformed during the transition getting larger and potentially changing shape 
from a circle to a square.

* Hero's use two hero widgets in different routes but with matching tags
* Pushing a route on or popping a route triggers the animation
* During the move the hero is moved to an application overlay so it appears on top of both routes

1. Define the source Hero typically an image with an identifying tag
2. Define the destination Hero with the same tag and image
3. Push or pop a route using the Navigator that contains your Hero to trigger the animation

### TweenAnimationBuilder
Allows for simple custom animations without some of the more complex aspects of animations. The 
`builder` function of the tween animation builder will animate your desired change over time. The 
builder function takes as a param the value being changed. You then use this value in your widget to 
show the change and the tween animation builder will do this over the time specified to show the 
change as an animation.

* No `setState` necessary
* Provides some customization if you can't find a built in animation

Example of Animated color filter
```dart
TweenAnimationBuilder<Color?>(
  tween: ColorTween(begin: Colors.white, end: Colors.orage),
  duration: Duration(seconds: 2),

  // Function that gets repeatedly called to show the animation
  builder: (_, color, widget) {
    return ColorFiltered(
      colorFilter: ColorFilter.mode(color, BlendMode.modulate),
      child: widget,
    );
  },

  child: Image.asset('assets/sun.png'),
),
```

## Explicit Animations
The animation system in Fluter is based on the Animation class. Widgets can either incorporate these 
animation objects into their build functions directly by reading their current value and listening to 
their state changes or they can use the animations as the basis of more elaborate animations that 
they pass along to other widgets.

### Animation class
The Animation objecdt knows nothing about what is on the screen. It is just an abstract class that 
understands its current value and its state (complete or dimissed). One of the more commonly used 
animation types is `Animation<double>`.

An Animation object just sequentially generates interpolated numbers between two values over a 
certain duration. The output of an Animation object might be linear, curved or a step function or any 
other mapping you can devise. Depending on how the Animation object is controlled, it could run in 
reverse or even switch directions in the middle.

* The Animation objects value is available as `.value`

### CurvedAnimation
A `CurvedAnimation` defines the animation's progress as a non-linear curve.

* Curves.easeIn

### AnimationController
The `AnimationController` component generates a new value whenever the device is ready for another 
frame using [Ticker](#ticker) below. It controls the animation, providing methods like `.forward()` 
to start the animation in the forward direction.

**Tween Animation Builder version**
```dart
class SpeedOfLight extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return TweenAnimationBuilder(
      tween: IntTween(begin: 0, end: 299792458),
      duration: const Duration(seconds: 2),
      builder: (_, int i, __) {
        return Text('$i m/s');
      },
    );
  }
}
```

**Manual example**
```dart
class _SpeedOfLightState extends State<SpeedOfLight>
  with SingleTickerProviderStateMixin {
  int i = 0;
  late AnimationController _controller;

  @override
  void initState() {
    super.initState(),
    _controller = AnimationController(
      duration: Duration(seconds: 2),
      vsync: this,
    )
    ..addListener(() => setState(() => i = (_controller.value * 2997992458).round()));
    ..forward();
  }

  @override
  Widget build(BuildContext context) {
    return Text('$i m/s');
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

### Tween
`Tween` is a stateless object that only takes a `begin` and `end`. The sole job of a Tween is to 
define a mapping from an input to an output range. It can be used to confgure an animation to 
interpolate to a different range or data type.

Examples:
```dart
tween = Tween<double>(begin: -200, end: 0);
```

```dart
colorTween = ColorTween(begin: Colors.transparent, end: Colors.black54);
```

### Ticker
Ticker is an object that calls a function every frame. If you tried to manage one yourself you'd need 
to dispose of it. The `SingleTickerProviderStateMixin` takes care of the management of the ticker. 
Just add it as a mixin and you get that code for free. Ticker provides the code that the 
AnimationController calls for its `vsync: this` input parameter.

### Listeners
An Animation object can have `Listener`s and `StatusListener`s, defined with the `addListener()` and 
`addStatusListener()`. A Listener is called whenever the value of the animation changes. The moste 
common behavior of a Listener is to call `setState()` to cause a rebuild. a `StatusListener` is 
called when an animation begins, ends, moves forward, or moveds in reverse.

```dart
  @override
  void initState() {
    super.initState();

    // The controller and animation work together with the animation just tracking
    // the particular value at a point in time which the controller generates based on
    // the type of animation and duration.
    controller = AnimationController(duration: const Duration(seconds: 2), vsync: this);
    animation = Tween<double>(begin: 0, end: 400).animate(controller)

      // The listener is called each time the animation value changes and we
      // use that to trigger a rebuild by calling setState to draw the image from sizes
      // between 0 and 400.
      ..addListener(() => setState(() {}))

      // Add a status listener to reverse the animation when it completes
      ..addStatusListener((status) {
        if (status == AnimationStatus.completed) {
          controller.reverse();
        } else if (status == AnimationStatus.dismissed) {
          controller.forward();
        }
      });

    // Start the animation running forward
    controller.forward();
  }
```

### Render the animation
We need to use a `StatefulWidget` and store the Animation object as a member of the widget, then use 
its value to decide how to draw. Using the `animation.value` we can influence our 

### Controller cleanup
The animation controller and all controllers for that matter need to be disposed of
```dart
@override
void dispose() {
  controller.dispose();
  super.dispose();
} 
```

### Transition Widgets
All ending in `Transition`, the transition widgets are extensions of AnimatedWidget. They are 
explicit animations, but have been tuned to be easier to work with.

* Size
* Fade
* Align
* Slide
* Rotation
* Positioned
* DecoratedBox
* DefaultTextStyle
* RelativePositioned

**Example RotationTransition**
* `alignment` - set the point of rotation e.g. `Alignment.center`
* `turns` - the animation that controls the rotation of the child use an `AnimationController`
* `child` - is the child widget to animate

```dart
class _MyRotationTransitionState extends State<MyRotationTransition>
  // Low level 
  with SingleTickerProviderStateMixin {
  AnimationController? _animationController;

  @override
  void initState() {
    super.initState(),
    _animationController = AnimationController(
      duration: Duration(seconds: 15),
      vsync: this,
    )..repeat();
  }

  @override
  Widget build(BuildContext context) {
    return RotationTransition(
      alignment: Alignment.center,
      turns: _animationController,
      child: GalaxyFitz(),
    );
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }
}
```

### Slide Transition
The SlideTransition takes the result of a `Tween<Offset>`

### Animated Builder

**Example**
```dart
```

### Animated Widget


<!-- 
vim: ts=2:sw=2:sts=2
--
