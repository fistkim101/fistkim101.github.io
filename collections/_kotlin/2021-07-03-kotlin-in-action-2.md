---
layout: article
title: (Kotlin In Action) 2장 코틀린 기초
tags: 코틀린
---

<style>
red { color: Red }
orange { color: Orange }
green { color: Green }
blue { color: Blue }
</style>

<br>
<br>

# 목차

* 2.1 기본 요소: 함수와 변수
    * 2.1.1 Hello, World!
    * 2.1.2 함수
    * 2.1.3 변수
    * 2.1.4 더 쉽게 문자열 형식 지정: 문자열 템플릿
* 2.2 클래스와 프로퍼티
    * 2.2.1 프로퍼티
    * 2.2.2 커스텀 접근자
    * 2.2.3 코틀린 소스코드 구조: 디렉터리와 패키지
* 2.3 선택 표현과 처리: enum과 when
    * 2.3.1 enum 클래스 정의
    * 2.3.2 when 으로 enum 클래스 다루기
    * 2.3.3 when 과 임의의 객체를 함께 사용
    * 2.3.4 인자 없는 when 사용
    * 2.3.5 스마트 캐스트: 타입 검사와 타입 캐스트를 조합
    * 2.3.6 리팩토링: if를 when 으로 변경
    * 2.3.7 if와 when의 분기에서 블록 사용
* 2.4 대상을 이터레이션: while 과 for 루프
    * 2.4.1 while 루프
    * 2.4.2 수에 대한 이터레이션: 범위와 수열
    * 2.4.3 맵에 대한 이터레이션
    * 2.4.4 in 으로 컬렉션이나 범위의 원소 검사
* 2.5 코틀린의 예외 처리
    * 2.5.1 try, catch, finally
    * 2.5.2 try 를 식으로 사용
* 2.6 요약

<br>

# 학습내용

<br>

## 2.1 기본 요소: 함수와 변수
### 2.1.1 Hello, World!
이 후 이어질 소챕터들에서 어떤 내용들을 다룰지에 대한 간략한 안내를 해주는 소챕터였다. 아래 두 내용 빼고는 특별히 정리할 내용은 없었다.

* 함수를 최상위 수준에 정의할 수 있다. 자바처럼 반드시 클래스안에 함수를 넣지 않아도 된다.
* System.out.println 대신 println 을 쓸 수 있다. 사실 이게 중요한게 아니고 자바에서 사용하는 함수들을 감싼 래퍼 함수들을 제공하고 있어서
편하게 println 만 사용하면 되었고, 이런 식으로 제공하는 래퍼 함수가 많다고 한다. 실제로 println 을 들어가보면 아래와 같이 되어 있다.
```kotlin
/** Prints the given [message] and the line separator to the standard output stream. */
@kotlin.internal.InlineOnly
public actual inline fun println(message: Any?) {
    System.out.println(message)
}
```

<br>

### 2.1.2 함수
아래 세 가지가 이 소챕터의 학습 포인트였다.
* 코틀린의 기본적인 함수 형태
* expression 과 statement 의 차이(feat. if는 expression 이다)
* expression body function vs block body function

<br>

기본적인 함수 형태는 말 그대로 코틀린의 함수가 어떻게 생겼는지, 어떻게 선언 해야하는 지 였다. 예제로 나온 함수는 아래와 같다.
```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

<br>

코틀린에서는 이 식을 아래와 같이 조금 더 간결하게 표현할 수 있다.
```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

<br>

