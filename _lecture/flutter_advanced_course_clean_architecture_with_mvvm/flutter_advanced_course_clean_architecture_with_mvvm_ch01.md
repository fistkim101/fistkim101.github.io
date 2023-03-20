---
layout: default
title: CH 1 Introduction
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 1
---

- Layer 구성(강의 커리큘럼 구성의 기준이 된다) 및 각 Layer 에 대한 이해
- MVVM 에 관한 이해

<br>

## Layer 구성(강의 커리큘럼 구성의 기준이 된다) 및 각 Layer 에 대한 이해 
강의 자료가 일단 좀 통일성이 없어서 아쉽다. 강의 자료 그대로 첨부한다. 심지어 제공도 안해줘서 굳이 스크린샷을 찍어야한다. 

![](/images/flutter_clean_architecture_01_1.png)
![](/images/flutter_clean_architecture_01_2.png)
![](/images/flutter_clean_architecture_01_3.png)
![](/images/flutter_clean_architecture_01_4.png)
![](/images/flutter_clean_architecture_01_5.png)
![](/images/flutter_clean_architecture_01_6.png)
![](/images/flutter_clean_architecture_01_7.png)
![](/images/flutter_clean_architecture_01_8.png)

강의자료만 봐서는 정확히 이해가 가지 않아서 아래 세 가지 포스팅을 참고 했다.

- [Very good layered architecture in Flutter](https://verygood.ventures/blog/very-good-flutter-architecture)
- [Flutter Clean Architecture Series — Part 1 (UPDATED)](https://devmuaz.medium.com/flutter-clean-architecture-series-part-1-d2d4c2e75c47)
- [Flutter x Clean Architecture](https://itnext.io/flutter-clean-architecture-b53ce9e19d5a)

위의 각각의 포스팅에서 다루고 있는 Layer 구분들을 도식화 한 이미지들은 아래와 같다.

<br>

![](/images/flutter_layer_1.png)
![](/images/flutter_layer_2.png)
![](/images/flutter_layer_3.png)
![](/images/flutter_layer_4.png)

포스팅마다 약간씩 사용되는 용어가 다른 부분이 있긴 했는데 공통적으로 결국 같은 이야기를 하고 있다.

일단 개인적으로는 세 포스팅 중 [Flutter x Clean Architecture](https://itnext.io/flutter-clean-architecture-b53ce9e19d5a) 가 가장 보기에는 좋았던 것 같다.
너무 자세히 또는 너무 단순히 묘사하지 않으면서도 정확하게 각 Layer 의 역할과 각 Layer 에 속하는 것들의 역할들을 설명하고 있어서다.

각 개념에 대해서 나의 말로 풀어서 다시 정리를 해보자.


### Layer 정의 
일단 Layer 들의 구분에 대해서 알아보기 전에 Layer 자체에 대한 정의를 해야한다.
포스팅에 이미 정리가 잘 되어 있는데 단순하게 표현하자면 Layer 는 Architecture 를 구성하는 개념이다.
독립되어 있으면서 Architecture 의 구성에 포함되기 위해서 특정한 책임을 지는 개념적인 구분 계층이다.

> <b>Architecture</b><br>
> Let’s start with the basics: what is app architecture?
> App architecture is the logical way we organize our projects and how the various components interact with
> each other to fulfill the business requirements.
> We want to follow standards and make it easy to identify the components that we'll need to develop
> features in our codebase. The way we establish the relationship and interactions between these
> components can reduce or add complexity to our projects, 
> which has a significant impact on the team's productivity.

<br>

> <b>Layers</b><br>
> Now that we know what architecture is, let’s define layers.
> Layers are the components that compose your architecture.
> We can define these by assigning a specific responsibility to them.
> We should keep these layers simple, yet isolated enough to achieve a maintainable codebase.

<br>

### 각 Layer(Presentation Layer, Domain Layer, Data Layer) 탐구 및 Layer 구성 역할 이해
각 구성에 관해서까지 세세하게 도식화가 된 것은 아래 그림이 가장 좋았다.

![](/images/flutter_layer_4.png)

[Flutter x Clean Architecture](https://itnext.io/flutter-clean-architecture-b53ce9e19d5a) 이 포스팅에 나온 그림인데
아쉬운 점은 Layer 자체만 설명했으면 좋겠는데 일부 개념이 Bloc 에 의존적이다.

<br>

## MVVM 에 관한 이해
사실 순서가 VVMM 인데, 왜 MVVM 으로 용어가 정립 되었는지 모르겠다.
[이 포스팅](https://betterprogramming.pub/how-to-use-mvvm-in-flutter-4b28b63da2ca) 을 보고 학습했다.

정말 간단히 정리하자면 View는 말 그대로 View 다. 사용자와 상호작용 하면서 사용자의 action 을 ViewModel 로 전달하는 역할과,
ViewModel 으로부터 응답을 받아 사용자에게 전달하는 것을 책임진다. 여기서 확실히 해야할 것은 View 는 상태관리를 하진 않는다는 것이다.

ViewModel 은 View 에 데이터를 제공하면서 상태관리도 담당한다.

Model 은 data access layer 와 동일한 역할이다.