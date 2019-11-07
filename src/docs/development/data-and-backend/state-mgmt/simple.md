---
title: 간단한 앱 상태 관리
prev:
  title: 임시 상태 vs 앱 상태
  path: /docs/development/data-and-backend/state-mgmt/ephemeral-vs-app
next:
  title: 접근 방법 목록
  path: /docs/development/data-and-backend/state-mgmt/options
---

이제 [선언적 UI 프로그래밍](/docs/development/data-and-backend/state-mgmt/declarative)
과 [임시상태와 앱 상태](/docs/development/data-and-backend/state-mgmt/ephemeral-vs-app)의 구분에 대해 배웠으니,
간단한 앱 상태 관리에 대해 배울 준비가 되었습니다.

이 페이지에서는, `provider` 패키지를 사용할것입니다. 
Flutter가 처음이고 Redux, Rx, hooks 등과 같은 다른 접근방법을 사용할 강력한 이유가 없다면, 
이것이 당신이 시작해야 할 접근 방법일 것입니다. `provider`는 이해하기 쉽고, 코드를 많이 사용하지 않습니다.
또한 다른 모든 접근 방식에 적용 할 수 있는 개념을 사용합니다.

즉, 다른 리액티브 프레임워크에서의 상태 관리에 대하여 강력한 배경지식이 있다면 
[다음 페이지](/docs/development/data-and-backend/state-mgmt/options)에서 패키지와 튜토리얼을 찾을 수 있습니다.

## 우리의 예제 {% asset development/data-and-backend/state-mgmt/model-shopper-screencast alt="An animated gif showing a Flutter app in use. It starts with the user on a login screen. They log in and are taken to the catalog screen, with a list of items. The click on several items, and as they do so, the items are marked as "added". The user clicks on a button and gets taken to the cart view. They see the items there. They go back to the catalog, and the items they bought still show "added". End of animation." class='site-image-right' %}

설명을 위해 다음과 같은 간단한 앱을 고려해 보십시오.

앱은 3개의 화면이 있습니다. 로그인, 상품, 장바구니 (각각 `MyLoginScreen`과 `MyCatalog`, `MyCart`로 표시합니다.)
쇼핑 앱이 될 수도 있지만, 같은 구조로 간단한 소셜 네트워킹 앱을 생각할수도 있습니다. (상품 대신 담벼락, 장바구니 대신 즐겨찾기로 바꿔보세요)

상품 화면은 `MyAppBar`라는 커스텀 AppBar와, 많은 `MyListItems` 항목이 스크롤되는 목록을 포함합니다.

여기에 앱의 위젯 트리를 시각화 하였습니다.

{% asset development/data-and-backend/state-mgmt/simple-widget-tree alt="A widget tree with MyApp at the top, and MyLoginScreen, MyCatalog and MyCart below it. MyLoginScreen and MyCart area leaf nodes, but MyCatalog have two children: MyAppBar and a list of MyListItems." %}

{% comment %}
  Source drawing for the png above: https://docs.google.com/drawings/d/1KXxAl_Ctxc-avhR4uE58BXBM6Tyhy0pQMCsSMFHVL_0/edit?zx=y4m1lzbhsrvx
{% endcomment %}

최소 6개의 `Widget`의 서브클래스가 있습니다. 대부분은 다른곳의 상태를 접근해야 할 것입니다.
예를들어, 각각의 `MyListItem`은 장바구니 추가가 가능해야 할것입니다. 장바구니에 이미 들어있는 상품을 표시하고 있는지도 알고 싶을수도 있습니다.

이것은 첫번째 질문을 줍니다. 장바구니의 상태는 어디에 둬야 할까요?


## 상태 위로 올리기

Flutter에서는 해당 상태를 사용하는 위젯보다 상위에 상태를 두어야 합니다. 

왜냐구요? Flutter 와 같은 선언적 프레임워크에서는, UI를 변경하고싶다면, 다시 빌드해야만 합니다.
`MyCart.updateWith(somethingNew)` 와 같은 손쉬운 방법이 없습니다.
다른말로는 외부로부터 위젯의 함수를 호출하는 식으로 명령적 변경을 하기가 어렵습니다.
만약 이걸 되게 한다고 하더라도, 프레임워크가 도와주는 대신 당신이 직접 다투어야 합니다.

