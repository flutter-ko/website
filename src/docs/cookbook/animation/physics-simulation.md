---
title: 물리 시뮬레이션 이용하여 위젯에 애니메이션 주기
prev:
  title: Animate a page route transition
  path: /docs/cookbook/animation/page-route-animation
next:
  title: Animate the properties of a container
  path: /docs/cookbook/animation/animated-container
---

물리 시뮬레이션(Physics simulations)으로 앱의 인터렉션을 현실감 있고 인터랙티브하게 만들 수 있습니다.
예를 들어, 위젯에 스프링에 부착되거나 중력에 의해 떨어지는 것처럼 작동하는 
애니메이션을 만들 수 있습니다.

이 예제는 스프링 시뮬레이션을 사용하여 위젯을 
드래그 된 지점에서 중심으로 다시 이동하는 방법을 보여줍니다.

이 예제는 아래와 같은 단계로 진행됩니다:

1. 애니메이션 컨트롤러 설정
2. 제스처를 사용하여 위젯 이동
3. 위젯 애니메이션 적용
4. 스프링 운동을 시뮬레이션하기 위한 속도 계산


## Step 1: 애니메이션 컨트롤러 설정

상태가 있는 위젯(stateful widget)인 `DraggableCard`와 함께 시작하세요:

```dart
import 'package:flutter/material.dart';

main() {
  runApp(MaterialApp(home: PhysicsCardDragDemo()));
}

class PhysicsCardDragDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: DraggableCard(
        child: FlutterLogo(
          size: 128,
        ),
      ),
    );
  }
}

class DraggableCard extends StatefulWidget {
  final Widget child;
  DraggableCard({this.child});

  @override
  _DraggableCardState createState() => _DraggableCardState();
}

class _DraggableCardState extends State<DraggableCard> {
  @override
  void initState() {
    super.initState();
  }

  @override
  void dispose() {
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Align(
      child: Card(
        child: widget.child,
      ),
    );
  }
}
```

[SingleTickerProviderStateMixin][]에서 `_DraggableCardState` 클래스를 확장하세요.
그런 다음 `initState`에서 [AnimationController][]를 생성하고
`this`에 `vsync`를 설정하세요.

{{site.alert.note}}
  `SingleTickerProviderStateMixin`을 확장하면 상태 객체가 
  `AnimationController`의 `TickerProvider`가 될 수 있습니다.
  더 많은 정보를 원하시면 [TickerProvider][]를 참조하세요.
{{site.alert.end}}

```dart
class _DraggableCardState extends State<DraggableCard>
    with SingleTickerProviderStateMixin {
  AnimationController _controller;

  @override
  void initState() {
    _controller =
        AnimationController(vsync: this, duration: Duration(seconds: 1));
    super.initState();
  }


  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  //...
```

## Step 2: 제스처를 사용하여 위젯 이동

Make the widget move when it's dragged, and add an [Alignment][] field to the
`_DraggableCardState` class:

```dart
Alignment _dragAlignment = Alignment.center;
```

Add a [GestureDetector][] that handles the `onPanDown`, `onPanUpdate`, and
`onPanEnd` callbacks. To adjust the alignment, use a [MediaQuery][] to get the
size of the widget, and divide by 2. (This converts units of "pixels dragged" to
coordinates that [Align][] uses.) Then, set the `Align` widget's `alignment` to
`_dragAlignment`:

```dart
@override
Widget build(BuildContext context) {
  var size = MediaQuery.of(context).size;
  return GestureDetector(
    onPanDown: (details) {},
    onPanUpdate: (details) {
      setState(() {
        _dragAlignment += Alignment(
          details.delta.dx / (size.width / 2),
          details.delta.dy / (size.height / 2),
        );
      });
    },
    onPanEnd: (details) {},
    child: Align(
      child: Card(
        child: widget.child,
      ),
    ),
  );
}
```

## Step 3: 위젯 애니메이션 적용

When the widget is released, it should spring back to the center.

Add an `Animation<Alignment>` field and an `_runAnimation` method. This
method defines a `Tween` that interpolates between the point the widget was
dragged to, to the point in the center.

```dart
  Animation<Alignment> _animation;

  void _runAnimation() {
    _animation = _controller.drive(
      AlignmentTween(
        begin: _dragAlignment,
        end: Alignment.center,
      ),
    );
   _controller.reset();
   _controller.forward();
  }
```

Next, update `_dragAlignment` when the `AnimationController` produces a
value:

```dart
@override
void initState() {
  super.initState();
  _controller = AnimationController(vsync: this, duration: Duration(seconds: 1));
  _controller.addListener(() {
    setState(() {
      _dragAlignment = _animation.value;
    });
  });
}
```

Next, make the `Align` widget use the `_dragAlignment` field:

```dart
child: Align(
  alignment: _dragAlignment,
  child: Card(
    child: widget.child,
  ),
),
```


Finally, update the `GestureDetector` to manage the animation controller:


```dart
onPanDown: (details) {
 _controller.stop();
},
onPanUpdate: (details) {
 setState(() {
   _dragAlignment += Alignment(
     details.delta.dx / (size.width / 2),
     details.delta.dy / (size.height / 2),
   );
 });
},
onPanEnd: (details) {
 _runAnimation();
},
```

## Step 4: 스프링 운동을 시뮬레이션하기 위한 속도 계산

