---
layout: default
title: CH 2 Provider Overview
parent: Flutter Provider Essential
nav_order: 3
---

<br>

# section overview
![](/images/section-overview-page-001.jpg)
![](/images/section-overview-page-002.jpg)
![](/images/section-overview-page-003.jpg)

<br>

# necessity of provider
![](/images/necessity-of-provider-page-001.jpg)
![](/images/necessity-of-provider-page-002.jpg)
![](/images/necessity-of-provider-page-003.jpg)
![](/images/necessity-of-provider-page-004.jpg)

- Widget A 에서 counter 라는 데이터와 increment 라는 함수가 필요한 상황이고, Widget B 에서는 counter 라는 데이터만 필요한 상황.
- Widget A, Widget B 모두에서 동일한 데이터인 counter 가 필요하므로 공통된 부모 Widget C 에서 정의.
- counter 와 increment 의 경우 Widget C 에서 선언하지만 정작 제어는 Widget A 에서 하므로 Inversion of control 발생.
- <b>Widget B 에 counter 를 넘겨주기 위해서 Widget C 와 Widget B 사이의 Widget 이 필요하지도 않은 counter 라는 데이터를 갖게 됨.</b>

# managing state without provider

![](/images/managing-state-without-provider-1-page-001.jpg)

- Counter B 만 봐서는 어디서 rebuilding 이 발생하는지 추적하기가 쉽지 않다.
- 공통 상단 부모 Widget 에서 setState() 호출되므로 쓸데없이 더 많은 Widget 이 rebuild 대상이 되어 퍼포먼스가 떨어질 수 있다.

![](/images/managing-state-without-provider-2-page-001.jpg)
![](/images/managing-state-without-provider-2-page-002.jpg)

결국 StateManagement 는 아래 두 가지 행위가 핵심이다. <br>

1. Dependency Injection (Object를 Widget Tree 상에서 쉽게 접근할 수 있도록 한다.)
2. Synchronizing data and UI

![](/images/managing-state-without-provider-2-page-003.jpg)
![](/images/managing-state-without-provider-2-page-004.jpg)

- Provider 는 Widget 에 Widget 이 아닌 데이터와 method 에 쉽게 접근할 수 있는 방법을 제공한다.
- 데이터가 변경 되었을 때 데이터가 변경되었다는 사실을 그 데이터가 필요한 Widget 에 제공해서 필요시 rebuild 될 수 있도록 한다.
- 결국 비즈니스 로직과 화면이 분리 되는 것이다.

<br>

# dependency injection using provider

![](/images/dependency-injection-using-provider-page-001.jpg)

주목해야할 포인트는 아래 두 가지이다.
- Widget Tree 상에서 class Dog 로의 접근이 얼마나 쉽게 가능한지
- 이 방식이 생성자로 데이터를 넘겨주는 것보다 얼마나 더 간편한지

![](/images/dependency-injection-using-provider-page-002.jpg)

- Provider 역시 Widget 이다.
- create 프로퍼티에서 Widget 이 필요로 하는 dog 인스턴스를 만든다.
- create 에 assign 되는 함수가 return 하는 object 에 Provider 하위 Widget 들의 접근이 가능하다.
- Provider 의 static 함수인 of 를 이용하면 Widget Tree 를 위로 traverse 하면서 원하는 Type 의 인스턴스를 찾을 수 있다.
  - `Provider.of<Dog>(context)` 와 같은 식인데, 여기서 context 를 주는 이유는 context 를 이용해서 Widget Tree 를 위로 탐색하기 때문이다. 

![](/images/dependency-injection-using-provider-page-003.jpg)

- `<T>` 가 같은 두 개 이상의 인스턴스가 있는 경우에는 가장 가까운 인스턴스를 가져온다.

