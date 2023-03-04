---
layout: default
title: layout - single child
parent: Flutter Widget Index
nav_order: 7
---

<br>

- [Container](https://api.flutter.dev/flutter/widgets/Container-class.html)
  - child
  - margin : container 의 outside 방향으로 여백을 얼마나 둘지
  - padding : child 의 outside 방향과 container 의 inside 방향의 크기(결국 child 의 충전재 공간). 즉, padding 클수록 child 가 짜부됨.
  - decoration
  - alignment : Align the child within the container.

<br>

- [Align](https://api.flutter.dev/flutter/widgets/Align-class.html)
  A widget that aligns its child within itself and optionally sizes itself based on the child's size.
  
<br>

- [AspectRatio](https://api.flutter.dev/flutter/widgets/AspectRatio-class.html)
  A widget that attempts to size the child to a specific aspect ratio.

<br>

- [Baseline](https://api.flutter.dev/flutter/widgets/Baseline-class.html)

<br>

- [Center](https://api.flutter.dev/flutter/widgets/Center-class.html)
  Center 자체는 제약이 허락하는 한 최대한의 크기를 차지하고, child 를 중앙에 위치시킨다.
  
<br>

- [ConstrainedBox](https://api.flutter.dev/flutter/widgets/ConstrainedBox-class.html)
  A widget that imposes additional constraints on its child.

<br>

- [Expanded](https://api.flutter.dev/flutter/widgets/Expanded-class.html)
  A widget that expands a child of a Row, Column, or Flex so that the child fills the available space.
  설명대로 만약에 Row, Column, Flex 의 child 로 사용하지 않을 경우 'Incorrect use of ParentDataWidget.' 라는 에러가 뜬다.
  꼭 정해진 제약조건을 걸어줄 widget 에 먼저 크기를 잡아주고, 그 외에 유연하게 마음대로 남은 공간을 알아서 차지하도록 둘 child 를 Expanded 로 감싸는 식으로 사용하는 방법이 일반적이다.

<br>


