---
layout: default
title: CH 22 Presentation Layer - Localisation
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 22
---

<br>

다국어 설정을 했는데 강의에서는 easy_localization 과 flutter_phoenix 를 이용해서 처리했다.

[공식문서](https://docs.flutter.dev/development/accessibility-and-localization/internationalization) 에서 다루고 있는 방식은 아니었다.

공식문서에서 다루고 있는 방식을 실습한 [유투브](https://www.youtube.com/watch?v=WrqH5fF2ZuY) 와 공식문서를 같이 보고 처리하니까 쉽게 되었다.
이건 한국인 블로거가 플러터 다국어 처리에 대해서 정리해둔 [포스팅](https://jay-flow.medium.com/flutter-localizations-%EC%99%84%EC%A0%84-%EC%A0%95%EB%B3%B5-%ED%95%98%EA%B8%B0-8fa5f50a3fd2) 이다.

[intl 플러그인](https://plugins.jetbrains.com/plugin/13666-flutter-intl)을 써서 하는 것이 더 편리한 것 같아서 실제 프로젝트에서는 플러그인 적용해서 구현하는 것이 좋겠다.

내가 [공식문서를 보고 따라한 샘플 코드](https://github.com/fistkim101/flutter_i18n_sample)를 첨부한다.