여기서 두 가지 시사점이 있는데 첫째는 코틀린에서는 '식이 본문인 함수(expression body function)' 과 '블록이 본문인 함수(block body function)' 이렇게
두 가지가 존재한다는 것이고 둘째는 '식이 본문인 함수(expression body function)' 인 경우 [타입 추론](https://en.wikipedia.org/wiki/Type_inference) 이
가능하다는 것이다.

타입 추론은 '식이 본문인 함수(expression body function)' 에서만 가능하다.
이 이유는 곧 '블록이 본문인 함수(block body function)' 에서 굳이 불가능하게 만든 이유와도 맥이 같을거라 생각되는데,
개발팀이 이에 대해서 밝힌 이유는 일반적으로 블록이 본문인 함수에서는 다수의 return 문이 등장하여 조기종결 처리를 하기 때문에
이런 경우 반환 타입을 명시적으로 써주는 것이 가독성 측면에서 더 좋기 때문이다. 합리적인 이유라는 생각이 들었다.

그리고 본문에서는 식(expression)과 문(statement)의 차이에 대해서도 따로 다루고 있었다.
**<red>expression 의 경우 결과적으로 value 를 만들어낸다. 그래서 다른 expression 은 상위 expression 의 부분이 될 수 있다.
이러한 장점을 이용해서 여러 일반적인 패턴을 간결하게 표현할 수 있는데, 자바에서는 모든 제어구조가 statement 인 반면에
코틀린에서는 루프를 제외한 대부분의 제어 구조가 expression 이다.</red>**

<br>

### 2.1.3 변수

코틀린은 statically typed 언어이기 때문에 컴파일 타임에 컴파일러가 타입을 반드시 알아야한다.
하지만 타입추론이 가능하기 때문에 결국 모든 선언된 변수에 대해서 아래 두 가지 액션중 하나는 취해줘야 한다.

* 명시적으로 타입을 선언해주거나
* 타입 추론이 가능하도록 해주거나(= 값을 할당)

그래서 초기화 식이 없으면 반드시 변수를 선언하면서 타입을 반드시 명시해줘야 한다.

<br>

#### val vs var

코틀린에서는 변수 선언시 아래 키워드 두 가지를 사용할 수 있다.

* val(from value) - 변경 불가능(immutable), 자바의 final 과 같아서 초기화 이후 재대입이 불가능
* var(from variable) - 변경 가능(mutable)

**<red>재미있는 사실은 책에서 직접적으로 var 보다는 val 사용을 권하고 있다는 것이다. 기본적으로 모든 변수를 val 키워드를
사용해서 불변 변수로 선언하라고 권고한다. 이유는 immutable과 순수 함수가 합쳐질때 코드가 함수형 프로그래밍에 가까워 지기 때문이다.</red>**

더 자세한 것은 5장에서 다룬다고 하니 일단은 val, var 의 차이만 숙지하고 넘어갔다.

<br>

### 2.1.4 더 쉽게 문자열 형식 지정: 문자열 템플릿

개인적인 경험으로는 실무에서 생각보다 문자열을 가공해서 결과를 만들어서 내려줘야했던 경우가 많았다.
API의 응답으로 문자열을 내려주지 않더라도 에러 메세지를 만들때 + 로 이를 만들다보니 불편한 경우가 많았다.

코틀린에서는 $ 를 통해서 문자열 안에서 변수를 사용할 수 있도록 해줌으로써 편리함을 제공한다.
아래와 같이 출력이 가능하다.
```kotlin
fun main() {
    val name = "fistkim101"
    println("${name}, hello" )
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=49296:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
fistkim101, hello

Process finished with exit code 0
```

<br>

아래와 같이 함수도 참조가 가능하며 return type 이 String 이 아니라도 알아서 형변환을 해준다.
```kotlin
fun main() {
    val name = "fistkim101"
    println("${name()}, hello" )
}

fun name() = "fistkim101-function"
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=49340:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
fistkim101-function, hello

Process finished with exit code 0
```

<br>

**<red>재미있는 것은 책에서 단일 변수를 참조한다고 해도 중괄호를 써주는 습관을 들이라고 권하고 있다는 것이다.
이유는 사람이 읽을때 중괄호를 쓴 경우가 더 가독성이 높다는 것인데 맞는 말이라는 생각이 들었다.</red>**

그럼 개발을 할때 애초에 $name 처럼 중괄호를 쓰지않고 단일변수만 쓰는 것을 아예 금지하는 것도 좋았을 것 같은데,
본인들 철학에 위배되지만 개발 자유도를 위해서 탐탁치 않은데 개발을 한건가..? 이런 생각이 들었다.

아무튼 문자열 템플릿을 쓸 경우 무조건 중괄호를 쳐야겠다는 생각을 했다.

<br>

## 2.2 클래스와 프로퍼티
### 2.2.1 프로퍼티
코틀린에서의 프로퍼티는 자바에서의 필드와 접근자가 합쳐진 개념이다. 실제로 프로퍼티라는 개념은 코틀린에서 뿐만이 아니라 다른 곳에서도 통용된다고 한다.
왜나는 처음 듣는가.. 아무튼 내가 줄곧 사용해온 필드라는 단어는 실제로 해당 클래스에 선언된 필드일 뿐이고 이것이 프로퍼티라고는 할 수 없다.

확실한 이해를 위해서 책의 용어에 따라 다시 용어를 정리해보자.

* 필드 - 클래스에 선언된 변수
* 접근자 - getter, setter와 같이 필드에 접근하게 해주는 메소드
* 프로퍼티 - 필드 + 접근자

코틀린에서는 자바에서 관습적으로 작성해줘야 했던 코드들을 기본적으로 제공해준다. 책에서 사용된 예시는 아래와 같다.
```kotlin
fun main() {
    val fistkim = Person("jk", false)
    println(fistkim.name)
    println(fistkim.isMarried)

    // fistkim.name = "jk2" <-- val 로 선언해줬기 때문에 setter가 구현되지 않아서 에러
    fistkim.isMarried = true
    println(fistkim.isMarried)
}

class Person(val name: String, var isMarried: Boolean)
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=49469:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
jk
false
true

Process finished with exit code 0
```

**<red>위 코드를 용어를 써서 표현하자면 Person 이라는 클래스가 있고 이 클래스는 name 과 isMarried 라는 프로퍼티를 갖고 있다고 할 수 있다.
name 의 경우 val(immutable) 이기 때문에 private 필드와 getter 만 만들어지며,
isMarried 의 경우 var(mutable) 이기 때문에 private 필드와 getter, setter 가 만들어진다.
여기서 생성되는 private field 를 프로퍼티의 값을 저장하기 위한 필드라는 의미로 backing field 라고 지칭한다고 한다.</red>**

코드에서 알 수 있듯이 자바에서처럼 .setIsMarried(false) 가 아니라 .isMarried = false 처럼 재할당을 해주는 방식으로 값을 바꾼다.

재밌는 것은 코틀린-> 자바코드 생성시 프로퍼티의 네이밍에 따라서 자바코드의 이름에 예외가 생기는 경우가 있는데 프로퍼티의 이름이 is로 시작되는 경우이다.
이 때에는 getter 에는 get 이 붙지 않고 프로퍼티 이름을 그대로 사용해야하며, setter 역시 is 를 set 으로 바꾼 이름을 써야 한다고 한다.
이건 근데 코틀린과 자바를 혼용해서 쓰는 프로젝트에서 신경써야할 이슈이므로 일단은 '이런게 있구나' 라고만 알고 넘어가면 될 것 같다.

<br>

### 2.2.2 커스텀 접근자
선언한 프로퍼티의 커스텀한 접근자를 만들 수 있다.

```kotlin
fun main() {
    val longRectangle = Rectangle(20, 10)
    println(longRectangle.isSquare)

    longRectangle.width = 20
    println(longRectangle.isSquare)
}

class Rectangle(var height: Int, var width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=51349:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
false
true

Process finished with exit code 0
```

<br>

[공식문서](https://kotlinlang.org/docs/properties.html#getters-and-setters) 에서도 getter, setter 만드는 방식에 대해서 언급하고 있었다.
문서에서 안내해주고 있는 syntax 는 아래와 같다.
```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

<br>

하지만 프로퍼티로 선언해주지 않고 아래와 같이 파라미터가 없는 함수로 이를 구현할 수도 있다.
```kotlin
class Rectangle(var height: Int, var width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }

    fun isSquare() = height == width // 실제로는 getter와 이름이 같아서 에러가 난다
}
```

<br>

이 부분에 관해서 책애서 따로 언급을 하고 있는데, 파라미터가 없는 함수를 정의하는 방식과 커스텀 getter 를 정의하는 방식 모두
성능상에서는 차이가 없어서 가독성을 기준으로 구현 방식을 선택하면 된다고 말하고 있었다.

**<red>하지만 일반적으로 클래스의 '특성'을 표현하기 위한 데이터라면 커스텀 getter 를 구현해주는 것이 함수로 구현해주는 것 보다 조금 더
일반적인 클래스-프로퍼티 구조에 가까운 것 같고 책에서도 이러한 논리로 커스텀 getter 를 추천하고 있다.</red>**

<br>

### 2.2.3 코틀린 소스코드 구조: 디렉터리와 패키지
자바랑 크게 다른 점이 두 가지 있었다.

**1) 함수를 파일의 최상위 수준으로 정의할 수 있다 보니(클래스에 속할 필요가 없다보니) import 로 다른 파일의 함수를 바로 사용이 가능하다.**

말그대로 코틀린에서는 함수가 최상위 수준의 정의가 가능하므로 특정 클래스에 속할 필요가 없어서 import 로 해당 파일의 전체 혹은 사용하고자 하는
선언만 설정해주면 사용이 가능하다. 자바에서는 클래스가 최상위 수준이고 필드나 메소드가 모두 클래스에 속했어야 했기 때문에 import 를 해도
해당 객체를 만들어서 그 객체의 public 한 자원만 사용이 가능했던 것과 비교하면 대조적이었다.

**2) package 키워드로 선언해준 경로와 실제 디렉토리 경로가 일치할 필요가 없다.**

아무래도 개발 유연성을 주기 위해서 실제 디렉토리 경로와 반드시 일치할 필요는 없도록 해줬다고 생각되었다.
하지만 아쉽게도 100% 코틀린 프로젝트가 아닌이상 자바와 혼용되어 사용될 것이고 상호 운용성을 위해서 자바의 방식을 따라가 줘야하는 것 같고,
책에서도 이러한 맥락에서 자바와의 상호운용성을 위해서 자바의 방식을 따르라고 권고되어 있다.

실습을 하면서 실제로 존재하지 않는 가상의 디렉토리인 vo 를 package 키워드를 통해서 설정해주고
이를 import 해서 사용했는데 오류없이 잘 작동하는 것을 확인해보았다.

```kotlin
package shape.vo

import java.util.Random

class Rectangle(val height: Int, val width: Int)

class Circle(val height: Int, val width: Int)

fun createRandomRectangle(): Rectangle {
    val random = Random()
    return Rectangle(random.nextInt(10), random.nextInt(10))
}

fun createRandomCircle(): Circle {
    val random = Random()
    return Circle(random.nextInt(10), random.nextInt(10))
}
```
```kotlin
import shape.vo.*

fun main() {
    val randomRectangle = createRandomRectangle()
    println(randomRectangle.height)

    val randomCircle = createRandomCircle()
    println(randomCircle.height)
}
```
{% include image.html src="/assets/images/kotlin/kotlin-package.png" text="vo 라는 디렉토리는 존재하지 않는다. 하지만 잘 작동함." %}

<br>

**<red>추가로 코틀린에서는 value object 에 대해서 자바에서 사용하는 준비 코드들을 기본적으로 제공하므로 각각의 클래스를 정의하는
코드의 양이 자바에 비해서 매우 작아지기 때문에 각각의 클래스를 각각의 파일로 만들어 주는 것은 비효율을 초래하므로 여러 클래스를
하나의 파일에 넣는 것을 주저하지 않기를 책에서 권고하고 있었다.</red>**

위 예제에서 Rectangle 과 Circle 를 자바에서 구현했다면 각각 하나의 파일이 되었을 텐데, 책의 권고에 따라서 하나의 파일에 모두 넣었다.

<br>

## 2.3 선택 표현과 처리: enum과 when
### 2.3.1 enum 클래스 정의
자바와 마찬가지로 상수 뿐만이 아니라 프로퍼티 및 메소드 선언이 가능하다. 기본 syntax는 [공식 문서](https://kotlinlang.org/docs/enum-classes.html) 에 있고
아래는 내가 만든 예제 코드이다. 코틀린에서 유일하게 세미콜론이 필수적으로 사용되는 곳이다.
```kotlin
package shape

enum class ShapeObject(var color: String) {
    RECTANGLE("white"), CIRCLE("white");
    // enum에 function 을 선언하기 위해서는 상수 목록과 function 사이에 세미콜론을 넣어야 한다.

    fun changeColor(color: String) {
        this.color = color
    }
}
```

<br>

### 2.3.2 when 으로 enum 클래스 다루기
when 이 여기서 나온 이유는 특별히 없었다. 그냥 enum 하고 엮어서 설명하기가 편해서 그렇게 구성된 것으로 보인다.
when 은 자바의 switch 와 비슷하지만 break 을 따로 걸어줄 필요가 없어서 버그가 발생할 확률을 낮췄다고 한다.

**<red>뿐만 아니라 하나의 분기조건 내에서 콤마(,) 를 통해서 여러 값을 조건으로 걸 수 있다.</red>**

```kotlin
package shape

enum class ShapeObject(var color: String) {
    RECTANGLE("white"), CIRCLE("black");

    val isMonochrome: Boolean
        get() = when (color) {
            "white", "black" -> true
            else -> false
        }

    fun changeColor(color: String) {
        this.color = color
    }
}
```
```kotlin
import shape.ShapeObject

fun main() {
    val whiteRectangle = ShapeObject.RECTANGLE
    println(whiteRectangle.isMonochrome) // true

    val blackCircle = ShapeObject.CIRCLE
    println(blackCircle.isMonochrome) // true

    whiteRectangle.changeColor("red")
    blackCircle.changeColor("red")
    println(whiteRectangle.isMonochrome) // false
    println(blackCircle.isMonochrome) // false
}
```

ShapeObject 에서 when 이 value 를 만들어내는 expression 이기 때문에 get() = when~ 처럼 블록을 정해주지 않고 바로 expression 을 사용할 수 있었다.

<br>

### 2.3.3 when 과 임의의 객체를 함께 사용
책에서 강조하고 있는 switch 와 비교한 when 의 우월성은 break 를 걸 필요가 없다는 것과, 분기 조건에 expression 을 넣을 수 있다는 것이다.
2.3.3 에서는 when 의 분기 조건에 expression 을 넣음으로써 코드가 얼마나 간결해질 수 있는가를 확인할 수 있었다.

```kotlin
package shape

import java.lang.Exception

enum class Color {
    RED, YELLOW, BLUE, GREEN, VIOLET, INDIGO, ORANGE
}

fun mix(c1: Color, c2: Color) = when (setOf(c1, c2)) {
    setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
    setOf(Color.YELLOW, Color.BLUE) -> Color.GREEN
    setOf(Color.BLUE, Color.VIOLET) -> Color.INDIGO
    else -> throw Exception("Dirty color")
}
```
```kotlin
import shape.Color
import shape.mix

fun main() {
    println(mix(Color.RED, Color.YELLOW)) // ORANGE
    println(mix(Color.YELLOW, Color.BLUE)) // GREEN
    println(mix(Color.BLUE, Color.VIOLET)) // INDIGO
    println(mix(Color.BLUE, Color.BLUE)) // throw Exception("Dirty color)
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=52353:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
ORANGE
GREEN
INDIGO
Exception in thread "main" java.lang.Exception: Dirty color
	at shape.ColorKt.mix(Color.kt:13)
	at MainKt.main(Main.kt:8)
	at MainKt.main(Main.kt)

Process finished with exit code 1
```

<br>

### 2.3.4 인자 없는 when 사용
when 은 파라미터가 없이도 사용이 가능하다는 것을 학습했고, 그 예제로 위 '2.3.3 when 과 임의의 객체를 함께 사용' 에서 사용한 예제의 리팩토링을 했다.

```kotlin
fun mix(c1: Color, c2: Color) = when (setOf(c1, c2)) {
    setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
    setOf(Color.YELLOW, Color.BLUE) -> Color.GREEN
    setOf(Color.BLUE, Color.VIOLET) -> Color.INDIGO
    else -> throw Exception("Dirty color")
}
```
```kotlin
fun mix(c1: Color, c2: Color) = when {
    (c1 == Color.RED && c2 == Color.YELLOW ||
            c1 == Color.YELLOW && c2 == Color.RED) -> Color.ORANGE
    (c1 == Color.YELLOW && c2 == Color.BLUE ||
            c1 == Color.BLUE && c2 == Color.YELLOW) -> Color.GREEN
    (c1 == Color.BLUE && c2 == Color.VIOLET ||
            c1 == Color.VIOLET && c2 == Color.BLUE) -> Color.INDIGO
    else -> throw Exception("Dirty color")
}
```

가독성은 떨어지지만 불필요하게 setOf 객체를 생성하고 GC의 대상을 늘리는 것 보다 효율 측면에서는 더 좋게 개선이 되었다.
개선을 하는게 학습 포인트가 아니고 'when expression 은 인자 없이 사용이 가능하다' 는 것을 숙지하는게 학습 포인트다.

<br>

### 2.3.5 스마트 캐스트: 타입 검사와 타입 캐스트를 조합
코틀린에서는 타입 검사와 함께 타입 캐스트를 동시에 해결하는 스마트 캐스트를 지원한다. 특정 타입인지 검사를 했다면,
검사 조건을 거친 블록 내부에서는 마치 그 타입인 것처럼 사용할 수 있는 것이다. 내부적으로 컴파일러가 캐스팅을 해주기 때문에 가능한 원리이다.

책에서 사용된 예제를 그대로 실습했다.
```kotlin
import java.lang.IllegalArgumentException

fun main() {
    val result = evaluation(Sum(Sum(Num(1), Num(2)), Num(4)))
    println(result)
}

interface Expression
class Num(val value: Int) : Expression
class Sum(val left: Expression, val right: Expression) : Expression

fun evaluation(expression: Expression): Int {
    if (expression is Num) {
        return expression.value
    }

    if (expression is Sum) {
        return evaluation(expression.left) + evaluation(expression.right)
    }

    throw IllegalArgumentException()
}
```

evaluation 함수에서 타입 검사를 거친 뒤 해당 타입으로 따로 캐스팅을 해주지 않고 바로 해당 타입이라 간주하고 코드를 작성할 수 있었다.
자바였다면 타입 검사가 true 라고 가정된 조건문 내부 이지만 따로 캐스팅을 해줬어야 했다는 것을 감안하면
개발 편의가 무척 좋아졌다고 말할 수 있다.

**<red>이렇게 코틀린에서는 is 를 통해서 타입 검사를 거쳤다면 변수를 원하는 타입으로 다시 캐스팅 해주지 않아도 마치 처음부터 그 변수가
원하는 타입으로 선언된 것처럼 사용할 수 있는데 이를 스카트 캐스트라고 부르며, 컴파일러가 알아서 캐스팅을 수행해주는 원리에 의해 동작한다.</red>**

예제 코드에서 보면 Num 타입 검사를 거쳤기에 바로 .value를 사용할 수 있었고, Sum 타입 검사를 거쳤기에 바로 .left 와 .right 을
사용할 수 있었던 것이 스마트 캐스트가 적용된 것이라고 할 수 있다.

또 한가지 중요한 점은 스마트 캐스트는 is 를 통해 검사하고 난 시점과 캐스팅 되었다고 가정하고 해당 변수를 사용하는 시점 사이에
그 값이 변할 가능성이 없는 경우만 작동한다. 이에 관해서는 [공식 문서](https://kotlinlang.org/docs/typecasts.html#smart-casts) 를 참고하면 된다.

<br>

### 2.3.6 리팩토링: if를 when 으로 변경
if 가 값을 만들어내는 expression 이라는 것을 고려하면 아래와 같이 리팩토링도 가능하다.
```kotlin
fun evaluation(expression: Expression): Int =  // <- expression을 바로 대입
    if (expression is Num) {
        expression.value
    } else if (expression is Sum) {
        evaluation(expression.left) + evaluation(expression.right)
    } else {
        throw IllegalArgumentException()
    }
```

<br>

when 내에서의 조건 식에서도 스마트 캐스트가 이뤄질 수 있으며, when 역시 값을 만드는 expression 이므로 아래와 같은 코드도 가능하다. 제일 깔끔해보인다.
```kotlin
fun evaluation(expression: Expression): Int =
    when (expression) {
        is Num -> expression.value
        is Sum -> evaluation(expression.left) + evaluation(expression.right)
        else -> throw IllegalArgumentException()
    }
```

<br>

### 2.3.7 if와 when의 분기에서 블록 사용
expression 에서 block 을 사용할 경우 block 의 마지막이 expression 의 결과인 것을 감안하면, 위의 '2.3.6 리팩토링: if를 when 으로 변경'과 같은 코드에서
로그를 남기고 싶을 때 마지막 expression 전에 처리를 해줘야한다.
```kotlin
fun evaluation(expression: Expression): Int =
    when (expression) {
        is Num -> {
            println("Num processing, value: ${expression.value}")
            expression.value
        }
        is Sum -> {
            println("Sum processing, left: ${evaluation(expression.left)} || right: ${evaluation(expression.right)}")
            evaluation(expression.left) + evaluation(expression.right)
        }
        else -> throw IllegalArgumentException()
    }
```

expression 의 내부에서 block 을 사용했을 경우에 한정한 설명이므로 함수에는 적용되지 않는 부분이다. 헷갈리지 않도록 하자.

<br>

## 2.4 대상을 이터레이션: while 과 for 루프
### 2.4.1 while 루프
자바와 다른게 1도 없다.

<br>

### 2.4.2 수에 대한 이터레이션: 범위와 수열
자바에서 for 문을 처음 배울 때 사용되는 for(int i = 0; i < 10; i++){ } 형태의 for 문이 코틀린에서는 존재하지 않는다.

**<red>코틀린에서는 위 for 문에서 선언하고 있는 초깃값, 증가 값, 최종 값을 명시하기 위해서 수열(progression) 개념을 사용한다.</red>**

수열을 사용하되 끝값을 포함 시킬지 안시킬지 경우에 따라 .. 혹은 until이 사용되며 증가 값으로는 step 을 사용하게 된다.
이 부분에 관한 구체적인 사항은 3장에서 다시 다룬다고 하니 일단은 쓰임만 간략히 실습하고 넘어갔다.

책에서 예제로 든 코드는 피즈버즈라는 게임을 구현한 코드였다. 특정 값을 순차적으로 올리면서 3으로 나눠지면 피즈, 5로 나눠지면 버즈
3과 5 모두로 나눠지면 피즈버즈 라고 출력하는 게임이다.
```kotlin
fun main() {
    for (i in 1..100) {
        fizzBuzz(i)
    }
}

fun fizzBuzz(index: Int) {
    when {
        index % 15 == 0 -> print(" FIZZBUZZ ")
        index % 3 == 0 -> print(" FIZZ ")
        index % 5 == 0 -> print(" BUZZ")
        else -> print(" ${index} ")
    }
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=55113:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
 1  2  FIZZ  4  BUZZ FIZZ  7  8  FIZZ  BUZZ 11  FIZZ  13  14  FIZZBUZZ  16  17  FIZZ  19  BUZZ FIZZ  22  23  FIZZ  BUZZ 26  FIZZ  28  29  FIZZBUZZ  31  32  FIZZ  34  BUZZ FIZZ  37  38  FIZZ  BUZZ 41  FIZZ  43  44  FIZZBUZZ  46  47  FIZZ  49  BUZZ FIZZ  52  53  FIZZ  BUZZ 56  FIZZ  58  59  FIZZBUZZ  61  62  FIZZ  64  BUZZ FIZZ  67  68  FIZZ  BUZZ 71  FIZZ  73  74  FIZZBUZZ  76  77  FIZZ  79  BUZZ FIZZ  82  83  FIZZ  BUZZ 86  FIZZ  88  89  FIZZBUZZ  91  92  FIZZ  94  BUZZ FIZZ  97  98  FIZZ  BUZZ
Process finished with exit code 0
```

<br>

역방향 수열은 아래와 같이 만든다.
```kotlin
fun main() {
    for (i in 100 downTo 1) {
        print(" ${i} ")
    }
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=55138:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
 100  99  98  97  96  95  94  93  92  91  90  89  88  87  86  85  84  83  82  81  80  79  78  77  76  75  74  73  72  71  70  69  68  67  66  65  64  63  62  61  60  59  58  57  56  55  54  53  52  51  50  49  48  47  46  45  44  43  42  41  40  39  38  37  36  35  34  33  32  31  30  29  28  27  26  25  24  23  22  21  20  19  18  17  16  15  14  13  12  11  10  9  8  7  6  5  4  3  2  1
Process finished with exit code 0
```

<br>

역방향에서든 순방향에서든 범위에서 step 키워드를 통해서 증가값 컨트롤이 가능하다.
예전에 basic syntax 를 알아볼때 조금 다뤘지만 step 을 특별히 명시하지 않으면 항상 step 1 이 생략되어 있다고 보면 이해하기가 편한 것 같다.
```kotlin
fun main() {
    for (i in 100 downTo 1 step 2) {
        print(" ${i} ")
    }
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=55244:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
 100  98  96  94  92  90  88  86  84  82  80  78  76  74  72  70  68  66  64  62  60  58  56  54  52  50  48  46  44  42  40  38  36  34  32  30  28  26  24  22  20  18  16  14  12  10  8  6  4  2
Process finished with exit code 0
```

<br>

끝 값을 포함하지 않으려면 until 을 사용한다.
```kotlin
fun main() {
    for (i in 1 until 10) {
        print(" ${i} ")
    }
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=55253:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
 1  2  3  4  5  6  7  8  9
Process finished with exit code 0
```

<br>

### 2.4.3 맵에 대한 이터레이션
이 소챕터의 학습 포인트는 '코틀린에서는 컬렉션 이터레이션을 편리하게 해주는 구조 분해 문법을 제공한다' 였다. 제목은 맵에 대한 이터레이션이지만 사실 컬렉션들에 대해서 따로
알아볼 때 더 자세히 학습해야할 것들이었고 이번엔 대략적으로 '이런 것들을 제공해준다' 만 인지하면 충분한 것 같다.

예제로 나온 코드를 그대로 실습해보았다.
```kotlin
import java.util.*

fun main() {

    val binaryReps = TreeMap<Char, String>()

    for (char in 'A'..'F') {
        val binary = Integer.toBinaryString(char.code)
        binaryReps[char] = binary
    }

    for ((letter, binary) in binaryReps) {
        println("${letter} = ${binary}")
    }

}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=55875:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
A = 1000001
B = 1000010
C = 1000011
D = 1000100
E = 1000101
F = 1000110

Process finished with exit code 0
```

자바에서 .entrySet 을 이용해서 key 와 value 를 이터레이션 시킬 수 있었는데, 이걸 더 간편하게 제공해줘서 좋았다.
TreeMap 이 구체적으로 어떻게 동작하는지, 구조 분해 문법이 내부적으로 어떻게 동작하는지에 대해서는 여기서는 깊게 알아보지 않겠다.

맵에 대한 이터레이션보다 사실 더 반가웠던(?) 것은 리스트에서 인덱스를 같이 제공하는 것이었다.
```kotlin
fun main() {

    val list = arrayListOf("10", "11", "12")
    for ((index, element) in list.withIndex()) {
        println("${index} = ${element}")
    }

}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=55894:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
0 = 10
1 = 11
2 = 12

Process finished with exit code 0
```

이건 정말 편리한 거다. 자바에서 인덱스가 필요할 때 인덱스로 사용한 변수를 따로 선언해주고 이를 루프 안에서 한번의 회전마다 ++ 를 해줬어야 했는데,
코틀린에서는 구조 분해 구문 덕분에 인덱스를 바로 뽑아 쓸 수 있다. withIndex 에 대해서도 3장에서 자세히 다룬다고 하니 여기까지만 정리하고 넘어간다.

<br>

### 2.4.4 in 으로 컬렉션이나 범위의 원소 검사
```kotlin
fun main() {

    val fruits = arrayListOf("banana", "orange", "apple")
    val isBananaExist = "banana" in fruits // true
    val isAppleExist = "apple" !in fruits // false
    val isGrapeExist = "grape" in fruits // false

    println(isBananaExist)
    println(isAppleExist)
    println(isGrapeExist)

}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=55926:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
true
false
false

Process finished with exit code 0
```

<br>

여기서 in 에 대해서 알아보기 위해서 레퍼런스를 따라가보면 아래 자바 코드가 나온다.
```java
    /**
     * Returns {@code true} if this list contains the specified element.
     * More formally, returns {@code true} if and only if this list contains
     * at least one element {@code e} such that
     * {@code Objects.equals(o, e)}.
     *
     * @param o element whose presence in this list is to be tested
     * @return {@code true} if this list contains the specified element
     */
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```

<br>

여기서 indexOf 를 추적해서 찾아보면 아래와 같은 코드가 나온다.
```java

    /**
     * Returns the index of the first occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the lowest index {@code i} such that
     * {@code Objects.equals(o, get(i))},
     * or -1 if there is no such index.
     */
    public int indexOf(Object o) {
        return indexOfRange(o, 0, size);
    }

    int indexOfRange(Object o, int start, int end) {
        Object[] es = elementData;
        if (o == null) {
            for (int i = start; i < end; i++) {
                if (es[i] == null) {
                    return i;
                }
            }
        } else {
            for (int i = start; i < end; i++) {
                if (o.equals(es[i])) {
                    return i;
                }
            }
        }
        return -1;
    }
```

<br>

**<blue>개인적으로 결국 핵심은 o.equals(es[i]) 에 대한 이해가 in 에 대한 이해라고 생각한다.</blue>**
사실 이건 새로운 것도 아니고 굉장히 일반적인 부분인데 hashcode 의 비교를 통해서 포함여부를 판단해주는 것이다.

**<red>결국 요약하자면 리스트에서의 in은 순차탐색(index 를 순차적으로 증가)을 하면서 hashcode 의 비교를 통해서
포함 여부를 찾아주는 원리에 의해서 동작하는 키워드인 것이다.</red>**

<br>

## 2.5 코틀린의 예외 처리
이번 소챕터는 선행 학습이 필요한 개념이 있었다. 자바의 checked exception 과 unchecked exception 에 대한 이해가 선행적으로 필요했다.
**<red>둘의 차이를 간단히 말하자면 '명시적으로 에러 처리(check)를 강요(안하면 컴파일에서부터 에러가 난다)' 당하면 checked exception 이고
그렇지 않다면 unchecked exception 라고 할 수 있다.</red>**

{% include image.html src="/assets/images/kotlin/exception-type-1.png" %}
{% include image.html src="/assets/images/kotlin/exception-type-2.png" %}

**<red>주의할 것은 runtime exception 의 상속 여부만으로 이를 구분지으면 error 가 포함이 되지 않는다는 것이다.
따라서 명시적으로 에러처리를 해주지 않으면 컴파일 에러가 나는 것이 checked exception 이라고 할 수 있으며,
반대로 명시적으로 에러처리를 해주지 않아도 컴파일 에러가 나지 않는 것이 unchecked exception 라고 할 수 있다.
대표적으로 runtime exception 을 상속하는 모든 exception 들이 unchecked exception 이라고 할 수 있다.</red>**
상세히 각각 알아보자.

**<red>checked exception 은 반드시 명시적으로 check 가 필요한 exception 이다.</red>**
그렇기에 컴파일 시점부터 특정 코드가 checked exception 을 유발할 가능성이 있고, 명시적으로 이를 처리하지 않았다면 컴파일 에러가 나게 된다.
그래서 명시적으로 에러를 처리하게 되고 이는 곧 checked 된 exception 이므로 checked 라는 과거형이 나오지 않았나 싶다.
이 에러는 예외가 발생했을 시 기본으로 롤백이 발생되지 않는다.

**<red>unchecked exception 도 같은 맥락에서 풀어보면 check 가 되지 않은 exception 이다. 즉, 명시적으로 처리하지 않아도 컴파일에 문제가 없고
run 에 문제가 없기 때문에 check 가 되지 않은 것이다.</red>** unchecked exception, 기본적으로 롤백을 발생시킨다.

<br>

### 2.5.1 try, catch, finally
이 소챕터에서는 코틀린에서 try, catch, finally 를 어떻게 사용하는지 간단한 예제를 살펴보았다.
```kotlin
import java.io.BufferedReader
import java.io.StringReader
import java.lang.NumberFormatException

fun main() {
    val reader = BufferedReader(StringReader("239"))
    println(readNumber(reader))
}

fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        return null
    } finally {
        reader.close()
    }
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=57945:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
239

Process finished with exit code 0
```

<br>

사실 이 소챕터의 학습 포인트는 try, catch, finally 의 사용을 위한 기본 syntax 가 아니었다.
**<red>코틀린에서는 checked exception 과 unchecked exception 을 구분하지 않는다는 것이 학습 포인트였다.
이 말인즉 반드시 예외처리를 하도록 강요당하는(하지 않으면 컴파일 에러를 발생시키는) exception 이 없다는 뜻이다.</red>**

**<blue>여기 깔린 철학은 '예외 처리를 강제 당해서 처리 구문을 작성한다고 해도 유의미한 처리를 하기 보다는 그냥 무시하게 되거나 의미 없이
예외를 발생시키는 코드가 나오더라' 라는 생각이다. 그래서 강제를 시켜봤자 실제로 오류를 제대로 방지하지 못하기 때문에 그냥
강제를 하지 않겠다는 의도이다.</blue>**

위 예제 코드에서도 .readLine()과 .close() 에서 자바에서는 checked exception 인 IOException 이 발생할 수 있어서 명시적인 처리가 강제되는데
코틀린에서는 이를 명시적으로 다루지 않아줘도 된다. 위 맥락에서 해석하자면 'IOException 나봤자 네가 의미있게 뭔갈 조치할 수 있는건 없어'
라는 생각이 깔려있기에 애초에 강제하지도 않는 것이다.

실제로 이 코틀린 코드를 자바 코드로 변경해보면 아래와 같이 IDE 가 컴파일 에러가 나지 않도록 IOException 처리를 하라고 미리 알려준다.

{% include image.html src="/assets/images/kotlin/no-checked-exception.png" %}

<br>

### 2.5.2 try 를 식으로 사용
try도 expression 이므로 아래와 같이 리팩토링이 가능하다.
```kotlin
fun readNumber(reader: BufferedReader): Int? =
    try {
        val line = reader.readLine()
        Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        null
    } finally {
        reader.close()
    }
```

<br>
<br>
