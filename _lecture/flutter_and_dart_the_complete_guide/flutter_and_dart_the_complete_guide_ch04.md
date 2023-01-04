---
layout: default
title: CH 4 Widgets, Styling, Adding Logic - Building a Real App (PERSONAL EXPENSES APP) 
parent: Flutter & Dart - The Complete Guide
nav_order: 5
---

- Column, Row 
  - mainAxisAlignment
  - crossAxisAlignment
- SizedBox vs Container
- sort_child_properties_last
- unnecessary_const

### SizedBox vs Container
[스택오버플로우에서도 있는 질문](https://stackoverflow.com/questions/55716322/flutter-sizedbox-vs-container-why-use-one-instead-of-the-other)이고,
[플러터 공식 가이드](https://dart-lang.github.io/linter/lints/sized_box_for_whitespace.html) 에도 있는 내용이다.

> The main advantage seems to be that SizedBox can be const and won't even create a new instance during runtime.

Container가 성능적으로 더 무겁다. 따라서 SizedBox로 충분하면 SizedBox를 사용하기.

### [sort_child_properties_last](https://dart-lang.github.io/linter/lints/sort_child_properties_last.html)
child, children 프로퍼티는 맨 하단으로 내리기. 그게 권장되는 스타일.

### [unnecessary_const](https://dart-lang.github.io/linter/lints/unnecessary_const.html)
const 는 최상단에만 해주면 된다. 쓸데없이 반복하지 말라.

