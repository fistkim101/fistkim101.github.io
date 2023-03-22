---
layout: default
title: CH 2 Presentation Layer - Resources Manager
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 2
---

## resource 모두 중앙에서 관리하도록 ~manager 방식으로 처리
color, text, route, style, value 등 각 widget 에서 세부적으로 가져다 쓰는 것들을 resource 로 분류하고 color_manager, route_manager 와 같은 식으로
manager 라는 클래스 내에서 모두 관리하도록 처리했다. 결국 여러 widget 에서 공통적으로 사용될 것이고, 또 앱 자체의 통일성을 위한 목적이기도 하다.
  
예를 들어 text 를 다룰때 앱에서 regular style 로 사이즈를 제외한 나머지 속성들을 정의한 것이 있을때, 이는 regular style 로 사용될 모든 곳에서
공통적으로 사용될 것이기에 font_manager 에서 regular style 을 named parameter 로 변하는 인자들만 다르게 받고, 기본값을 할당 해주는 식으로 사용하면
코드의 반복을 줄이고 리팩토링에도 유리 할 수 있다. 만약 regular style 의 폰트가 바뀐다고 한다면 이렇게 중앙에서 관리되지 않는 경우 여러 곳에서 다 바꿔줘야하니까.

예를 든 text 처럼 다른 모든 것들도 같은 맥락에서 중앙 관리 방식의 효용을 확인할 수 있다.

## 기기 해상도에 따른 image 사이즈 커스텀 load
어느 기기든 동일한 사이즈의 이미지를 보여주는 것이 아니라, 기기의 해상도에 따라서 같은 이미지이지만 다른 해상도의 파일로 표현해야하는 상황이 있다.
대표적으로 splash screen 에서 화면 정중앙에 나오는 로고같은 정적 자원이 이에 해당한다.

최근에 사용해본 앱 중 로딩에 사용된 아이콘이 픽셀이 각지게 나와서 디자인부분에서 썩 좋지않다는 생각을 들게 한 앱이 있는데,
이렇게 만들게 아니면 잘 기억해두자.

일단 강의도 결국 공식문서를 잘 따르고 있으니,
[공식문서](https://docs.flutter.dev/development/ui/assets-and-images#loading-images) 참고하자.