```dart
class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Provider<Dog>(
      create: (context) => Dog(
        name: 'Sun',
        breed: 'Bulldog',
        age: 3,
      ),
      child: MaterialApp(
        title: 'Provider 02',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Provider 02'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              '- name: ${Provider.of<Dog>(context).name}',
              style: TextStyle(fontSize: 20.0),
            ),
            SizedBox(height: 10.0),
            BreedAndAge(),
          ],
        ),
      ),
    );
  }
}
```

<br>

# ChangeNotifier & addListener

![](/images/changeNotifier-and-addListener-page-001.jpg)
![](/images/changeNotifier-and-addListener-page-002.jpg)

- addListener 는 자동으로 dispose 되지 않기 때문에 수동으로 직접 dispose 시켜줘야한다.
- 코드 내 Provider 제거 후 ChangeNotifier 와 addListener 를 같이 쓰면서 Listener 가 작동하는 것을 보고, 불편한 점을 살펴보았다.(Provider 삭제 후 생성자로 데이터를 넘겨주었다.)

<br>

# ChangeNotifierProvider

![](/images/ChangeNotifierProvider-page-001.jpg)
![](/images/ChangeNotifierProvider-page-002.jpg)

- `Provider.of<T>(context)` 로 인스턴스를 탐색 & 접근시 해당 데이터의 변경을 listen 해야할지의 필요성(데이터 변경에 따른 UI rebuild)이 있을때와 없을때 각각 옵션값이 다름을 인지한다.
  - `Provider.of<Dog>(context).age`
  - `Provider.of<Dog>(context, listen: false).age`

결론적으로 아래와 같다. 결국 아래 두 가지가 State Management 이다.
- ChangeNotifierProvider 는 데이터를 필요로 하는 Widget 이 dependency injection 을 받음으로써 데이터(인스턴스)에 쉽게 접근할 수 있게 해준다.
- 데이터의 변경이 발생했을 때 이 데이터의 변화에 맞춰서 선택적으로 Widget 을 rebuild 할 수 있게 해준다.

```dart
import 'package:flutter/foundation.dart';

class Dog with ChangeNotifier {
  final String name;
  final String breed;
  int age;
  Dog({
    required this.name,
    required this.breed,
    this.age = 1,
  });

  void grow() {
    age++;
    notifyListeners();
  }
}
```

{: .point }
ChangeNotifier 의 notifyListeners() 는 이를 구독하고 있는 모든 listener 들에게 변경 사실을 알리고 rebuild 하도록 만든다. [공식문서](https://api.flutter.dev/flutter/foundation/ChangeNotifier/notifyListeners.html)를 참고하자.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'models/dog.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<Dog>(
      create: (context) => Dog(name: 'dog04', breed: 'breed04'),
      child: MaterialApp(
        title: 'Provider 04',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Provider 04'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              '- name: ${Provider.of<Dog>(context).name}',
              style: TextStyle(fontSize: 20.0),
            ),
            SizedBox(height: 10.0),
            BreedAndAge(),
          ],
        ),
      ),
    );
  }
}

class BreedAndAge extends StatelessWidget {
  const BreedAndAge({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '- breed: ${Provider.of<Dog>(context).breed}',
          style: TextStyle(fontSize: 20.0),
        ),
        SizedBox(height: 10.0),
        Age(),
      ],
    );
  }
}

