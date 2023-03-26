---
layout: default
title: CH 8 Application Layer - Dependency Injection
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 8
---

<br>

## get_it 패키지를 이용한 dependency injection - registerSingleton  vs registerLazySingleton 

```dart
final locator = GetIt.instance;

Future<void> init() async {
  locator.registerSingleton<T>(() => ...);
  
  /// registers a type as Singleton by passing an [instance] of that type
  /// that will be returned on each call of [get] on that type
```
```dart
final locator = GetIt.instance;

Future<void> init() async {
  locator.registerLazySingleton<T>(() => ...);
  
  /// registers a type as Singleton by passing a factory function that will be called
  /// on the first call of [get] on that type  
```

강의에서는 registerLazySingleton 를 사용했고, 이 외에 다른 [유투브 강의](https://www.youtube.com/watch?v=DbV5RV2HRUk) 에서도 동일하게 registerLazySingleton 를 사용했다.

registerLazySingleton 는 말 그래도 lazy 하게 register 한다는 의미인데 위 설명에도 쓰여있듯이 register 하는 대상 자체가 registerLazySingleton 에서는 instance 가 아니라
factory function 이다. 비슷한 예로 JPA 에서 프록시 객체처럼 첫번째 call 때 비로소 실제 instance 가 생성된다고 볼 수 있다. 아래는 chat gpt 에서 차이를 물었을때 나온 설명이다.

<hr>

get_it 패키지는 의존성 주입을 구현하기 위한 패키지 중 하나입니다. 이 패키지에서는 singleton과 lazySingleton 두 가지 종류의 등록 방법을 제공합니다.

<br>

```dart
GetIt.instance.registerSingleton<MyService>(MyService());
```

singleton: singleton은 처음 사용될 때 인스턴스를 생성하고, 이후에는 항상 동일한 인스턴스를 반환하는 등록 방식입니다. get_it 패키지에서 registerSingleton 메서드를 사용하여 등록할 수 있습니다.

<br>

```dart
GetIt.instance.registerLazySingleton<MyService>(() => MyService());
```

lazySingleton: lazySingleton은 처음 사용될 때 인스턴스를 생성하고, 이후에는 항상 동일한 인스턴스를 반환하는 등록 방식입니다.
singleton과의 차이점은 처음 사용되기 전까지 인스턴스를 생성하지 않는다는 점입니다. get_it 패키지에서 registerLazySingleton 메서드를 사용하여 등록할 수 있습니다.

즉, singleton은 등록 시점에 바로 인스턴스를 생성하고, lazySingleton은 인스턴스가 필요할 때 생성됩니다.
따라서 lazySingleton은 애플리케이션이 시작될 때 많은 메모리를 차지하는 경우 유용합니다. 일반적으로 애플리케이션에서 사용하는 대부분의 서비스는 lazySingleton으로 등록하는 것이 좋습니다.

<hr>

<br>

## get_it 패키지를 이용한 dependency injection - module 에 필요한 di 처리 (usecase, viewModel 등)

강의에서는 아래와 같이 처리하고 있다.

```dart
final instance = GetIt.instance;

Future<void> initAppModule() async {
  final sharedPrefs = await SharedPreferences.getInstance();

  // shared prefs instance
  instance.registerLazySingleton<SharedPreferences>(() => sharedPrefs);

  // app prefs instance
  instance
      .registerLazySingleton<AppPreferences>(() => AppPreferences(instance()));

  // network info
  instance.registerLazySingleton<NetworkInfo>(
      () => NetworkInfoImpl(DataConnectionChecker()));

  // dio factory
  instance.registerLazySingleton<DioFactory>(() => DioFactory(instance()));

  // app  service client
  final dio = await instance<DioFactory>().getDio();
  instance.registerLazySingleton<AppServiceClient>(() => AppServiceClient(dio));

  // remote data source
  instance.registerLazySingleton<RemoteDataSource>(
      () => RemoteDataSourceImplementer(instance()));

  // local data source
  instance.registerLazySingleton<LocalDataSource>(
      () => LocalDataSourceImplementer());

  // repository
  instance.registerLazySingleton<Repository>(
      () => RepositoryImpl(instance(), instance(), instance()));
}

initLoginModule() {
  if (!GetIt.I.isRegistered<LoginUseCase>()) {
    instance.registerFactory<LoginUseCase>(() => LoginUseCase(instance()));
    instance.registerFactory<LoginViewModel>(() => LoginViewModel(instance()));
  }
}
```

initAppModule 에서 application layer 의 di를 모두 처리해주고, 로그인 view 에서 필요한 usecase, viewModel 은 initLoginModule 에서 따로 처리해준다.
그리고 이를 아래와 같이 resetModules 에서 한번에 처리해주면서도 route 에서도 따로 또 넣어주고 있다.

```dart
resetModules() {
  instance.reset(dispose: false);
  initAppModule();
  initHomeModule();
  initLoginModule();
  initRegisterModule();
  initForgotPasswordModule();
  initStoreDetailsModule();
}
```
```dart
  static Route<dynamic> getRoute(RouteSettings routeSettings) {
    switch (routeSettings.name) {
      case Routes.splashRoute:
        return MaterialPageRoute(builder: (_) => SplashView());
      case Routes.loginRoute:
        initLoginModule();
        return MaterialPageRoute(builder: (_) => LoginView());
      case Routes.onBoardingRoute:
        return MaterialPageRoute(builder: (_) => OnBoardingView());
      case Routes.registerRoute:
        initRegisterModule();
        return MaterialPageRoute(builder: (_) => RegisterView());
      case Routes.forgotPasswordRoute:
        initForgotPasswordModule();
        return MaterialPageRoute(builder: (_) => ForgotPasswordView());
      case Routes.mainRoute:
        initHomeModule();
        return MaterialPageRoute(builder: (_) => MainView());
      case Routes.storeDetailsRoute:
        initStoreDetailsModule();
        return MaterialPageRoute(builder: (_) => StoreDetailsView());
      default:
        return unDefinedRoute();
    }
  }
```

그리고 usecase, viewModel 의 di 처리와 application layer 에서의 di 처리 사이에 가장 큰 차이라 한다면 registerFactory() 를 사용했다는 것이다.

```dart
  /// registers a type so that a new instance will be created on each call of [get] on that type
  /// [T] type to register
  /// [factoryFunc] factory function for this type
  /// [instanceName] if you provide a value here your factory gets registered with that
  /// name instead of a type. This should only be necessary if you need to register more
  /// than one instance of one type. Its highly not recommended
  void registerFactory<T extends Object>(
    FactoryFunc<T> factoryFunc, {
    String? instanceName,
  });
```
`registers a type so that a new instance will be created on each call of [get] on that type` 설명과 같이 사용 될 때마다 새로운 객체를 반환해주도록 하기 위해서 Factory 를 등록한다.

여기까지가 강의에서 사용된 usecase, viewModel 에 대한 구현 방식이다. 강의에서 명확하게 이 부분을 왜 다르게 구현했는가를 설명을 해주지 않고 있다.

결과적으로 usecase, viewModel 에 대해서 registerLazySingleton 과 registerFactory 를 사용했을때 각각 어떻게 다르게 처리 되는가를 생각해보면 좋을 것 같다.
일단 registerLazySingleton 의 경우 계속 썼던 것을 재활용하게 되는 것이고 registerFactory 를 사용하면 쓸 때마다 새로 만들어서 사용하는 것이다.

이때 새로 만들게 되면 당연히 지역변수와 같은 객체의 상태자체가 전부 초기화 되어서 처음 상태로 돌아가는 것이고 상태를 유지시키고 싶다면 singleton 을 사용해야 할 것이다.
viewModel 에서 특별히 유지해야할 상태가 없고, 받아와서 보여주는 데이터 모두 서버와 동기화로 동작하는 것이라면 굳이 새로 만들 필요 없이 singleton 을 사용하는 것이 나을 것 같다.

<br>

## get_it 패키지를 이용한 dependency injection - 사용 형태

강의 코드처럼 쓰는 것은 명시적인 class 가 없어서 개인적으로 내가 선호하는 스타일은 아니다.
차후 프로젝트 진행시 아래와 같이 class 내에서 static 필드 정의를 해주고 꺼내 쓰도록 하자.

```dart
class Injector {
  Injector._();

  static GetIt get locator => GetIt.instance;

  static Future<void> init() async {
    locator.registerLazySingleton<NetworkStatus>(
        () => NetworkStatusImpl(DataConnectionChecker()));

    final sharedPrefs = await SharedPreferences.getInstance();
    locator
        .registerLazySingleton<AppPreference>(() => AppPreference(sharedPrefs));

    locator.registerLazySingleton<DioFactory>(() => DioFactory(locator()));

    final dio = await locator<DioFactory>().getDio();
    locator.registerLazySingleton<AppHttpClient>(() => AppHttpClient(dio));

    locator.registerLazySingleton<RemoteDataSource>(() =>
        RemoteDataSourceImpl(locator<AppHttpClient>())); // <AppHttpClient> 생략가능

    locator.registerLazySingleton<UserRepository>(
        () => UserRepositoryImpl(locator(), locator()));
    // locator.registerLazySingleton<UserRepository>(
    //     () => UserRepositoryImpl(locator<NetworkStatus>(), locator<RemoteDataSource>()));
  }

  static initRegisterModule() {
    if (!locator.isRegistered<RegisterUseCase>()) {
      locator
          .registerFactory<RegisterUseCase>(() => RegisterUseCase(locator()));
    }
  }
}
```