The last step is to do a little math, to calculate the velocity of the widget
after it's finished being dragged. This is so that the widget realistically
continues at that speed before being snapped back. (The `_runAnimation` method
already sets the direction by setting the animation's start and end alignment.)

First, import the `physics` package:

```dart
import 'package:flutter/physics.dart';
```

The `onPanEnd` callback provides a [DragEndDetails][] object. This object
provides the velocity of the pointer when it stopped contacting the screen. The
velocity is in pixels per second, but the `Align` widget doesn't use pixels. It
uses coordinate values between [-1.0, -1.0] and [1.0, 1.0], where [0.0, 0.0]
represents the center. The `size` calculated in step 2 is used to convert pixels
to coordinate values in this range.

Finally, `AnimationController` has an `animateWith()` method that can be given a
[SpringSimulation][]:

```dart
void _runAnimation(Offset pixelsPerSecond, Size size) {
  _animation = _controller.drive(
    AlignmentTween(
      begin: _dragAlignment,
      end: Alignment.center,
    ),
  );
  // Calculate the velocity relative to the unit interval, [0,1],
  // used by the animation controller.
  final unitsPerSecondX = pixelsPerSecond.dx / size.width;
  final unitsPerSecondY = pixelsPerSecond.dy / size.height;
  final unitsPerSecond = Offset(unitsPerSecondX, unitsPerSecondY);
  final unitVelocity = unitsPerSecond.distance;

  const spring = SpringDescription(
    mass: 30,
    stiffness: 1,
    damping: 1,
  );

  final simulation = SpringSimulation(spring, 0, 1, -unitVelocity);

  _controller.animateWith(simulation);
}
```

Don't forget to call `_runAnimation()`  with the velocity and size:

```dart
onPanEnd: (details) {
  _runAnimation(details.velocity.pixelsPerSecond, size);
},
```

{{site.alert.note}}
  Now that the animation controller uses a simulation it's `duration` argument
  is no longer required.
{{site.alert.end}}

## Complete Example

```dart
import 'package:flutter/material.dart';
import 'package:flutter/physics.dart';

main() {
  runApp(MaterialApp(home: PhysicsCardDragDemo()));
}

class PhysicsCardDragDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: DraggableCard(
        child: FlutterLogo(
          size: 128,
        ),
      ),
    );
  }
}

/// A draggable card that moves back to [Alignment.center] when it's
/// released.
class DraggableCard extends StatefulWidget {
  final Widget child;
  DraggableCard({this.child});

  @override
  _DraggableCardState createState() => _DraggableCardState();
}

class _DraggableCardState extends State<DraggableCard>
    with SingleTickerProviderStateMixin {
  AnimationController _controller;

  /// The alignment of the card as it is dragged or being animated.
  ///
  /// While the card is being dragged, this value is set to the values computed
  /// in the GestureDetector onPanUpdate callback. If the animation is running,
  /// this value is set to the value of the [_animation].
  Alignment _dragAlignment = Alignment.center;

  Animation<Alignment> _animation;

  /// Calculates and runs a [SpringSimulation].
  void _runAnimation(Offset pixelsPerSecond, Size size) {
    _animation = _controller.drive(
      AlignmentTween(
        begin: _dragAlignment,
        end: Alignment.center,
      ),
    );
    // Calculate the velocity relative to the unit interval, [0,1],
    // used by the animation controller.
    final unitsPerSecondX = pixelsPerSecond.dx / size.width;
    final unitsPerSecondY = pixelsPerSecond.dy / size.height;
    final unitsPerSecond = Offset(unitsPerSecondX, unitsPerSecondY);
    final unitVelocity = unitsPerSecond.distance;

    const spring = SpringDescription(
      mass: 30,
      stiffness: 1,
      damping: 1,
    );

    final simulation = SpringSimulation(spring, 0, 1, -unitVelocity);

    _controller.animateWith(simulation);
  }

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);

    _controller.addListener(() {
      setState(() {
        _dragAlignment = _animation.value;
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final size = MediaQuery.of(context).size;
    return GestureDetector(
      onPanDown: (details) {
        _controller.stop();
      },
      onPanUpdate: (details) {
        setState(() {
          _dragAlignment += Alignment(
            details.delta.dx / (size.width / 2),
            details.delta.dy / (size.height / 2),
          );
        });
      },
      onPanEnd: (details) {
        _runAnimation(details.velocity.pixelsPerSecond, size);
      },
      child: Align(
        alignment: _dragAlignment,
        child: Card(
          child: widget.child,
        ),
      ),
    );
  }
}
```

![Demo showing a widget being dragged and snapped back to the center](/images/cookbook/animation-physics-card-drag.gif){:.site-mobile-screenshot}

[Align]: {{site.api}}/flutter/widgets/Align-class.html
[Alignment]: {{site.api}}/flutter/painting/Alignment-class.html
[AnimationController]: {{site.api}}/flutter/animation/AnimationController-class.html
[GestureDetector]: {{site.api}}/flutter/widgets/GestureDetector-class.html
[SingleTickerProviderStateMixin]: {{site.api}}/flutter/widgets/SingleTickerProviderStateMixin-mixin.html
[TickerProvider]: {{site.api}}/flutter/scheduler/TickerProvider-class.html
[MediaQuery]: {{site.api}}/flutter/widgets/MediaQuery-class.html
[DragEndDetails]: {{site.api}}/flutter/gestures/DragEndDetails-class.html
[SpringSimulation]: {{site.api}}/flutter/physics/SpringSimulation-class.html
