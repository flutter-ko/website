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

우리의 예제에서 `contents`는 `MyApp` 안에  있어야 합니다. 값이 변경될때마다 `MyCart`를 위에서부터 다시 빌드합니다. 
이때문에 `MyCart`는 라이프사이클에 대하여 신경쓰지 않아도 되고, `contents`에서 무엇을 보여줄지만 정의하면 됩니다. 
값이 변경되면, 이전의 `MyCart`위젯은 사라지고 새로운 것으로 교체됩니다.

{% asset development/data-and-backend/state-mgmt/simple-widget-tree-with-cart alt="Same widget tree as above, but now we show a small 'cart' badge next to MyApp, and there are two arrows here. One comes from one of the MyListItems to the 'cart', and another one goes from the 'cart' to the MyCart widget." %}

{% comment %}
  Source drawing for the png above: https://docs.google.com/drawings/d/1ErMyaX4fwfbIW9ABuPAlHELLGMsU6cdxPDFz_elsS9k/edit?zx=j42inp8903pt
{% endcomment %}

이것은 위젯이 변경 불가하다고 말하는 것을 의미합니다. 위젯은 변경되지 않고, 새로운 위젯으로 교체 됩니다.

이제 쇼핑카트의 상태를 어디에 둘지 정했으니, 어떻게 접근할지를 알아봅니다.

## state 에 접근하기

사용자가 상품 카탈로그에서 하나의 아이템을 클릭하면, 쇼핑카트에 추가됩니다. 
하지만 쇼핑카트는 `MyListItem`보다 상위에 있습니다. 어떻게 해결 할까요?

간단한 옵션은 `MyListItem`이 클릭되었을때 호출될수 있는 콜백을 제공하는 것입니다,
다트의 함수는 1급 객체로서, 원하는대로 전달할 수 있습니다. 
그러므로 `MyCatalog` 안에 다음과 같은것을 넣을수 있습니다:

<?code-excerpt "state_mgmt/simple/lib/src/passing_callbacks.dart (methods)"?>
```dart
@override
Widget build(BuildContext context) {
  return SomeWidget(
    // 위젯을 구성하며 위에서 언급한 함수를 전달합니다.
    MyListItem(myTapCallback),
  );
}

void myTapCallback(Item item) {
  print('user tapped on $item');
}
```

이것은 잘 동작하기는 합니다. 하지만 앱 상태의 입장에서 보면 많은 곳에서 변경되어야 하고, 수많은 콜백을 전달해야 해서 금방 늙을수 있습니다. (주의. 콜백 지옥)

다행히도 Flutter 에는 위젯에 속한 하위 위젯(바로 붙어있는 자식 위젯 뿐만 아닌 하위 트리에 속한 모든 위젯)으로 데이터와 서비스를 제공할수 있는 방법이 
있습니다. _Everything is a Widget™_인 Flutter 에서 기대할수 있듯이 이와 같은 매커니즘은 `InheritedWidget`, `InheritedNotifier`, 
`InheritedModel` 과 같은 특별한 종류의 위젯입니다. 이 위젯들은 지금 하려는것에 비해 약간 low-level 이기 때문에 여기서 설명하지는 않습니다. 

대신에, 우리는 위의 low-level 위젯들과 동작하지만 좀더 손쉽게 사용할수 있는 패키지를 사용할 것입니다. `provider`입니다.

`provider`를 이용해서, 당신은 콜백이나 `InheritedWidgets`에 대해 걱정할 필요가 없습니다. 하지만 다음 세가지 개념을 이해해야 합니다.

* ChangeNotifier
* ChangeNotifierProvider
* Consumer


## ChangeNotifier

`ChangeNotifier`는 Flutter SDK 에 포함된 간단한 클래스로 listener 들에게 change notification 을 제공합니다.
다른말로 `ChangeNotifier`인 것이 있다면, 이것의 변경사항에 대해 구독할수 있습니다. 용어에 익숙한 분에게는 이것은 Observable의 한 형태입니다. 

`provider`에서 `ChangeNotifier`는 어플리케이션 상태를 캡슐화 하는 방법중 하나입니다. 
매우 간단한 앱이라면 한개의 `ChangeNotifier`를 사용할수 있습니다.
복잡한 앱에서는 여러개의 모델을 가지고 있을것이므로 여러개의 `ChangeNotifier`가 있을것입니다. 
(`ChangeNotifier`를 `provider`와 같이 쓰지 않아도 되지만, 같이 쓰기에 쉬운 클래스입니다.)

우리의 쇼핑 앱을 예로 들어보면, 쇼핑카트의 상태를 `ChangeNotifier`안에서 관리하고자 합니다. 
아래와 같이 `ChangeNotifier`를 확장하는 새로운 클래스를 작성합니다.

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

`ChangeNotifier`에 특정한 유일한 코드는 `notifyListeners()`를 호출하는 것입니다. 이 함수를 앱의 UI를 변경할 수 있는 모델 변경이 생길때마다 
호출하십시오. `CartModel`에 있는 나머지는 모두 해당 모델과 그 비즈니스 로직입니다.

