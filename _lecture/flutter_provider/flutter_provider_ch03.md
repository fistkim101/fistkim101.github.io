---
layout: default
title: CH 3 TODO App(실습)
parent: Flutter Provider Essential
nav_order: 4
---

<br>

# TODO App 개요

![](/images/TODO+App+Overview-page-001.jpg)
![](/images/TODO+App+Overview-page-002.jpg)

- TODO App 을 위 슬라이드에 나온 세 가지 방식으로 각각 총 세 번 구현한다.
- 하나의 폴더 내에 세 앱을 각각 만들고, 소스는 폴더 최상단에서 git init 해서 관리해야겠다.

<br>

![](/images/TODO+App+Overview-page-003.jpg)

- Independent State 의 예시로는 TODO Item 의 '완료여부' 를 들 수 있다.
  - 불변하는 값이 아니고 완료가 될 경우 값이 변하는 성질이 있고, 이 변경의 여부에 따라 Widget 의 rebuild 가 필요하기 때문에 ChangeNotifierProvider 를 사용해야한다.
- Computed State 는 다른 것(것들) 에 의존된 Computed 된 값이다. 예시로는 '미완료 Item 수' 를 들 수 있다.

<br>

<hr>

![](/images/TODO+App+Overview-page-004.jpg)

{: .point }
<b>*이거 되게 중요한 슬라이드다.</b><br>
<b>*실습 하면서 위 슬라이드 원칙들을 준수하며 살을 붙여나가고 내면화 시키도록 한다.</b>
<b>*TODO App 실습 다 끝내고 위 원칙들과 내가 실습하면서 느낀 것들 종합해서 원칙 다시 정리한다.</b>

<hr>

<br>

# App structure

![](/images/TODO+App+Structure-page-001.jpg)
