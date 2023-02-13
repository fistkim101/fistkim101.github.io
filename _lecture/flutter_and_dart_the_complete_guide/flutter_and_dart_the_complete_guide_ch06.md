---
layout: default
title: CH 6 Widget & Flutter internals - Deep dive
parent: Flutter & Dart - The Complete Guide
nav_order: 7
---

- flutter in action 에서 다뤘던 플러터가 어떻게 위젯을 그리는지에 관한 원리에 대해서 학습 (tree 세 가지와 상호 의존 관계)
- const 가 성능향상 되는 이유는 위젯 트리 싹다 다시 만들때(widget 은 불변이기 때문에) const 위젯은 다시 만들 필요가 없다고 플러터에게 알려주는 것이기 때문
- .dart 파일 내에 모든 위젯은 프라이빗 메소드로 빼서 _build~ 같은 식으로 사용
- Widget Lifecycle (flutter in action 에서 이미 다뤘음)
- App Lifecycle (main.dart 에서 WidgetsBindingObserver 을 mixin 해서 appState를 파라미터로 받을 수 있다.)
- Understanding Context
  - meta information on the widget & its location in the widget tree
- Using Keys
  - With new keys generated constantly, Flutter finds no widgets for the elements that now look for widget type + key. Hence new state objects are created all the time.
  - UniqueKey vs ValueKey 
    - Unlike UniqueKey(), ValueKey() does not (re-)calculate a random value but simply wraps a non-changing identifier provided by you.

### widget tree, element tree 가 서로 연결되는 과정 복습

1. configuration 역할을 하는 widget tree 가 생성
2. widget 별로 이에 대응하는 element 들이 만들어지며 (element tree 생성) widget과 연결
3. state 객체는 각 element 가 관리
4. widget tree 에서 하나의 widget 이 삭제됨
5. widget tree 는 전부 새로 그려짐
6. 새로 그려진 widget tree 에 맞게 element tree 는 기존의 element 들이 각각의 어떤 widget 을 참조해야할지 결정
7. 이때 결정되는 기준 = (class type이 같은지? && key가 같은지?)
8. 참조할 widget을 찾지 못한 element는 비로소 이때 삭제된다. (즉, widget 의 삭제에 이어서 즉시 삭제되지 않고 참조할 widget 을 탐색해본 뒤 없을때 지운다)

![](images/concept-widget-element-tree.png)
![](images/concept-app-lifecycle.png)
![](images/concept-context.png)
![](images/concept-widget-key.png)