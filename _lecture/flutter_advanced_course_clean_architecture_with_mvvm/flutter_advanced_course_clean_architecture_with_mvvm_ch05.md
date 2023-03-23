---
layout: default
title: CH 5 Presentation Layer - MVVM
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 5
---

## MVVM 기본 패턴

[유투브](https://www.youtube.com/watch?v=wh6jfZJelf0&t=75s) 에 괜찮은 강의가 있어 참고했다. udemy 강의에서는 계층이 너무 분리가 되어서
일단 MVVM 자체에 대한 이해를 높히기 위해서 기본 패턴을 먼저 학습했다.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_clean_architecture/presentation/simple_mvvm/simple_view_model.dart';

class SimpleScreen extends StatefulWidget {
  @override
  _SimpleScreenState createState() => _SimpleScreenState();
}

class _SimpleScreenState extends State<SimpleScreen> {
  SimpleViewModel viewModel = SimpleViewModel();

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Scaffold(
        body: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.center,
          mainAxisSize: MainAxisSize.max,
          children: [
            StreamBuilder(
              stream: viewModel.mvvmStream,
              builder: (context, snapshot) {
                print('StreamBuilder > build > snapshot : ${snapshot.data}');
                int count = 0;
                if (snapshot.data != null) {
                  count = snapshot.data!.count;
                }
                return Center(
                  child: Text(
                    count.toString(),
                    style: TextStyle(fontSize: 30),
                  ),
                );
              },
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                IconButton(
                  onPressed: () {
                    viewModel.increaseCounter();
                  },
                  icon: Icon(Icons.exposure_plus_1),
                ),
                IconButton(
                  onPressed: () {
                    viewModel.decreaseCounter();
                  },
                  icon: Icon(Icons.exposure_minus_1),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```
```dart
import 'dart:async';

import 'package:flutter_clean_architecture/presentation/simple_mvvm/simple_model.dart';

class SimpleViewModel {
  late SimpleModel _model;
  final StreamController<SimpleModel> _streamController =
      StreamController<SimpleModel>();

  Stream<SimpleModel> get mvvmStream => _streamController.stream;

  SimpleViewModel() {
    _model = SimpleModel();
  }

  void update() {
    _streamController.sink.add(_model);
  }

  void increaseCounter() {
    _model.count++;
    update();
  }

  void decreaseCounter() {
    _model.count--;
    update();
  }
}
```
```dart
class SimpleModel {
  int count = 0;

  @override
  String toString() {
    return 'SimpleModel{count: $count}';
  }
}
```

Provider 에서 notifyListeners(); 를 사용하는 방식과 유사하다. 결국 Stream 도 Publish, Subscribe 원리이며 이를 이용해서
해당 Stream 을 구독하는 StreamBuilder 를 통해서 변경이 있을때마다 이를 받아 rebuild 하는 방식이다.

## 강의에서 사용된 MVVM 패턴

[소스코드](https://github.com/minafarideleia/complete_advanced_flutter/tree/Lecture_41_How_to_Recive_Data_in_View_From_Viewmodel) 가 너무 길어서 링크만 남긴다.

위 기본형과 비슷하지만 가장 크게 다른 부분은 계층이 더 세분화 되어 있다는 것이다. 대표적으로 아래 클래스가 있다.
```dart
import 'dart:async';

import 'package:complete_advanced_flutter/presentation/common/state_renderer/state_render_impl.dart';
import 'package:rxdart/rxdart.dart';

abstract class BaseViewModel extends BaseViewModelInputs
    with BaseViewModelOutputs {
  StreamController _inputStateStreamController =
  BehaviorSubject<FlowState>();

  @override
  Sink get inputState => _inputStateStreamController.sink;

  @override
  Stream<FlowState> get outputState =>
      _inputStateStreamController.stream.map((flowState) => flowState);

  @override
  void dispose() {
    _inputStateStreamController.close();
  }

// shared variables and functions that will be used through any view model.
}

abstract class BaseViewModelInputs {
  void start(); // will be called while init. of view model
  void dispose(); // will be called when viewmodel dies.

  Sink get inputState;
}

abstract class BaseViewModelOutputs {
  Stream<FlowState> get outputState;
}
```

BaseViewModel 라는 최상위 class 를 만들고 모든 ViewModel 이 이를 상속하도록 한다. 각 ViewModel 에서도 또 계층을 아래와 같이 나눈다.

```dart
// inputs mean the orders that our view model will recieve from our view
abstract class OnBoardingViewModelInputs {
  void goNext(); // when user clicks on right arrow or swipe left.
  void goPrevious(); // when user clicks on left arrow or swipe right.
  void onPageChanged(int index);

  Sink
      get inputSliderViewObject; // this is the way to add data to the stream .. stream input
}

// outputs mean data or results that will be sent from our view model to our view
abstract class OnBoardingViewModelOutputs {
  Stream<SliderViewObject> get outputSliderViewObject;
}

class SliderViewObject {
  SliderObject sliderObject;
  int numOfSlides;
  int currentIndex;

  SliderViewObject(this.sliderObject, this.numOfSlides, this.currentIndex);
}
```

이렇게 ViewModel 이 있는 .dart 에 위와 같이 abstract class 를 또 만들어서 BaseViewModel 을 상속하게 한뒤 저것들을 다 같이 mixin 한다.

goNext, goPrevious, onPageChanged 는 해당 ViewModel 특유의 것이라서 ViewModel 에 선언해도 되지만 아마도 명시적으로 ~Input 내에 넣어주어서 확실하게
분리시키려는 의도로 보인다.

OnBoardingViewModelInputs 의 inputSliderViewObject 은 좀 잘못된 사용이라고 나는 생각한다. 왜냐면 저건 ViewModel -> View 로의 명확한 방향성이 있는데
OnBoardingViewModelInputs 내에 위치함으로써 마치 View -> ViewModel 인 것처럼 오해를 불러 일으킨다.

강사도 주석에 add data to the stream 이라고 정확하게 써놨다. 즉, 이건 ViewModel 에서 변경된 데이터를 View 가 Stream 에서 받아 쓸 수 있도록 Stream 에 넣어주는 용도라는 말인데
정확히 방향이 ViewModel -> View 다. 그런데 ~ViewModelInputs 안에 넣어두면 말이 맞지 않다.

OnBoardingViewModelOutputs 도 굳이 저렇게 따로 빼줄 필요까진 없다고 보는데 아마도 명확한 분리르 위해서 명시적으로 그냥 저렇게 처리한 것 같다.