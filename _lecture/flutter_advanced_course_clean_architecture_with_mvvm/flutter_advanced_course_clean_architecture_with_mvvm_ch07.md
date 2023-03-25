---
layout: default
title: CH 7 Presentation Layer - Login Flow
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 7
---

<br>

ch05 에서 학습한 Stream 을 이용한 View-ViewModel 상호 작용에 대해서 실제로 Login 화면에 적용하는 챕터였다.
개인적으로 특별히 정리할 만한 내용은 없었다.

좀 특이했던 것은 formField 의 validation 조차도 ViewModel 로 위임하고, 위임에 따라 그 결과 값도 당연히 Stream 으로 받게 한 것이었다.
View 에서는 모든 비즈니스 로직을 빼는 것을 목표로 한다면 맞는 것이긴 하다. 나는 Provider 를 쓸건데, 그럼 나는 Provider 에서 validation 을 처리하는
함수를 만들고 이를 `context.read<ProviderType>().isValid(text)` 와 같은 식으로 처리를 해야할 것 같다.

또 한 가지 특이한 처리는 login() 을 ViewModel 에서 처리하고 이 결과를 View 의 initState 에서 ViewModel 내에서 login 성공 여부에 대한
StreamController 로 직접 참조해서 listen 을 하며 성공시 token 부여를 받아 SharedPreference 에 set 하고는 Navigator.of(context).pushReplacementNamed()로
넘어가도록 처리한 것이다.

즉, login() 성공을 await 로 기다려서 그 뒤에 바로 처리를 하는게 아니라 애초에 initState 시에 login 성공 여부에 대한 StreamController 를 listen 으로 참조해두고
거기서 처음부터 라우터 이동을 처리해둔 것이다. 이러한 처리방식이 새롭고 재미있었다.

아쉬웠던 점은 다른 항목들은 전부 ViewModel 에서 outputStream 을 따로 변수로 만들어서 참조해놓고는, 저건 StreamController 를 직접참조 하는 건 코드 통일성을 깨트리는 행동 같은데
이렇게 처리를 한 점이 좀 이상했다.
