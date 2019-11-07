---
title: 선언적으로 생각하기
prev:
  title: 시작
  path: /docs/development/data-and-backend/state-mgmt
next:
  title: Ephemeral vs. app state
  path: /docs/development/data-and-backend/state-mgmt/ephemeral-vs-app
---

Android SDK나 iOS UIKit 과 같은 명령형 프레임워크에서 Flutter로 왔다면, 앱 개발에 대해서 새로운 관점으로 생각해야 합니다.

당신이 해보는 추정이 Flutter 에는 적용되지 않을 수도 있습니다. 예를 들면, Flutter에서는 UI의 일부분만을 변경하는 대신  처음부터 빌드 하는것이 허용됩니다. 
Flutter는 그럴 수 있을정도로 충분히 빠릅니다, 필요하다면 매 프레임마다도 가능합니다. 

Flutter 는 _선언적_입니다. 이는 Flutter가 UI를 현재 state를 반영하는 상태로 빌드한다는 것을 의미합니다:

{% asset development/data-and-backend/state-mgmt/ui-equals-function-of-state alt="UI = f(state) 를 표현하는 수학식. 'UI' 는 화면 구성입니다. 'f'는 빌드 함수입니다. 'state'는 어플리케이션의 상태입니다." %}

{% comment %}
위 png의 원본 그림: https://docs.google.com/drawings/d/1RDcR5LyFtzhpmiT5-UupXBeos2Ban5cUTU0-JujS3Os/edit?usp=sharing
{% endcomment %}

당신 앱의 state 가 변경될때 (예를들어 사용자가 설정 화면에서 스위치를 조작할때), 당신은 state 를 변경하고, 그것이 사용자 인터페이스를 다시 그리도록 트리거 합니다.
UI 자체헤는 (`widget.setText` 같은) 명령적 변화가 없습니다. — state를 바꾸면, UI는 처음부터 다시 빌드됩니다. 

[시작하기 가이드](/docs/get-started/flutter-for/declarative)에서 UI 프로그래밍으로의 선언적 접근에 대해 더 읽어보세요. 

UI 프로그래밍의 선언적 스타일은 많은 장점을 가지고 있습니다. 놀랍게도 UI의 모든 상태로는 단 한개의 코드 경로만 존재합니다. 
UI가 주어진 상태에서 어떻게 생겨야 할지 한번만 설명을 하면, 그것이 전부입니다.

처음에는 이 방식의 프로그래밍이 필수적이지 않아 보일수 있습니다. 그래서 이 섹션이 있습니다. 계속 읽어주세요.
