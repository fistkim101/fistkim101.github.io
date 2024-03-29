---
layout: default
title: 02 JAVA GC
parent: TIL
nav_order: 2
---

거의 대부분의 주제들이 그렇겠지만 특히 GC에 대해서는 매우 자세하게 다루고 있는 이미 나와있는 여러 좋은 레퍼런스들이 많다.
발행된지는 좀 됬지만 [네이버 D2에서 나온 포스팅](https://d2.naver.com/helloworld/1329) 이 GC를 이해하는데 도움이 많이 되었다.
그래서 이번 포스팅은 위 포스팅을 기준으로하여 궁금한 것들을 찾아가면서 JVM의 GC에 대한 나의 이해를 정리하는 것을 목표로 했다.

<br>

## GC는 무엇이며 어떻게 동작할까

### GC가 동작하기 위한 조건 'stop-the-world'
GC역시 하나의 처리이므로 자원을 끌어다 쓰는 행위이다. GC는 'stop-the-world'의 개념 아래에서 동작한다. 'stop-the-world'란 GC를 실행하기 위해서 JVM이 어플리케이션 자체를 멈춰버리는 것을 뜻한다.
GC의 동작을 위해서 JVM이 'stop-the-world'를 한다는 것은 GC를 수행하는 쓰레드를 제외한 모든 쓰레드를 멈춘다는 것을 의미한다.
GC의 동작 알고리즘에는 여러 종류가 있는데 어떤 종류의 알고리즘을 택하더라도 GC를 실행하기 위해서는 'stop-the-world' 환경 아래에서 동작하여야 한다.
그래서 GC 튜닝을 한다면 대게 'stop-the-world' 환경에 묶인 시간을 줄이는 것이 목적이 된다고 한다.

<br>

### GC의 대상, 무엇이 가비지인가
가비지 컬렉터는 말 그대로 [가비지 컬렉션](https://ko.wikipedia.org/wiki/%EC%93%B0%EB%A0%88%EA%B8%B0_%EC%88%98%EC%A7%91_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))을 수행하는 주체이다.
그러면 결국엔 '가비지'라는 것을 어떻게 규정하느냐가 중요해진다. 여기서 가비지는 '어떠한 객체로부터 유효한 참조가 되지 않고 있는 객체' 를 뜻한다.
즉, 존재할 의미가 없는데(=쓰이지도 않는데) 메모리 상에서 자리만 차지하고 있는 것을 의미한다. 더 압축적으로 말하면 [메모리 누수](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EB%A6%AC_%EB%88%84%EC%88%98)를 유발하는 것이 곧 가비지이다.
reachable vs unreachable 에 대한 판단이 끝나고 unreachable로 판단되면 가비지로 간주되어서 메모리 해제의 대상이 된다고 할 수 있다.

<br>

### GC는 어떻게 메모리를 회수할까
기본적으로 [Mark and Sweep](https://nobilitycat.tistory.com/entry/Mark-and-Sweep) 전략을 사용한다.
자세한 것은 링크에 설명되어 있다. 나의 이해를 간략히 정리하자면 아래와 같다. 예시가 조금 살벌하지만 이해하기 편하다.

1) GC는 모든 오브젝트에 접근할 수 있는데 이 때 마치 모든 오브젝트의 데스노트와 같은 Map을 가지고 있다.

2) 일단 모든 오브젝트의 이름을 데스노트에 적는다

3) 모든 오브젝트를 순회하며 reachable한 오브젝트는 데스노트에서 이름을 지워준다

4) 3번 과정이 끝나면 데스노트에 적힌 모든 오브젝트의 메모리를 회수한다.

<br>

### 정리
지금까지 이해한 사항을 정리해보면 아래와 같다.
* 대전제 : GC는 running time에 계속해서 할당될 수 있는 heap 공간에서 활동한다.
* GC는 GC를 수행하는 쓰레드를 제외한 모든 쓰레드가 멈춰진 'stop-the-world' 상태에서 작동한다.
* 가비지라는 것은 unreachable한 객체(유효한 참조가 이뤄지지 않고 있는 객체)를 의미한다.