<!-- skip -->
```dart
// BAD: 이러지 마세요
void myTapHandler() {
  var cartWidget = somehowGetMyCartWidget();
  cartWidget.updateWith(item);
}
```

만약 위의 코드를 어떻게 동작하게 한다고 하더라도, `MyCart`위젯에서 다음과 같은것을 해결해야 합니다.

<!-- skip -->
```dart
// BAD: 이러지 마세요
Widget build(BuildContext context) {
  return SomeWidget(
    // 장바구니의 초기 상태입니다.
  );
}

void updateWith(Item item) {
  // 여기서 능력껏 UI를 변경해야 합니다.
}
```

새로운 데이터를 적용하기 전에 현재의 UI 상태가 어떤지 고려해야 할것입니다. 
이러한 방법으로는 오류를 피하기 어렵습니다.

Flutter에서는 위젯의 내용이 바뀔때마다 새로운 위젯을 구성합니다.
`MyCart.updateWith(somethingNew)` (함수 호출) 대신 `MyCart(contents)` (constructor) 를 사용합니다.
상위 위젯의 build 메서드를 통해서만 새로운 위젯을 구성할 수 있기 때문에, 
`contents`를 변경하고 싶다면, `MyCart`의 부모 또는 그보다 상위 위젯에 `contents`가 존재해야 합니다.

<?code-excerpt "state_mgmt/simple/lib/src/provider.dart (myTapHandler)"?>
```dart
// GOOD
void myTapHandler(BuildContext context) {
  var cartModel = somehowGetMyCartModel(context);
  cartModel.add(item);
}
```

이제 `MyCart` 는 어떠한 UI 버전을 빌드하더라도 한개의 코드 경로만 가지게 됩니다.

<?code-excerpt "state_mgmt/simple/lib/src/provider.dart (build)"?>
```dart
// GOOD
Widget build(BuildContext context) {
  var cartModel = somehowGetMyCartModel(context);
  return SomeWidget(
    // Just construct the UI once, using the current state of the cart.
    // ···
  );
}
```

In our example, `contents` needs to live in `MyApp`. Whenever it changes,
it rebuilds `MyCart` from above (more on that later). Because of this,
`MyCart` doesn't need to worry about lifecycle&mdash;it just declares
what to show for any given `contents`. When that changes, the old
`MyCart` widget disappears and is completely replaced by the new one.

{% asset development/data-and-backend/state-mgmt/simple-widget-tree-with-cart alt="Same widget tree as above, but now we show a small 'cart' badge next to MyApp, and there are two arrows here. One comes from one of the MyListItems to the 'cart', and another one goes from the 'cart' to the MyCart widget." %}

{% comment %}
  Source drawing for the png above: https://docs.google.com/drawings/d/1ErMyaX4fwfbIW9ABuPAlHELLGMsU6cdxPDFz_elsS9k/edit?zx=j42inp8903pt
{% endcomment %}

This is what we mean when we say that widgets are immutable.
They don't change&mdash;they get replaced.

Now that we know where to put the state of the cart, let's see how
to access it.

## Accessing the state

When user clicks on one of the items in the catalog,
it’s added to the cart. But since the cart lives above `MyListItem`,
how do we do that?

A simple option is to provide a callback that `MyListItem` can call
when it is clicked. Dart's functions are first class objects,
so you can pass them around any way you want. So, inside
`MyCatalog` you can have the following:

<?code-excerpt "state_mgmt/simple/lib/src/passing_callbacks.dart (methods)"?>
```dart
@override
Widget build(BuildContext context) {
  return SomeWidget(
    // Construct the widget, passing it a reference to the method above.
    MyListItem(myTapCallback),
  );
}

void myTapCallback(Item item) {
  print('user tapped on $item');
}
```

This works okay, but for an app state that you need to modify from
many different places, you'd have to pass around a lot of
callbacks&mdash;which gets old pretty quickly.

Fortunately, Flutter has mechanisms for widgets to provide data and
services to their descendants (in other words, not just their children,
but any widgets below them). As you would expect from Flutter,
where _Everything is a Widget™_, these mechanisms are just special
kinds of widgets&mdash;`InheritedWidget`, `InheritedNotifier`,
`InheritedModel`, and more. We won't be covering those here,
because they are a bit low-level for what we're trying to do.

