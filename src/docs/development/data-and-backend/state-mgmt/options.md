---
title: List of state management approaches
prev:
  title: Simple app state management
  path: /docs/development/data-and-backend/state-mgmt/simple
---

상태 관리는 복잡한 주제입니다. 만약 당신의 문제가 해결된것 같지 않거나, 아래 페이지들에서 설명한 방법이 Use Case 에 맞지 않느것 같다고 생각된다면, 아마 맞을겁니다.

다음 링크들에서 자세한 내용을 알아보세요. 많은 문서들이 Flutter 커뮤니티에서 기여하였습니다.


## 일반적인 개요

* [Flutter로 반응형 모바일 앱 만들기](https://www.youtube.com/watch?v=RS36gBEp8OI&feature=youtu.be),
  Google I/O 2018 비디오와 
  [첨부 문서]({{site.flutter-medium}}/build-reactive-mobile-apps-in-flutter-companion-article-13950959e381)
* [Flutter 아키텍처 샘플](http://fluttersamples.com/), by Brian Egan

## setState

* [Flutter앱에 상호작용 추가](/docs/development/ui/interactive),
  Flutter 튜토리얼
* [Google Flutter의 기본 상태 관리]({{site.medium}}/@agungsurya/basic-state-management-in-google-flutter-6ee73608f96d),
  by Agung Surya

## InheritedWidget &amp; InheritedModel 

* [InheritedWidget 문서](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html)
* [Managing Flutter Application State With InheritedWidgets]({{site.flutter-medium}}/managing-flutter-application-state-with-inheritedwidgets-1140452befe1),
  by Hans Muller
* [Inheriting Widgets](https://medium.com/@mehmetf_71205/inheriting-widgets-b7ac56dbbeb1),
  by Mehmet Fidanboylu
* [Using Flutter Inherited Widgets
  Effectively](https://ericwindmill.com/articles/inherited_widget/),
  by Eric Windmill
* [Widget - State - Context -
  InheritedWidget](https://www.didierboelens.com/2018/06/widget---state---context---inheritedwidget/),
  by Didier Bolelens

## Provider &amp; Scoped Model

* [Provider package]({{site.pub-pkg}}/provider)
* [Scoped Model package]({{site.pub-pkg}}/scoped_model)
* [Simple State Management]({{site.url}}/docs/development/data-and-backend/state-mgmt/simple),
  the previous page in this section
* [You might not need Redux: The Flutter edition](https://proandroiddev.com/you-might-not-need-redux-the-flutter-edition-9c11eba006d7), by Ryan Edge
* [Managing state with the scoped model pattern in Dart's Flutter
  framework](https://www.youtube.com/watch?v=-MCeWP3rgI0),
  a video by Tensor Programming
* [Flutter: Inherited Widget and Scoped Model Explained,
  part 1](https://www.youtube.com/watch?v=j-27MZwRbFw),
  a video by MTechViral
* [Flutter state management&mdash;scoped
  model](https://www.youtube.com/watch?v=Oql5bU-Uvso)

## Redux

* [Animation Management with Redux and Flutter](https://www.youtube.com/watch?v=9ZkLtr0Fbgk), DartConf 2018 비디오 [미디엄 문서]({{site.flutter-medium}}/animation-management-with-flutter-and-flux-redux-94729e6585fa)
* [Flutter Redux package]({{site.pub-pkg}}/flutter_redux)
* [Flutter 에서 Redux 소개](https://blog.novoda.com/introduction-to-redux-in-flutter/), by Xavi Rigau
* [Flutter + Redux&mdash;쇼핑 리스트 앱 만들기](https://hackernoon.com/flutter-redux-how-to-make-shopping-list-app-1cd315e79b65),
  by Paulina Szklarska on Hackernoon
* [Fluuter 와 Redux 로 TODO 어플리케이션 만들기 (CRUD) &mdash; Part 1](https://www.youtube.com/watch?v=Wj216eSBBWs),
  Tensor Programming 의 비디오
* [Flutter Redux Thunk, an example]({{site.medium}}/flutterpub/flutter-redux-thunk-27c2f2b80a3b),
  by Jack Wong
* [Building a (large) Flutter app with Redux](https://hillelcoren.com/2018/06/01/building-a-large-flutter-app-with-redux/),
  by Hillel Coren
* [Fish-Redux - An assembled flutter application framework based on Redux](https://github.com/alibaba/fish-redux/),
  by Alibaba
* [Async Redux - Redux without boilerplate. Allows for both sync and async reducers]({{site.pub}}/packages/async_redux/),
  by Marcelo Glasberg

## BLoC / Rx

* [BLoC 패턴을 이용하여 Flutter 프로젝트 설계하기]({{site.medium}}/flutterpub/architecting-your-flutter-project-bd04e144a8f1),
  Sagar Suri
* [Bloc Library](https://felangel.github.io/bloc), by Felix Angelov
* [Reactive Programming - Streams - BLoC - 실용 사례](https://www.didierboelens.com/2018/12/reactive-programming---streams---bloc---practical-use-cases), by Didier Boelens

## MobX

* [MobX.dart, Hassle free state-management for your Dart and Flutter apps](https://github.com/mobxjs/mobx.dart)
* [Getting started with MobX.dart](https://mobx.pub/getting-started)
* [Flutter: State Management with Mobx](https://www.youtube.com/watch?v=p-MUBLOEkCs), a video by Paul Halliday
