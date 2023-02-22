---
layout: default
title: basic
parent: Flutter Widget Index
nav_order: 1
---

<br>

- [Scaffold](https://api.flutter.dev/flutter/material/Scaffold-class.html)
  - [drawer](https://api.flutter.dev/flutter/material/Drawer-class.html)
  - appBar
    - leading
    - title
    - actions[]
  - body
  - floatingActionButton

<br>

- [Row](https://api.flutter.dev/flutter/widgets/Row-class.html)
  - mainAxisAlignment
  - crossAxisAlignment
  - mainAxisSize<br>
    The width of the Row is determined by the mainAxisSize property. If the mainAxisSize property is MainAxisSize.max, then the width of the Row is the max width of the incoming constraints. If the mainAxisSize property is MainAxisSize.min, then the width of the Row is the sum of widths of the children (subject to the incoming constraints).
  - Troubleshooting
    - Why does my row have a yellow and black warning stripe?<br>
      ```dart
      Row(
        children: const <Widget>[
          FlutterLogo(),
          Expanded(
            child: Text("Flutter's hot reload helps you quickly and easily experiment, build UIs, add features, and fix bug faster. Experience sub-second reload times, without losing state, on emulators, simulators, and hardware for iOS and Android."),
          ),
          Icon(Icons.sentiment_very_satisfied),
        ],
      )
      ```      
      Now, the row first asks the logo to lay out, and then asks the icon to lay out. The Icon, like the logo, is happy to take on a reasonable size (also 24 pixels, not coincidentally, since both FlutterLogo and Icon honor the ambient IconTheme). This leaves some room left over, and now the row tells the text exactly how wide to be: the exact width of the remaining space. The text, now happy to comply to a reasonable request, wraps the text within that width, and you end up with a paragraph split over several lines.<br>

<br>

- [Column](https://api.flutter.dev/flutter/widgets/Column-class.html)
  - mainAxisAlignment
  - crossAxisAlignment
  - mainAxisSize<br>
    The height of the Column is determined by the mainAxisSize property. If the mainAxisSize property is MainAxisSize.max, then the height of the Column is the max height of the incoming constraints. If the mainAxisSize property is MainAxisSize.min, then the height of the Column is the sum of heights of the children (subject to the incoming constraints).
  - Troubleshooting
    - When the incoming vertical constraints are unbounded
      When a Column has one or more Expanded or Flexible children, and is placed in another Column, or in a ListView, or in some other context that does not provide a maximum height constraint for the Column, you will get an exception at runtime saying that there are children with non-zero flex but the vertical constraints are unbounded.<br>
      조건 1. `Column 이 하나 이상의 Expanded 또는 Flexible 을 갖고 있는 상태`에서 조건 2. `ListView 나 Column 에 속하고 있거나, 혹은 Column 에 maximum height constraint 을 주지 않는 context 에 속하는` 경우에 'there are children with non-zero flex but the vertical constraints are unbounded.' 에러가 난다.  

      The problem, as described in the details that accompany that exception, is that using Flexible or Expanded means that the remaining space after laying out all the other children must be shared equally, but if the incoming vertical constraints are unbounded, there is infinite remaining space.<br>
      Column 내에 Expanded 나 Flexible 이 있으면 '남는 공간(전체 공간 중 나머지 child 들을 그리고 남는)'을 나눠가지라는 뜻인데 여기서 '전체 공간' 자체가 unbounded 이다 보니(왜냐면 해당 Column 이 unbounded 된 Column 의 children 중 하나이기 때문) 결국 '남는 공간' 이 unbounded 니까 에러가 발생한다.

      The key to solving this problem is usually to determine why the Column is receiving unbounded vertical constraints.

      One common reason for this to happen is that the Column has been placed in another Column (without using Expanded or Flexible around the inner nested Column). When a Column lays out its non-flex children (those that have neither Expanded or Flexible around them), it gives them unbounded constraints so that they can determine their own dimensions (passing unbounded constraints usually signals to the child that it should shrink-wrap its contents). The solution in this case is typically to just wrap the inner column in an Expanded to indicate that it should take the remaining space of the outer column, rather than being allowed to take any amount of room it desires.

      Another reason for this message to be displayed is nesting a Column inside a ListView or other vertical scrollable. In that scenario, there really is infinite vertical space (the whole point of a vertical scrolling list is to allow infinite space vertically). In such scenarios, it is usually worth examining why the inner Column should have an Expanded or Flexible child: what size should the inner children really be? The solution in this case is typically to remove the Expanded or Flexible widgets from around the inner children.

      <b>해결책은 아래 유투브에도 잘 나와있지만 inner Column 의 높이를 명시적으로 말해주면 된다. 정해진 수치(MediaQuery 로 계산이 되어도 되니 결국에 정해지는 고정값이면 된다)를 height 로 잡아주거나, Expanded 로 감싸주면 된다.</b> 

{% include youtube.html id="jckqXR5CrPI" %}