Instead, we are going to use a package that works with the low-level
widgets but is simple to use. It's called `provider`.

With `provider`, you don't need to worry about callbacks or
`InheritedWidgets`. But you do need to understand 3 concepts:

* ChangeNotifier
* ChangeNotifierProvider
* Consumer


## ChangeNotifier

`ChangeNotifier` is a simple class included in the Flutter SDK which provides
change notification to its listeners. In other words, if something is 
a `ChangeNotifier`, you can subscribe to its changes. (It is a form of 
Observable, for those familiar with the term.)

In `provider`, `ChangeNotifier` is one way to encapsulate your application 
state. For very simple apps, you get by with a single `ChangeNotifier`. 
In complex ones, you'll have several models, and therefore several 
`ChangeNotifiers`. (You don't need to use `ChangeNotifier` with `provider`
at all, but it's an easy class to work with.)

In our shopping app example, we want to manage the state of the cart in a
`ChangeNotifier`. We create a new class that extends it, like so:

<?code-excerpt "state_mgmt/simple/lib/src/provider.dart (model)" replace="/ChangeNotifier/[!$&!]/g;/notifyListeners/[!$&!]/g"?>
```dart
class CartModel extends [!ChangeNotifier!] {
  /// Internal, private state of the cart.
  final List<Item> _items = [];

  /// An unmodifiable view of the items in the cart.
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  /// The current total price of all items (assuming all items cost $42).
  int get totalPrice => _items.length * 42;

  /// Adds [item] to cart. This is the only way to modify the cart from outside.
  void add(Item item) {
    _items.add(item);
    // This call tells the widgets that are listening to this model to rebuild.
    [!notifyListeners!]();
  }
}
```

The only code that is specific to `ChangeNotifier` is the call 
to `notifyListeners()`. Call this method any time the model changes in a way 
that might change your app's UI. Everything else in `CartModel` is the 
model itself and its business logic.

`ChangeNotifier` is part of `flutter:foundation` and doesn't depend on 
any higher-level classes in Flutter. It's easily testable (you don't even need
to use [widget testing](/docs/testing#widget-tests) for it). For example,
here's a simple unit test of `CartModel`:

<?code-excerpt "state_mgmt/simple/test/model_test.dart (test)"?>
```dart
test('adding item increases total cost', () {
  final cart = CartModel();
  final startingPrice = cart.totalPrice;
  cart.addListener(() {
    expect(cart.totalPrice, greaterThan(startingPrice));
  });
  cart.add(Item('Dash'));
});
```


## ChangeNotifierProvider

`ChangeNotifierProvider` is the widget that provides an instance of 
a `ChangeNotifier` to its descendants. It comes from the `provider` package.

We already know where to put `ChangeNotifierProvider`: above the widgets that
will need to access it. In the case of `CartModel`, that means somewhere 
above both `MyCart` and `MyCatalog`.

You don't want to place `ChangeNotifierProvider` higher than necessary
(because you don't want to pollute the scope). But in our case,
the only widget that is on top of both `MyCart` and `MyCatalog` is `MyApp`.

<?code-excerpt "state_mgmt/simple/lib/main.dart (main)" replace="/ChangeNotifierProvider/[!$&!]/g"?>
```dart
void main() {
  runApp(
    [!ChangeNotifierProvider!](
      builder: (context) => CartModel(),
      child: MyApp(),
    ),
  );
}
```

Note that we're defining a builder which will create a new instance
of `CartModel`. `ChangeNotifierProvider` is smart enough _not_ to rebuild
`CartModel` unless absolutely necessary. It will also automatically call
`dispose()` on `CartModel` when the instance is no longer needed.  

If you want to provide more than one class, you can use `MultiProvider`:

<?code-excerpt "state_mgmt/simple/lib/main.dart (multi-provider-main)" replace="/multiProviderMain/main/g;/MultiProvider/[!$&!]/g"?>
```dart
void main() {
  runApp(
    [!MultiProvider!](
      providers: [
        ChangeNotifierProvider(builder: (context) => CartModel()),
        Provider(builder: (context) => SomeOtherClass()),
      ],
      child: MyApp(),
    ),
  );
}
```

## Consumer

Now that `CartModel` is provided to widgets in our app through the
`ChangeNotifierProvider` declaration at the top, we can start using it.

This is done through the `Consumer` widget.

<?code-excerpt "state_mgmt/simple/lib/src/provider.dart (descendant)" replace="/Consumer/[!$&!]/g"?>
```dart
return [!Consumer!]<CartModel>(
  builder: (context, cart, child) {
    return Text("Total price: ${cart.totalPrice}");
  },
);
```

We must specify the type of the model that we want to access.
In this case, we want `CartModel`, so we write
`Consumer<CartModel>`. If you don't specify the generic (`<CartModel>`),
the `provider` package won't be able to help you. `provider` is based on types,
and without the type, it doesn't know what you want.

The only required argument of the `Consumer` widget
is the builder. Builder is a function that is called whenever the
`ChangeNotifier` changes. (In other words, when you call `notifyListeners()`
in your model, all the builder methods of all the corresponding
`Consumer` widgets are called.)

The builder is called with three arguments. The first one is `context`,
which you also get in every build method.

The second argument of the builder function is the instance of 
the `ChangeNotifier`. It's what we were asking for in the first place. You can
use the data in the model to define what the UI should look like 
at any given point.

The third argument is `child`, which is there for optimization.
If you have a large widget subtree under your `Consumer`
that _doesn't_ change when the model changes, you can construct it
once and get it through the builder.

<?code-excerpt "state_mgmt/simple/lib/src/performance.dart (child)" replace="/\bchild\b/[!$&!]/g"?>
```dart
return Consumer<CartModel>(
  builder: (context, cart, [!child!]) => Stack(
        children: [
          // Use SomeExpensiveWidget here, without rebuilding every time.
          [!child!],
          Text("Total price: ${cart.totalPrice}"),
        ],
      ),
  // Build the expensive widget here.
  [!child!]: SomeExpensiveWidget(),
);
```

It is best practice to put your `Consumer` widgets as deep in the tree
as possible. You don't want to rebuild large portions of the UI
just because some detail somewhere changed.

<?code-excerpt "state_mgmt/simple/lib/src/performance.dart (nonLeafDescendant)"?>
```dart
// DON'T DO THIS
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return HumongousWidget(
      // ...
      child: AnotherMonstrousWidget(
        // ...
        child: Text('Total price: ${cart.totalPrice}'),
      ),
    );
  },
);
```

Instead:

<?code-excerpt "state_mgmt/simple/lib/src/performance.dart (leafDescendant)"?>
```dart
// DO THIS
return HumongousWidget(
  // ...
  child: AnotherMonstrousWidget(
    // ...
    child: Consumer<CartModel>(
      builder: (context, cart, child) {
        return Text('Total price: ${cart.totalPrice}');
      },
    ),
  ),
);
```

### Provider.of

Sometimes, you don't really need the _data_ in the model to change the
UI but you still need to access it. For example, a `ClearCart`
button wants to allow the user to remove everything from the cart.
It doesn't need to display the contents of the cart,
it just needs to call the `clear()` method.

We could use `Consumer<CartModel>` for this,
but that would be wasteful. We'd be asking the framework to
rebuild a widget that doesn't need to be rebuilt.

For this use case, we can use `Provider.of`, with the `listen` parameter
set to `false`. 

<?code-excerpt "state_mgmt/simple/lib/src/performance.dart (nonRebuilding)" replace="/listen: false/[!$&!]/g"?>
```dart
Provider.of<CartModel>(context, [!listen: false!]).add(item);
```

Using the above line in a build method will not cause this widget to
rebuild when `notifyListeners` is called.


## Putting it all together

You can [check out the
example]({{site.github}}/flutter/samples/tree/master/provider_shopper)
covered in this article. If you want something simpler,
you can see how the simple Counter app looks like when [built with
`provider`](https://github.com/flutter/samples/tree/master/provider_counter).

When you're ready to play around with `provider` yourself,
don't forget to add the dependency on it to your `pubspec.yaml` first.

```yaml
name: my_name
description: Blah blah blah.

# ...

dependencies:
  flutter:
    sdk: flutter

  provider: ^3.0.0

dev_dependencies:
  # ...
```

이제 `import 'package:provider/provider.dart';` 를 하고 개발을 시작해보세요.
