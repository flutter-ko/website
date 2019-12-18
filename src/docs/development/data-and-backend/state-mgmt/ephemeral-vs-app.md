---
title: Differentiate between ephemeral state and app state
prev:
  title: 선언적으로 생각하기
  path: /docs/development/data-and-backend/state-mgmt/declarative
next:
  title: 간단한 앱 상태 관리
  path: /docs/development/data-and-backend/state-mgmt/simple
---

_이 문서는 앱 상태와 임시 상태(ephemeral state), 그것들을 Flutter 앱에서 어떻게 관리할지를 설명합니다.._

가장 넓은 의미에서 앱의 상태는 앱이 실행될때 메모리에 있는 모든 것입니다. 여기에는 앱의 asset, 
UI에 관하여 Flutter 프레임워크가 갖고있는 모든 변수들, 애니매이션 상태, 텍스처, 폰트 등이 있습니다. 
이 상태에 대한 가장 넓은 정의는 맞는 말이지만, 앱을 설계하는데 별로 유용하지 않습니다.

우선, (텍스처 같은) 일부 상태는 당신이 아예 관리하지 않습니다. 프레임워크가 대신 처리해줍니다.
그러므로, state의 좀더 실용적인 정의는 "어느 시점이던 UI를 재구성 하는데 필요한 모든 데이터"라고 할 수 있습니다.
둘째로, _직접 관리_하는 상태는 임시 상태와 앱 상태의 두가지 개념적 유형으로 나눌수 있습니다. 

## 임시 상태(Ephemeral state)

임시 상태(때때로 _UI state_ 나 _local state_ 로 불리는)는 한개의 위젯에 깔끔하게 담겨있는 상태입니다.

이것은 의도적으로 모호한 정의입니다. 여기 몇가지 예가 있습니다.

* [`PageView`][] 에 있는 현제 페이지
* 복잡한 애니메이션의 현재 진행상태
* `BottomNavigationBar`에서 현재 선택된 탭

위젯 트리의 다른 부분은 이러한 종류의 상태에 접근할 필요가 거의 없습니다.
직렬화를 할 필요도 없고, 복잡하게 바뀌지도 않습니다.

다른말로, 이런 경우에 (ScopedModel, Redux 등등 같은) 상태 관리 기술을 사용할 이유가 없습니다. 
필요한것은 `StatefulWidget` 하나입니다. 

아래를 보면, BottomNavigationBar의 현재 선택된 항목이 `_MyHomepageState` 클래스의 `_index` 필드에 담겨있는 것을 볼 수 있습니다.
이 예시에서 `_index`는 임시 상태입니다.

<?code-excerpt "state_mgmt/simple/lib/src/set_state.dart (Ephemeral)" plaster="// ... items ..."?>
```dart
class MyHomepage extends StatefulWidget {
  @override
  _MyHomepageState createState() => _MyHomepageState();
}

class _MyHomepageState extends State<MyHomepage> {
  int _index = 0;

  @override
  Widget build(BuildContext context) {
    return BottomNavigationBar(
      currentIndex: _index,
      onTap: (newIndex) {
        setState(() {
          _index = newIndex;
        });
      },
      // ... items ...
    );
  }
}
```

여기서 `setState()` 와 StatefulWidget의 State 클래스 내의 필드를 사용하는것이 완전히 자연스럽습니다.
앱의 다른 부분에서는 `_index` 를 접근할 필요가 없습니다.
해당 변수는 `MyHomepage` 위젯 내에서만 변경됩니다.
그리고 사용자가 앱을 종료하고 다시 시작한면 `_index`가 0으로 돌아가는 것을 신경 쓰지 않습니다.

## 앱 상태(App state)

임시 상태가 아니며 앱의 여러 부분에서 공유하고 사용자 세션간에 유지하려는 상태를 앱 상태라고 부릅니다.
(때로는 공유 상태라고도 부르기도 합니다.)

앱 상테의 예:

* User preferences
* 로그인 정보
* 소셜 네트워킹 앱의 노티피케이션
* 전자상거래 앱의 쇼핑카트
* 뉴스 앱 내 기사의 읽음/안읽음 상태

앱 상태를 관리하기 위해서, 가능한 옵션들을 조사해봐야 할 것입니다. 앱의 성격과 복잡도, 팀의 기존 경험 
그리고 많은 다른 관점을 기반으로 선택해야 합니다. 계속 읽어보세요.

## 명확한 규칙이 없습니다

명확하게 하자면, 앱의 모든 상태를 관리하기 위해서 `State` 와 `setState()`를 사용 _할 수_ 있습니다.
실은 Flutter 팀은 많은 간단한 앱 샘플에서 이렇게 합니다. (`flutter create`에서 매번 생성되는 스타터 앱도 포함해서요.)

다른 방향으로도 갑니다. 예를들어 당신은 &mdash;특정 앱의 컨텍스트 안에서&mdash; 하단 네비게이션 바의 선택된 탭이 _임시 상태가 아니도록_ 결정할수 있습니다.
클래스의 외부에서 변경해야 할 필요가 있을 수도 있고, 세션간에 유지해야 할 수도, 다른 경우가 있을수도 있습니다.
이런 경우엔 `_index` 변수는 앱 상태입니다.

특정 변수가 임시 상태인지, 앱상태인지를 구분하는 명확하고 보편적인 규칙은 없습니다. 때때로 임시 상태에서 앱 상태로 
또는 그 반대로 리팩토링 해야 할수 있습니다. 예를 들어, 명확한 임시 상태로 시작했지만, 어플리케이션의 기능이 늘어나면서, 
앱 상태로 옮겨야 할 수도 있습니다.

그런 의미에서 아래의 잘못된 그림을 한번 보세요.

{% asset development/data-and-backend/state-mgmt/ephemeral-vs-app-state alt="흐름도. 'Data'로 시작. '누가 필요한가?'. 세가지 옵션: '대부분의 위젯' 과 '일부 위젯', '단일 위젯'. 앞의 두 옵션은 '앱 상태'로 이어짐. '단일 위젯' 옵션은 '임시 상태로' 이어짐." %}

{% comment %}
위 png의 원본 그림: : https://docs.google.com/drawings/d/1p5Bvuagin9DZH8bNrpGfpQQvKwLartYhIvD0WKGa64k/edit?usp=sharing
{% endcomment %}

Redux의 저자 Dan Abramov는 React의 setState와 Redux의 store에 대해 물었을때 이렇게 답했습니다.

> "경험의 법칙은: [Do whatever is less awkward][](덜 어색한걸 하라)."

결론적으로, 모든 Flutter 앱에는 두가지 개념적 유형의 상태가 있습니다.
임시 상태는 `State` 와 `setState()`로 구현될 수 있고, 종종 단일 위젯에 지역적입니다. 나머지는 앱 상태입니다.
두가지 유형 모두 Flutter 앱에서 사용할 수 있으며, 두 유형의 구분은 당신의 선호도 및 앱의 복잡도에 기반합니다.


[`PageView`]: {{site.api}}/flutter/widgets/PageView-class.html
[Do whatever is less awkward]: {{site.github}}/reduxjs/redux/issues/1287#issuecomment-175351978
