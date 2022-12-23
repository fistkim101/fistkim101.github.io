---
layout: default
title: CH 3 플러터의 세계로
parent: Flutter In Action
nav_order: 3
---

- 플러터가 위젯을 그리는 원리
- 세 tree 간 참조 관계
- element 가 상태 객체를 관리한다

아래 영상과 CH3 의 내용의 교집합이 80% 수준이고, 오히려 아래 영상이 책의 텍스트보다 이해하기에 좋아서 아래 영상을 학습 및 정리한다.
이후 책에서 정리할 필요성이 있는 내용만 골라 정리한다.

{% include youtube.html id="996ZgFRENMs" %}

<br>

## 플러터가 위젯을 그리는 원리

![](/images/how-flutter-render-widget-01.png)

{: .point }
A widget is an immutable description of part of a user interface.
widget describes the configuration for an element. 

widget은 불변이다. widget 이 그림과 같이 교체될 수는 있지만 노드(위젯) 단위로 보면 불변이다.
widget은 element 에 대한 configuration 이다.

<br>

{: .point }
Element: an instantiation of a Widget at a particular location in the tree.  

{: .point }
RenderObject: handles size, layout, painting.

flutter가 UI 를 렌더링 하는 것은 widget을 바라보는게 아니라 RenderObject 를 바라보는 것이다.

<br>

![](/images/how-flutter-render-widget-02.png)
간단한 샘플로 어떻게 flutter가 최초로 렌더링을 하는지 그 과정을 살펴본다. 세 tree 를 어떤 순서로 어떤 과정에 의해서 만들게 되는지 보는게 포인트다.
<br>
<br>

![](/images/how-flutter-render-widget-03.png)
먼저 widget tree 를 완성한다.
<br>
<br>

![](/images/how-flutter-render-widget-04.png)
![](/images/how-flutter-render-widget-05.png)
그리고 element tree 를 widget 을 configuration 삼아 만든다.
<br>
<br>

![](/images/how-flutter-render-widget-06.png)
![](/images/how-flutter-render-widget-07.png)
renderObject 를 만든다.
<br>
<br>

여기서 widget 이 바뀌면 어떻게 되는지 살펴본다.

![](/images/how-flutter-render-widget-08.png)
위와 같은 케이스때 tree 들의 참조관계가 어떻게 변하는지 살펴보자.
<br>
<br>

![](/images/how-flutter-render-widget-09.png)
위에서 쭉 설명한 순서로 세 tree가 만들어지고 두 번째 runApp 이 실행된 단계다.
<br>
<br>

![](/images/how-flutter-render-widget-10.png)
flutter는 이 때 위 로직에 의해서 element update 를 판단한다.
같은 element 를 사용할지에 관해서 판단하게 된다.
<br>
<br>

![](/images/how-flutter-render-widget-11.png)
다음으로 renderObject 를 업데이트 하게 된다.
<br>
<br>

![](/images/how-flutter-render-widget-12.png)
최종적으로 위와 같이 render가 된다.

<br>
<br>

책에 따르면 렌더 객체는 '복잡하고 비싸서' 플러터 내부적으로 비싼 렌더 객체를 재사용하면서 동시에 비싸지 않은 위젯을 마음껏 파괴해서 결과적으로 성능 최적화를 이뤄낸다.

<br>

## 세 tree 간 참조 관계

![](/images/flutter-three-tree.png)

element 는 widget 을 참조하고, element 가 renderObject 를 만든다. renderObject 는 dart:ui 를 이용해서 화면을 그린다.

## element 가 상태 객체를 관리한다
![](/images/flutter-element-manage-state.png)
말 그대로 element가 상태 객체를 관리한다.

그래서 위 그림에서 widget tree 내 A, B의 위치를 바꿔도 element tree 의 참조는 변하지 않는다.
참조를 갱신하는 기준은 '런타임의 형식', '위젯의 키' 인데 둘이 변하지 않았다면 참조를 갱신하지 않는다.
![](/images/how-flutter-render-widget-10.png)

runTimeType 이 클래스라고 (지금으로서는) 확인이 된다.
이 예제에서 아래와 같이 FancyButtonSecond 로 클래스를 구분해주니 의도대로 색깔이 같이 바뀌는 것을 확인할 수 있었다.
```dart
    final incrementButton = FancyButton(
      child: Text(
        "Increment",
        style: TextStyle(color: Colors.white),
      ),
      onPressed: _incrementCounter,
    );

    final decrementButton = FancyButtonSecond(
      child: Text(
        "Decrement",
        style: TextStyle(color: Colors.white),
      ),
      onPressed: _decrementCounter,
    );
```
