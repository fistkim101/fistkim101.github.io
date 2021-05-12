---
layout: article
title: Servlet
tags: Spring Servlet DispatcherServlet DI IOC
---

<style>
red { color: Red }
orange { color: Orange }
green { color: Green }
blue { color: Blue }
</style>

<br>
<br>

### 들어가며
요즘 너무 강의만 듣고 강의커리큘럼에 학습 플랜 자체를 맡기다보니 호기심-driven 된 학습이 되지 않아서 효율이 좀 떨어짐을 느꼈다.
그래서 기존에 당연하다고 느꼈던 것들이나 '알고 있다고 착각'하는 것들에 대해서 다시 곱씹어보면서 '내가 모르는데 아는 것으로 착각했구나'하는 이슈들을 잡아서
하나씩 정리해나가는 중이다.

그런 성격의 이슈들 중 하나는 서블릿이고 이번 포스팅에 이걸 전반적으로 정리하려고 한다.
서블릿 자체만으로는 정리하는게 큰 의미가 있지 않고, 스프링과 연계하여 Servlet 이 어떤 역할을 하는지
야크쉐이빙하듯 정리해나가면 자연스럽게 실무적인 부분까지 엮어서 정리가 될 것 같다.

<br>

### 서블릿이란
DispatcherServlet 정리하기 전에 기본적인 서블릿 자체에 대한 개념부터 확실히 다지는게 필요한 것 같다.
[공식문서](https://docs.oracle.com/javaee/5/tutorial/doc/bnafe.html) 를 보면 아래와 같이 정리가 되어 있다.

> **What is a Servlet?**
 <br><br>
 A servlet is a Java programming language class that is used to extend the capabilities of servers that host applications accessed by means of a request-response programming model. Although servlets can respond to any type of request, they are commonly used to extend the applications hosted by web servers. For such applications, Java Servlet technology defines HTTP-specific servlet classes.<br><br>
 The javax.servlet and javax.servlet.http packages provide interfaces and classes for writing servlets. All servlets must implement the Servlet interface, which defines life-cycle methods. When implementing a generic service, you can use or extend the GenericServlet class provided with the Java Servlet API. The HttpServlet class provides methods, such as doGet and doPost, for handling HTTP-specific services.
 This chapter focuses on writing servlets that generate responses to HTTP requests.

두 문단을 합쳐서 이해를 바탕으로 다시 정리해보면 아래와 같이 정리가 된다.

**서블릿은 javax.servlet 과 javax.servlet.http 패키지가 제공하는 인터페이스와 클래스를 이용해서 만든 '자바 클래스'이다.
서블릿의 목적은 request-response programming model을 염두하고 만든 host application 이 원활하게 작동하도록 여러 기능들을 제공하는데 있다.
모든 서블릿은 결국 Servlet interface를 상속하기 때문에, 필연적으로 life-cycle methods를 지니게 된다.**

**<red>쉽게 말해서, javax.servlet 과 javax.servlet.http 패키지는 자바로 웹 어플리케이션을 만들 수 있도록 여러 자원(인터페이스, 클래스 = API)를 제공해주고 있고
이 자원들로 구현한 구현체들이 모두 서블릿인 것이다. 예를 들어 HttpServlet은 doGet이나 doPost와 같은 HTTP에 포커스된 서비스를 위한 메소드를 제공하고 있고,
이 HttpServlet를 상속하는 DispatcherServlet가 스프링프레임워크 맨 앞단에서 여러 처리를 해주는 사례를 들 수 있다.</red>**

추가로 알아둬야할 서블릿의 특징으로는 1요청당 1프로세스로 처리되는 것이 아니라 1요청당 1스레드를 만들어서 요청을 처리한다는 것이다.
이게 의미가 있는 이유는 스레드는 곧 하나의 프로세스 내에서 자원을 공유할 수 있어서(Thread Local) 기존의 기술에 비해서 속도가 빠르다는 것이다.

<br>

### 서블릿 컨테이너란
**<red>서블릿 스펙을 준수하여 만들어진 컨테이너로서 서블릿 handling 의 전반을 책임진다.</red>**
다르게 말하면 서블릿을 아무리 만들어도 우리는 서블릿 자체만을 직접 사용할 수 없고, 반드시 서블릿 컨테이너를 통해서 서블릿의 life-cycle에 맞춰서 서블릿을 사용할 수 있다는 것이다.
그래서 서블릿 컨테이너의 가장 주요한 기능이 '서블릿 생명주기 관리'이며 그 외에 세션 관리, 네트워크 서비스(ssl, http connect 등) 등을 수행한다.

<br>

### 서블릿의 생명주기
* 1) 요청을 받으면 서블릿 컨테이너가 서블릿 인스턴스의 init() 메소드를 호출하여 서블릿을 초기화한다.(동일 요청이 들어온 경우 init() 다시 호출하진 않음)
* 2) init() 이후 서블릿 컨테이너에서 request, response 객체를 만들고 요청을 처리하기 위해 스레드를 생성한다.
* 3) service() 를 호출하고 곧 http method와 url 매핑을 고려해서 doGet(), doPost() 등 의 메소드로 처리가 위임된다.
* 4) 처리된 응답을 response를 통해서 내보낸 이후 request, response 객체가 소멸하고 곧 이어 스레드가 소멸된다.

