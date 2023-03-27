---
layout: default
title: CH 6 Data / Domain Layers - Clean Architecture Design Pattern
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 6
---

## 각 Layer 역할 및 패키지로 확인

![](/images/concept_layers_01.png)
![](/images/concept_layers_02.png)

기존에 다른 레퍼런스에서 알아봤던 자료들과 비교했을때 Application Layer 부분이 조금 다르다. 일단 강의에서는 저렇게 사용한다.
강의를 모두 끝내면 다 들어가보지 않아도 모두 파악하고 있어야할 내용이다.

## Presentation Layer, Domain Layer, Data Layer 구현과 각 Layer 구성

각 Layer 가 실제로 어떻게 구현되는지에 대해서 학습했다.
[소스코드](https://github.com/fistkim101/flutter-clean-architecture-study) 를 첨부한다.

학습한 것을 기반으로 아래 그림을 만들었다. 각 구성과 세부 항목을 내재화 하자.
참고로 강의에서 규정한 Application Layer 는 아래 그림에서 생략했다.

Application Layer 는 다른 Layer 들과 상호작용 한다기 보다는 어플리케이션 전역에 적용되는 자원에 대한 Layer 이다.
백엔드에서 사용했던 AppConfig 패키지와 쓰임과 역할이 완전히 일치한다.

![](/images/concept_3layer.png)

여기서 UI Widget 과 State Management 사이의 상호 작용이 곧 view 와 viewModel 간의 상호작용이며 복습을 위해 아래 그림을 다시 보자.

![](/images/concept_view_viewmodel.png)

구현 하다가 잠깐씩 막히더라도 학습했던 것을 보고 구현을 하면 그만이다. 달달 외우는게 중요한 것이 아니고
저렇게 계층을 나누는 이유에 대해서 알고, 계층을 나누는 과정에서 어떤 방식으로 처리했는지와 그 과정에서
어떤 라이브러리를 사용했는지 등에 대해서 인지하자.

강의에서는 에러 코드에 대해서 더 잘 처리해 두었는데, 나의 프로젝트에서는 다르게 처리할 가능성이 커서 실습에서는 생략했다.
