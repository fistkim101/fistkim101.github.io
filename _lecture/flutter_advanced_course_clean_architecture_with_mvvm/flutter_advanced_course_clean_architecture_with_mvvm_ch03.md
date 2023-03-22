---
layout: default
title: CH 3 Presentation Layer - Splash Screen
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 3
---

## android - app name, splash 진입 전 화면 color 변경, Launcher icon 변경
앱 진입시 splash 전에 나오는 화면 색상은 android/app/src/main/res/drawable 에 정의되어 있다. 강의에서는 android/app/src/main/res/values/color.xml 에 색상을 정의하고
이를 android/app/src/main/res/drawable 에서 가져다 사용해서 처리했다.

app name 의 경우 android/app/src/main/AndroidManifest.xml 에 정의되어 있다. 강의에서는 android/app/src/main/res/values/strings.xml 에 app name 을 정의하고
이를 android/app/src/main/AndroidManifest.xml 에서 사용해서 해결했다.

흰색 나오는게 무조건 나쁜 것 같진 않다. 내가 사용하는 몇몇 앱은 흰색을 그대로 사용하고 있긴 하다. 이건 사용자 경험에 엄청나게 크리티컬한 것 같진 않으니,
가벼운 마음으로 나중에 결정해서 처리를 하던가 하자.

Launcher icon 변경의 경우 강의에서 [AndroidAssetStudio](https://romannurik.github.io/AndroidAssetStudio/) 이라는 사이트를 이용했다.
구체적인 방법은 나중에 강의를 다시 보자. 안젤라 강의에서도 이미 다뤘었기에 혹시 막히면 같이 참고하자.

<br>

## iOS - app name, splash 진입 전 화면 color 변경, Launcher icon 변경
app name 변경시 ios/Runner/Info.plist 에서 CFBundleName 를 변경해줬는데, 안젤라 강의에서는 이런 방식으로 안했던 것 같기도 하다. 나중에 크로스체크 하자.

splash 진입 전 화면 color 변경은 ios/Runner/Base.lproj/LaunchScreen.storyboard 내에 아래 속성을 변경해줬다.
```xml
<color key="backgroundColor" red="0.92941176470588238" green="0.59215686274509804" blue="0.15686274509803921" alpha="1" colorSpace="custom" customColorSpace="sRGB"/>
```

[Launcher icon 변경](https://www.appicon.co/) 은 이 사이트를 이용했다. 안젤라 강의에서 이미 자세히 다뤘었다.

위 절차는 모두 iOS 변경을 Android Studio 에서 하고 싶을때 하는 것인데, xcode 에서 처리하는 것이 더 직관적이고 편하다.
이 강의에서도 xcode 에서 변경하는 방식을 잘 보여주고 있고, 안젤라 강의에서도 자세히 다루고 있으니 나중에 적용시 xcode 에서 편하게 바꾸도록 하자.