`ChangeNotifier`는 `flutter:foundation`의 한 부분이고 Flutter의 다른 고차 클래스에 의존적이지 않습니다. 
쉽게 테스트할수 있습니다. ([위젯 테스트](/docs/testing#widget-tests)조차 사용할 필요가 없습니다).
예를 들어 아래의 `CartModel`의 간단한 유닛테스트를 보세요.

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

`ChangeNotifierProvider`는 `ChangeNotifier`를 하위 위젯에 제공하는 위젯입니다. 이 위젯은 `provider` 패키지와 같이 공급됩니다.

`ChangeNotifierProvider` 를 어디에 둘지는 이미 알고 있습니다. 접근이 필요한 위젯의 상위에 두면 됩니다.
`CartModel`의 경우, `MyCart`와 `MyCatalog` 모두의 상위에 있는 어딘가를 의미합니다.

`ChangeNotifierProvider`를 필요보다 상위에 두고 싶지는 않을것입니다. 왜냐하면 Scope 를 오염시키고 싶지는 않을것이니까요.
하지만 우리의 경우는 `MyCart`와 `MyCatalog` 상위의 유일한 위젯은 `MyApp`입니다.

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

`CartModel`의 새 인스턴스를 만드는 빌더를 선언하고 있는것을 참고하십시오. `ChangeNotifierProvider`는
꼭 필요한 경우를 제외하고 `CartModel` 을 _다시 빌드하지 않을_ 정도로 똑똑합니다. 또한 인스턴스가 더이상 필요 없을
 때에는 `CartModel`의 `dispose()` 를 자동으로 호출할 것입니다.

만약 한개 이상의 클래스를 제공하고자 하는 경우엔  `MultiProvider`를 사용할 수 있습니다.

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

이제 `CartModel`이 최상단에 선언된 `ChangeNotifierProvider`를 통하여 앱 내부의 위젯에 제공되고 이것을 사용할것입니다.

이것은 `Consumer`위젯을 통해서 이루어집니다.

<?code-excerpt "state_mgmt/simple/lib/src/provider.dart (descendant)" replace="/Consumer/[!$&!]/g"?>
```dart
return [!Consumer!]<CartModel>(
  builder: (context, cart, child) {
    return Text("Total price: ${cart.totalPrice}");
  },
);
```

우리는 우선 접근하고자 하는 모델을 지정해야 합니다. 이 경우 우리는 `CartModel`을 접근하고자 하므로 `Consumer<CartModel>`이라고 작성합니다.
만약 `<CartModel>` 제너릭을 지정하지 않는다면 `provider` 패키지는 당신을 도울수 없습니다.
`provider` 는 형에 기반하며, 형이 지정되어 있지 않다면 무엇을 원하는것인지 알지 못합니다.

`Consumer`에서 요구되는 유일한 인수는 builder 입니다. builder는 `ChangeNotifier` 가 변경될때마다 호출되는 함수입니다.
(다른말로, `notifyListeners()`를 호출하게 되면 해당하는 `Consumer` 위젯들의 모든 빌더 함수가 호출 됩니다.)

builder 는 3개의 인수로 호출됩니다. 첫번째 인수는 `context`로 모든 빌드 함수에서 받게 됩니다.

builder 함수의 두번쨰 인수는 `ChangeNotifier`의 인스턴스입니다. 이것은 우리가 처음에 알아 본 것입니다. 
모델의 데이터를 사용하여 특정 시점에서 UI의 모양이 어떻게 생겨야 할지 정의 할 수 있습니다.

세번째 인수는 최적화를 위하여 존재하는 `child` 입니다. 만약 `Consumer` 밑에 모델이 변경되어도 바뀌지 않는 큰 위젯 서브트리가 있다면, 
한번만 구성하고 빌더를 통해서 가지고 올수 있습니다.

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

`Consumer` 위젯을 가장 깊은 곳에 두는것이 효과적인 구현입니다. 작은 세부사항이 변경되었다고 하여서 UI의 큰 부분을 다시 빌드하고 싶지는 않을 것입니다.

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

대신:

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

때때로 모델의 _데이터_가 UI를 변경하는것을 필요로 하지 않지만 접근해야 할때가 있습니다.
예를들어, `ClearCart` 버튼은 사용자가 쇼핑카트의 모든것을 제거할수 있도록 허용하고자 합니다.
쇼핑카트의 내용을 보여줄 필요는 없고 `clear()` 함수만 호출하면 됩니다.

여기에 `Consumer<CartModel>`를 사용할수도 있지만, 낭비일것입니다. 프레임워크에게 
다시 빌드할 필요가 없는 위젯을 다시 빌드하도록 요구하는 것일것입니다.

이런 경우에 우리는 `listen` 매개 변수를 `false`로 설정하고 `Provider.of`를 사용할수 있습니다.

<?code-excerpt "state_mgmt/simple/lib/src/performance.dart (nonRebuilding)" replace="/listen: false/[!$&!]/g"?>
```dart
Provider.of<CartModel>(context, [!listen: false!]).add(item);
```

위의 라인을 빌드 함수에 사용하면 `notifyListeners` 이 호출되어도 위젯을 다시 그리지 않을것입니다.


## 하나로 만들기

이 문서에서 사용된 [예시]({{site.github}}/flutter/samples/tree/master/provider_shopper) 를 확인할 수 있습니다.
만약 더 간단한 것을 원한다면 [`provider`로 만든 간단한 카운터 앱](https://github.com/flutter/samples/tree/master/provider_counter)
을 보면 얼마나 간단한지 볼 수 있습니다. 

`provider`를 활용할 준비가 되었다면, `pubspec.yaml`에 패키지를 추가하는것을 잊지 마세요.


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