class Age extends StatelessWidget {
  const Age({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '- age: ${Provider.of<Dog>(context).age}',
          style: TextStyle(fontSize: 20.0),
        ),
        SizedBox(height: 20.0),
        ElevatedButton(
          onPressed: () => Provider.of<Dog>(context, listen: false).grow(),
          child: Text(
            'Grow',
            style: TextStyle(fontSize: 20.0),
          ),
        ),
      ],
    );
  }
}
```

<br>

# read, watch, select extension methods

![](/images/extension-methods-page-001.jpg)

- 4.1 부터 Provider 를 더 간편하게 쓸 수 있게 해주는 extension 이 도입됨.  
- 일종의 shortCut 이라고 보면 되고, 그냥 쓰지 말고 원형들이 뭔지 이해하고 있는게 중요하다.
- `context.select` 는 다수의 property 를 가지고 있는 object 의 특정 property 의 변화만 listen 하고 싶을 때 사용한다.
  - `context.watch` 는 특정 property 하나만 바뀌어도 rebuild 를 하는 것에 반해 `context.select` 는 listen 하고 싶은 것만 선별적으로 listen 이 가능하다.(퍼포먼스 고려 측면에서 사용할 수 있다.)

{: .point }
<b>context.watch vs context.select</b>

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'models/dog.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<Dog>(
      create: (context) => Dog(name: 'dog05', breed: 'breed05', age: 3),
      child: MaterialApp(
        title: 'Provider 05',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Provider 05'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              '- name: ${context.watch<Dog>().name}',
              style: TextStyle(fontSize: 20.0),
            ),
            SizedBox(height: 10.0),
            BreedAndAge(),
          ],
        ),
      ),
    );
  }
}

class BreedAndAge extends StatelessWidget {
  const BreedAndAge({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '- breed: ${context.select<Dog, String>((Dog dog) => dog.breed)}',
          style: TextStyle(fontSize: 20.0),
        ),
        SizedBox(height: 10.0),
        Age(),
      ],
    );
  }
}

class Age extends StatelessWidget {
  const Age({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '- age: ${context.select<Dog, int>((Dog dog) => dog.age)}',
          style: TextStyle(fontSize: 20.0),
        ),
        SizedBox(height: 20.0),
        ElevatedButton(
          onPressed: () => context.read<Dog>().grow(),
          child: Text(
            'Grow',
            style: TextStyle(fontSize: 20.0),
          ),
        ),
      ],
    );
  }
}
```

<br>

# Multiple Provider

![](/images/MultiProvider-page-001.jpg)

- 하나의 Provider 를 사용하더라도 MultiProvider 를 사용해놓으면 확장성이 있다.

<br>

# Future Provider

![](/images/FutureProvider-page-001.jpg)

- 강사님은 개인적으로 쓸 일이 거의 없었다고 함.
- 만약 쓸 일이 있으면 FutureBuilder 를 쓸 것 같다고 함.
- Widget Tree 에는 빌드가 되었는데 사용하고자 하는 값이 아직 준비가 되지 않았을때 사용한다.
- Future 가 resolve 되지 않았을때, initialData 로 build 하고 Future 가 resolve 되면 rebuild 된다. 총 2번 build 가 된다는 것. (만약 여러번 build 를 원한다면 StreamProvider 사용한다.)

```dart
class Babies {
  final int age;
  Babies({
    required this.age,
  });

  Future<int> getBabies() async {
    await Future.delayed(Duration(seconds: 3));

    if (age > 1 && age < 5) {
      return 4;
    } else if (age <= 1) {
      return 0;
    } else {
      return 2;
    }
  }
}
```
```dart
class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider<Dog>(
          create: (context) => Dog(name: 'dog06', breed: 'breed06', age: 3),
        ),
        FutureProvider<int>(
          initialData: 0,
          create: (context) {
            final int dogAge = context.read<Dog>().age;
            final babies = Babies(age: dogAge);
            return babies.getBabies();
          },
        ),
      ],
      child: MaterialApp(
        title: 'Provider 06',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(),
      ),
    );
  }
}
``` 

- 유심히 봐둬야할 부분은 FutureProvider 의 대상은 Future 이기만 하면 되는 것이라는 것이다. 즉, 인스턴스 내에 정의된 method 를 대상으로 하고 싶다면 인스턴스 전체를 return 할 필요가 없고 특정 메소드만 제공하는 형태로 사용해야 한다.
- FutureProvider 내에서 context.read 사용이 가능하다. 이유는 위에서 ChangeNotifierProvider 가 FutureProvider 보다 상위 Widget 으로 이미 사용되었기 때문이다. 

<br>

# StreamProvider

![](/images/StreamProvider-page-001.jpg)

