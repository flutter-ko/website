---
title: Named route로의 화면 전환
prev:
  title: 새로운 화면으로 이동하고, 되돌아오기
  path: /docs/cookbook/navigation/navigation-basics
next:
  title: 인자를 named route로 전달하기
  path: /docs/cookbook/navigation/navigate-with-arguments
js:
  - defer: true
    url: https://dartpad.dev/inject_embed.dart.js
---

In the [Navigate to a new screen and back][] recipe,
you learned how to navigate to a new screen by creating a new route and
pushing it to the [`Navigator`][].

하지만 만약 앱의 다른 많은 부분들에서 동일한 화면으로 이동하고자 한다면, 중복된 코드가
생기게 됩니다. 이러한 경우 _named route_ 를 정의하여 화면 전환에 사용하는 방법이 해결책이
될 수 있습니다.

To work with named routes,
use the [`Navigator.pushNamed()`][] function.
This example replicates the functionality from the original recipe,
demonstrating how to use named routes using the following steps:

  1. 두 개의 화면 만들기.
  2. Route 정의하기.
  3. `Navigator.pushNamed`를 사용하여 두 번째 화면으로 전환하기.
  4. `Navigator.pop`을 사용하여 첫 번째 화면으로 돌아가기.

## 1. 두 개의 화면 만들기

우선, 본 예제를 시작하기 위해 두 개의 화면이 필요합니다. 첫 번째 화면에는 두 번째 화면으로 
이동하기 위한 버튼 하나가 있고, 두 번째 화면에는 다시 첫 번째 화면으로 돌아가기 위한 버튼이
있습니다.

```dart
class FirstScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('First Screen'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Launch screen'),
          onPressed: () {
            // 클릭하면 두 번째 화면으로 전환합니다!
          },
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Second Screen"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            // 클릭하면 첫 번째 화면으로 돌아갑니다!
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

## 2. Route 정의하기

Next, define the routes by providing additional properties
to the [`MaterialApp`][] constructor: the `initialRoute`
and the `routes` themselves.

`initialRoute` 프로퍼티는 앱의 시작점을 나타내는 route를 정의하고, `routes` 프로퍼티는 이용가능한 
named route와 해당 route로 이동했을 때 빌드될 위젯을 정의합니다.

<!-- skip -->
```dart
MaterialApp(
  // "/"으로 named route와 함께 시작합니다. 본 예제에서는 FirstScreen 위젯에서 시작합니다.
  initialRoute: '/',
  routes: {
    // "/" Route로 이동하면, FirstScreen 위젯을 생성합니다.
    '/': (context) => FirstScreen(),
    // "/second" route로 이동하면, SecondScreen 위젯을 생성합니다.
    '/second': (context) => SecondScreen(),
  },
);
```

{{site.alert.warning}}
  `initialRoute`를 사용한다면, `home`프로퍼티를 정의하지 **마세요**.
{{site.alert.end}}

## 3. 두 번째 화면으로 전환하기

With the widgets and routes in place, trigger navigation by using the
[`Navigator.pushNamed()`][] method.
This tells Flutter to build the widget defined in the
`routes` table and launch the screen.

`FirstScreen` 위젯의 `build()` 메서드에 `onPressed()` 콜백을 다음과 같이 수정하세요:

<!-- skip -->
```dart
// `FirstScreen` 위젯의 콜백
onPressed: () {
  // Named route를 사용하여 두 번째 화면으로 전환합니다.
  Navigator.pushNamed(context, '/second');
}
```

## 4. 첫 번째 화면으로 돌아가기

To navigate back to the first screen, use the
[`Navigator.pop()`][] function.

<!-- skip -->
```dart
// SecondScreen 위젯의 콜백
onPressed: () {
  // 현재 route를 스택에서 제거함으로써 첫 번째 스크린으로 되돌아 갑니다.
  Navigator.pop(context);
}
```

## Interactive example

```run-dartpad:theme-light:mode-flutter:run-true:width-100%:height-600px:split-60
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    title: 'Named routes Demo',
    // "/"을 앱이 시작하게 될 route로 지정합니다. 본 예제에서는 FirstScreen 위젯이 첫 번째 페이지가
    // 될 것입니다.
    initialRoute: '/',
    routes: {
      // When we navigate to the "/" route, build the FirstScreen Widget
      // "/" Route로 이동하면, FirstScreen 위젯을 생성합니다.
      '/': (context) => FirstScreen(),
      // "/second" route로 이동하면, SecondScreen 위젯을 생성합니다.
      '/second': (context) => SecondScreen(),
    },
  ));
}

class FirstScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('First Screen'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Launch screen'),
          onPressed: () {
            // Named route를 사용하여 두 번째 화면으로 전환합니다.
            Navigator.pushNamed(context, '/second');
          },
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Second Screen"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            // 현재 route를 스택에서 제거함으로써 첫 번째 스크린으로 되돌아 갑니다.
            Navigator.pop(context);
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

<noscript>
  <img src="/images/cookbook/navigation-basics.gif" alt="Navigation Basics Demo" class="site-mobile-screenshot" />
</noscript>


[`MaterialApp`]: {{site.api}}/flutter/material/MaterialApp-class.html
[Navigate to a new screen and back]: /docs/cookbook/navigation/navigation-basics
[`Navigator`]: {{site.api}}/flutter/widgets/Navigator-class.html
[`Navigator.pop()`]: {{site.api}}/flutter/widgets/Navigator/pop.html
[`Navigator.pushNamed()`]: {{site.api}}/flutter/widgets/Navigator/pushNamed.html