{% include image.html src="/assets/images/spring/servlet-life-cycle.png" text="출처: https://galid1.tistory.com/487" %}

<br>

### 서블릿 컨테이너와 DI 간의 관계
스프링의 [ContextLoaderListener](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/ContextLoaderListener.html) 는
서블릿의 생명주기에 맞춰서 [ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html) 를
각 서블릿이 bean 객체들을 사용할 수 있게끔 [ServletContext](https://docs.oracle.com/cd/E17802_01/products/products/servlet/2.5/docs/servlet-2_5-mr2/javax/servlet/ServletContext.html) 에 바인딩 해주는 역할을 한다.
더 풀어서 설명하자면 ServletContext(모든 서블릿이 접근 가능한 공용 저장소와 같은 개념이다)에 ApplicationContext를 바인딩해주는 ContextLoaderListener가 DI를 가능하게 한다고 할 수 있다.

즉, ContextLoaderListener 의 존재 덕분에 서블릿이 ApplicationContext 에 접근하여 결과적으로 bean 객체들을 사용할 수 있다는 것이다. 그리고 ContextLoaderListener 는 이미 존재하고 있는
ApplicationContext를 단지 연결해주는 것이 아니라 이를 만들어서 바인딩해주는데 이때 ApplicationContext를 만들때 참조하는 것이 곧 스프링 설정이다.

<br>

### 스프링부트와 스프링프레임워크의 차이
지금까지 살펴본 관계는 스프링프레임워크에서의 서블릿 컨테이너와 DI의 관계였다. 스프링부트는 스프링 어플리케이션이 먼저 뜨고 그 안에
내장된 서블릿 컨테이너가 뜨고 그 내부에 bean들을 등록하게 되는 구조이다. 하지만 그러한 구조의 차이를 제외하고는 기본적으로 동작하는 방식은 비슷하다고 볼 수 있다.

* 스프링부트 => 스프링 안에 서블릿 컨테이너를 넣음
* 스프링프레임워크 => 서블릿 컨테이너 안에 스프링을 넣음

<br>

### DispatcherServlet 작동순서

{% include image.html src="/assets/images/spring/dispatcherservlet-interfaces.png" text="DispatcherServlet 이 갖고 있는 모든 인터페이스들" %}
DispatcherServlet은 위 그림에 나온 모든 인터페이스들을 가지고 [FrontController](https://en.wikipedia.org/wiki/Front_controller) 로서 요청을 처리한다.
이 사실을 염두하고 아래의 DispatcherServlet 작동 순서를 살펴보자.

<br>

1. HandlerMapping 에게 위임하여 요청을 처리할 핸들러를 찾는다. (핸들러 찾지 못하면 에러 발생)
2. 핸들러를 찾으면 이 핸들러를 실행할 수 있는 핸들러 어댑터를 찾는다.
3. HandlerAdapter 가 invokeHandlerMethod()를 통해서 해당 핸들러를 실행시킨다.
4. 응답값에 따라서 다르게 행동한다.<br>
    4-1. 뷰를 찾아줘야 할 경우 경우에 따라 RequestToViewNameTranslator, ViewResolver 에게 위임해서 뷰를 찾아고 모델 데이터를 렌더링한다.<br>
    4-2. @ResponseBody 가 있으면 Converter 를 사용해서 응답 본문을 만든다.<br>
    4-3. 에러가 발생했다면 HandlerExceptionResolver 에게 요청 처리를 위임한다.<br>
5. 최종적으로 응답을 보낸다.

<br>
<br>