<br>

### GC는 언제 동작해야 할까, 자주 동작할 수록 좋은 건가?
위에서 정리된 바와 같이 GC는 'stop-the-world' 환경에서 동작하기 때문에 이것 자체가 부담이거니와,
복잡도가 크고 규모가 큰 어플리케이션일 수록 한번 GC가 작동하는 것 자체가 비용일 수 있다.

그래서 이러한 효율의 문제를 감안하여 최적으로 GC를 활용하기 위해서 GC 작동하는 메모리상의 구역과
이에 따라 작동 시기가 구분되어 작동한다. 기본적으로 이러한 메모리상의 구분과 작동 시기에 대한 구분은
아래 가설을 전제로 한다.('약한 세대 가설' 이라고 칭한다)

> * 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
> * 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.

쉽게 정리하면 '보통은 한번 쓰고 안쓰는 객체가 대부분이고, 만든지 오래된 객체가 금방 만든 객체를 갖다 쓸 일이 거의 없다'는 뜻이다.
이 전제를 GC입장에서 해석하자면 '지금 unreachable한 상태라면 앞으로도 unreachable한 상태일 가능성이 크니까 지워도 되겠지' 라는 해석이 된다.

<br>

### GC의 메모리 구역에 따른 작동원리
기본적으로 메모리를 Young, Old 두 가지 영역으로 나누어서 각 영역에서 GC가 따로 동작하며, 살아 남은 객체가 promotion 되는 원리로 관리가 된다.
Young에서 사용되는 GC를 Minor GC라고 하며 Old에서 사용되는 GC를 Major GC라고 한다.

<br>

![](/images/gc-area.jpeg)

1) Young은 Eden이라는 하나의 영역과 Survivor이라는 두 개의 영역으로 나뉘어지는데, 객체가 새로 생성이 되면 무조건 Eden에 생성이 된다.

2) 계속해서 Eden에 객체가 생성이 되어 새로 생성될 자리가 없어지게 되면 Minor GC가 동작한다. 이 때 Eden에서 unreachable한 것은 메모리가 회수 되며 reachable한 것은 Survivor로 옮겨진다.

3) Survivor로 옮겨진 것도 안전하지 않다. Minor GC가 작동할 때마다 항상 메모리 회수의 대상이 되는데, 1~2의 과정에 의해서 Survivor에 누적적으로 쌓이다가 Survivor도 포화가 되면 다른 Survivor로 옮겨진다.

4) Survivor가 포화가 되면 결국 Old 영역으로 옮겨지게 된다. 1~4의 과정으로 Old에 계속해서 쌓이다가 Old가 꽉차면 그때 Major GC가 동작하여 Old 영역을 정리하게 된다.

cf) Old의 객체가 Young의 객체를 참조하게 될 경우 card table에 기록으로 쌓게 되며 Minor GC가 동작할 때 이 테이블을 참고하여 Old 영역에 있는 객체가 참조하고 있는 Young은 회수를 하지 않도록 한다.

<br>

### 왜 Survivor은 두 개 인가?
메모리 단편화 때문에 두 개를 갖고 있게 된다. 즉 계속 할당을 해주고 회수를 해주고 하는 사이에 단편화가 발생해서 총 빈공간이 남아는 있지만 중간중간 빈틈이 생겨버리는 것이다.
이를 해결하기 위해서 한 쪽 Survivor에서 다른 한 쪽 Survivor로 옮기면서 빈공간을 없애고 예쁘게 적재하는 식으로 처리하다가 더 이상 담을 수 없게 되면 Old 로 옮기게 되는 것이다.

그렇기 때문에 두 개의 Survivor에 데이터가 쌓여 있으면 안된다. 무조건 하나는 비어져 있는 상태가 정상인 것이다.
