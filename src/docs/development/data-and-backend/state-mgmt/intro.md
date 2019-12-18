---
title: State management
description: How to structure an app to manage the state of the data flowing through it.
next:
  title: 선언적으로 생각하기
  path: /docs/development/data-and-backend/state-mgmt/declarative
---

_이미 리액티브 앱의 상태 관리가 익숙하다면, 이 섹션을 생략하실 수 있습니다. 또한 [다른 접근 방법](/docs/development/data-and-backend/state-mgmt/options)을 검토해보고 싶을수도 있습니다._

{% asset development/data-and-backend/state-mgmt/state-management-explainer width="100%" alt="간단한 선언형 상태 관리 시스템의 동작을 보여주는 짧은 Animated GIF. 이것은 이 페이지에서 전체적으로 설명됩니다. 이 이미지는 장식입니다." %}

{% comment %} 
Source of the above animation tracked internally as b/122314402 
{% endcomment %}

Flutter를 탐험 하면서, 어플리케이션 전반에 걸쳐서 스크린간 어플리케이션의 상태를 공유해야 할 때가 있습니다.
선택할수 있는 많은 접근 방법이 있고, 많은 경우를 생각해야 합니다. 

다음 페이지들에서 Flutter 앱에서 상태를 다루는 방법의 기초에 대해 배우게 됩니다.
