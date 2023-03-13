---
layout: default
title: CH 14 Firebase, Image upload, Push notifications - Building a ChatApp 
parent: Flutter & Dart - The Complete Guide
nav_order: 15
---

- ~screen 내에 Scaffold 를 선언하는데, 그래서 ~screen 이 Scaffold 보다 1 level 상위에 생성된다고 볼 수 있다. 이 말인즉 ~screen 에 선언된 전역 함수 내에서는 context 를 통해서 Scaffold 에 접근할 수 없다.
  따라서 snackbar 등 Scaffold.of(context) 로 Scaffold 를 통해서 무엇인가를 호출하려면 Scaffold 아래 level 에 생성된 widget 의 context를 통해서 호출해야 한다.

- firebase 인증과 StreamProvider 와 SplashScreen 을 이용한 인증 및 router 패턴 정리(Flutter Provider Essential 수업에서 다룸)

- push notification 설정 잡는 부분 Android/iOS 각각 필요시 정리하고 각 운영체제별로 특이사항 정리(운영체제별 고유 패키지 용도 및 역할, iOS 는 key 받는 부분 특히)

  