---
layout: default
title: CH 2 Bloc Overview
parent: Flutter Bloc Essential
nav_order: 2
---

<br>

![](/images/Section+Overview-page-001.jpg)
![](/images/Section+Overview-page-002.jpg)
![](/images/Section+Overview-page-003.jpg)
![](/images/Section+Overview-page-004.jpg)
![](/images/Section+Overview-page-005.jpg)
![](/images/Section+Overview-page-006.jpg)
![](/images/Section+Overview-page-007.jpg)
![](/images/Section+Overview-page-008.jpg)
![](/images/Section+Overview-page-009.jpg)
![](/images/Section+Overview-page-010.jpg)
![](/images/Section+Overview-page-011.jpg)
![](/images/Section+Overview-page-012.jpg)
![](/images/Section+Overview-page-013.jpg)
![](/images/injecting+cubits+or+blocs-page-001.jpg)
![](/images/injecting+cubits+or+blocs-page-002.jpg)
![](/images/BlocBuilder-page-001.jpg)
![](/images/BlocListener+and+BlocConsumer-page-001.jpg)
![](/images/BlocListener+and+BlocConsumer-page-002.jpg)
![](/images/BlocListener+and+BlocConsumer-page-003.jpg)
![](/images/BlocListener+and+BlocConsumer-page-004.jpg)
![](/images/BlocListener+and+BlocConsumer-page-005.jpg)
![](/images/BlocListener+and+BlocConsumer-page-006.jpg)
![](/images/BuildContext+extension+methods-page-001.jpg)
![](/images/BuildContext+extension+methods-page-002.jpg)
![](/images/BuildContext+extension+methods-page-003.jpg)
![](/images/BuildContext+extension+methods-page-004.jpg)
![](/images/BuildContext+extension+methods-page-005.jpg)
![](/images/BuildContext+extension+methods-page-006.jpg)
![](/images/Bloc+Access+-+context-page-001.jpg)
![](/images/Bloc+Access+-+context-page-002.jpg)
![](/images/Bloc+Access+-+context-page-003.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-001.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-002.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-003.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-004.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-005.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-006.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-007.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-008.jpg)
![](/images/19+Observing+Cubit+and+Blocs(new)-page-009.jpg)
![](/images/Event+Transformer-page-001.jpg)
![](/images/Event+Transformer-page-002.jpg)
![](/images/Event+Transformer-page-003.jpg)
![](/images/Event+Transformer-page-004.jpg)
![](/images/Event+Transformer-page-005.jpg)

```dart
import 'package:bloc/bloc.dart';
import 'package:bloc_concurrency/bloc_concurrency.dart';
import 'package:equatable/equatable.dart';

part 'counter_event.dart';
part 'counter_state.dart';

class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterState.initial()) {
    // on<IncrementCounterEvent>(
    //   _handleIncrementCounterEvent,
    //   transformer: sequential(),
    // );

    // on<DecrementCounterEvent>(
    //   _handleDecrementCounterEvent,
    //   transformer: sequential(),
    // );

    on<CounterEvent>(
      (event, emit) async {
        if (event is IncrementCounterEvent) {
          await _handleIncrementCounterEvent(event, emit);
        } else if (event is DecrementCounterEvent) {
          await _handleDecrementCounterEvent(event, emit);
        }
      },
      transformer: sequential(),
    );
  }

  Future<void> _handleIncrementCounterEvent(event, emit) async {
    await Future.delayed(Duration(seconds: 4));
    emit(state.copyWith(counter: state.counter + 1));
  }

  Future<void> _handleDecrementCounterEvent(event, emit) async {
    await Future.delayed(Duration(seconds: 2));
    emit(state.copyWith(counter: state.counter - 1));
  }
}
```

![](/images/24+Hydrated+Bloc+(9.0)-page-001.jpg)
![](/images/24+Hydrated+Bloc+(9.0)-page-002.jpg)
![](/images/24+Hydrated+Bloc+(9.0)-page-003.jpg)
![](/images/24+Hydrated+Bloc+(9.0)-page-004.jpg)
![](/images/24+Hydrated+Bloc+(9.0)-page-005.jpg)
![](/images/24+Hydrated+Bloc+(9.0)-page-006.jpg)
![](/images/24+Hydrated+Bloc+(9.0)-page-007.jpg)
![](/images/RepositoryProvider-page-001.jpg)
![](/images/RepositoryProvider-page-002.jpg)
![](/images/RepositoryProvider-page-003.jpg)
![](/images/RepositoryProvider-page-004.jpg)
![](/images/RepositoryProvider-page-005.jpg)
![](/images/Cubit+vs+Bloc-page-001.jpg)
![](/images/Cubit+vs+Bloc-page-002.jpg)
