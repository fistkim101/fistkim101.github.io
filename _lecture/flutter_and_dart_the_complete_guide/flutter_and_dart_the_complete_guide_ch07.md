---
layout: default
title: CH 7 Navigation & Multiple Screens
parent: Flutter & Dart - The Complete Guide
nav_order: 8
---

- 따로 screen 이란 개념은 없다. 결국 widget 을 screen 으로 사용하는 것.
- 하지만 헷갈리지 않고 구분에 대한 인지를 직관적으로 하기 위해서 screen 의 경우 -screen 이런 식으로 네이밍 룰을 세우고 따르자.
- screen 이동시 뒤로가지 못하게 할 것이면 반드시 이동하면서 현재 페이지를 스택에서 없애도록 신경쓴다.
- screen 이동시 페이지가 스택으로 쌓이는 것을 꼼꼼하게 관리하지 못하면 메모리에 문제가 생길 수 있다.

![](/images/concept-page-stack.png)