- 강사님 개인적으로는 FutureProvider 보다 StreamProvider 를 사용할 일이 더 많았다고 함.
- 연속되는 Future value 를 타겟으로 하는 Provider

```dart
        StreamProvider<String>(
          initialData: 'Bark 0 times',
          create: (context) {
            final int dogAge = context.read<Dog>().age;
            final babies = Babies(age: dogAge * 2);
            return babies.bark();
          },
        ),
```

- create 는 한 번만 called 되기 때문에 watch 를 사용하면 에러가 발생한다. 논리적으로도 read 가 맞다.

```dart
class Babies {
  final int age;
  Babies({
    required this.age,
  });

  Future<int> getBabies() async {
    await Future.delayed(Duration(seconds: 3));

    if (age > 1 && age < 5) {
      return 4;
    } else if (age <= 1) {
      return 0;
    } else {
      return 2;
    }
  }

  Stream<String> bark() async* {
    for (int i = 1; i < age; i++) {
      await Future.delayed(Duration(seconds: 2));
      yield 'Bark $i times';
    }
  }
}
```

- async 와 async* 의 차이는 [여기](https://stackoverflow.com/questions/55397023/whats-the-difference-between-async-and-async-in-dart)를 참고한다. 간략히 설명하면 return 타입이 `Stream<T>` 를 생산하면(+ method 를 떠나지 않으면서) async* 를 사용한다.
- 이런 경우 return 을 사용하지 않는다. method 를 떠나지 않기 때문에 종결처리 하면 안된다. yield 를 사용하며, yield 의 사전적 의미는 '생산하다' 라는 뜻이다.

<br>

# Consumer

![](/images/Consumer-page-001.jpg)
![](/images/Consumer-page-002.jpg)

```dart
class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Provider 08'),
      ),
      body: Consumer<Dog>(
        builder: (BuildContext context, Dog dog, Widget? child) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              mainAxisSize: MainAxisSize.min,
              children: [
                child!,
                SizedBox(height: 10.0),
                Text(
                  '- name: ${dog.name}',
                  style: TextStyle(fontSize: 20.0),
                ),
                SizedBox(height: 10.0),
                BreedAndAge(),
              ],
            ),
          );
        },
        child: Text(
          'I like dogs very much',
          style: TextStyle(fontSize: 20.0),
        ),
      ),
    );
  }
}
```

- Consumer 의 파라미터 (BuildContext context, Dog dog, Widget? child) 중 child 는 builder 내에서 rebuild 될 필요가 없는 Widget 이 있는 경우를 대비해서 위와 같이 사용한다. 위 예제에서는 어떤 경우든 rebuild 될 필요가 없는 Text Widget 을 child 로 빼주었다.

<br>

# Consumer, builder, ProviderNotFoundException

![](/images/Consumer-builder-ProviderNotFoundException-page-001.jpg)
![](/images/Consumer-builder-ProviderNotFoundException-page-002.jpg)
![](/images/Consumer-builder-ProviderNotFoundException-page-003.jpg)
![](/images/Consumer-builder-ProviderNotFoundException-page-004.jpg)
![](/images/Consumer-builder-ProviderNotFoundException-page-005.jpg)
![](/images/Consumer-builder-ProviderNotFoundException-page-006.jpg)

- Consumer 를 쓰든 builder 를 쓰든 둘 중 편한 방법을 사용하자.

<br>

# Selector

![](/images/Selector-page-001.jpg)

- Consumer 와 유사한데 Consumer 보다 더 세세한 컨트롤을 가능하게 해준다.
- 앞에서 학습한 `context.select<T, R>((R selector(T value))) => R` 과 유사한 개념이다.
- 지금 학습한 Selector 라는 Widget 과 `context.select<T, R>((R selector(T value))) => R` 는 공통적으로 특정 property 의 변경에 대해서 react 할 수 있도록 하는 것.

<br>

# ProviderNotFoundException 더 알아보기, Builder Widget

![](/images/Selector-page-001.jpg)



















