---
layout: default
title: CH 14 Firebase, Image upload, Push notifications - Building a ChatApp 
parent: Flutter & Dart - The Complete Guide
nav_order: 15
---

- ~screen 내에 Scaffold 를 선언하는데, 그래서 ~screen 이 Scaffold 보다 1 level 상위에 생성된다고 볼 수 있다. 이 말인즉 ~screen 에 선언된 전역 함수 내에서는 context 를 통해서 Scaffold 에 접근할 수 없다. 따라서 snackbar 등 Scaffold.of(context) 로 Scaffold 를 통해서 무엇인가를 호출하려면 Scaffold 아래 level 에 생성된 widget 의 context를 통해서 호출해야 한다.
- firebase 인증과 StreamProvider 와 SplashScreen 을 이용한 인증 및 router 패턴 정리(Flutter Provider Essential 수업에서 다룸)
- push notification 설정 잡는 부분 Android/iOS 각각 필요시 정리하고 각 운영체제별로 특이사항 정리(운영체제별 고유 패키지 용도 및 역할, iOS 는 key 받는 부분 특히)
- firebase_cli 이용한 firebase 와 flutter 연동

<hr>

## firebase_cli 이용한 firebase 와 flutter 연동

### 1. firebase-cli 를 설치해야한다. mac 에서는 brew 로 설치가 가능하다. firebase 에 생성된 프로젝트 정보들을 가져오기 위해서 필요하다.

```bash
brew install firebase-cli
```

### 2. dart 로 global 하게 flutterfire_cli 를 activate 한다. 하면 .zshrc 수정이 필요할 수 있다.

```bash
# Install the CLI if not already done so
dart pub global activate flutterfire_cli

Package flutterfire_cli is currently active at version 0.2.7.
The package flutterfire_cli is already activated at newest available version.
To recompile executables, first run `dart pub global deactivate flutterfire_cli`.
Installed executable flutterfire.
Warning: Pub installs executables into $HOME/.pub-cache/bin, which is not on your path.
You can fix that by adding this to your shell\'s config file (.bashrc, .bash_profile, etc.):

export PATH="$PATH":"$HOME/.pub-cache/bin"

Activated flutterfire_cli 0.2.7.
```

### 3. firebase 를 연결하고자 하는 프로젝트 경로에 들어가서 firebase_core 의존성을 설치한다. firebase 와 app 연결을 책임지는 의존성이다. (firebase_core plugin, which is responsible for connecting your application to Firebase.)

```bash
flutter pub add firebase_core
```

### 4. flutterfire configure 명령어를 통해서 연결 설정을 잡아준다. 기존 프로젝트 연결로 실습하지 않았고 create a new project 로 실습했다.

```bash
# Run the `configure` command, select a Firebase project and platforms
flutterfire configure
```

![](/images/connection-firebase-flutter.png)

### 5. [공식문서](https://firebase.flutter.dev/docs/overview) 에 따라 아래 코드를 main 에 넣어준다.

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(MyApp());
}
```
