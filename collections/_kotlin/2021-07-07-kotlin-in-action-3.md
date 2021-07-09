---
layout: article
title: (Kotlin In Action) 3장 함수 정의와 호출
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

* 3.1 코틀린에서 컬렉션 만들기
* 3.2 함수를 호출하기 쉽게 만들기
    * 3.2.1 이름 붙인 인자
    * 3.2.2 디폴트 파라미터
    * 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티
* 3.3 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
    * 3.3.1 임포트와 확장 함수
    * 3.3.2 자바에서 확장 함수 호출
    * 3.3.3 확장 함수로 유틸리티 함수 정의
    * 3.3.4 확장 함수는 오버라이드할 수 없다
    * 3.3.5 확장 프로퍼티
* 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원
    * 3.4.1 자바 컬렉션 API 확장
    * 3.4.2 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의
    * 3.4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언
* 3.5 문자열과 정규식 다루기
    * 3.5.1 문자열 나누기
    * 3.5.2 정규식과 3중 따옴표로 묶은 문자열
    * 3.5.3 여러 줄 3중 따옴표 문자열
* 3.6 코드 다듬기: 로컬 함수와 확장
* 3.7 요약

<br>

# 학습내용

<br>

## 3.1 코틀린에서 컬렉션 만들기
책에서 거듭 강조하고 여러번 등장하는 말 중 하나는 '코틀린은 자바와 상호운용성이 좋아서 자바를 점진적으로 전환하거나 혼용하기에 매우 좋으면서도 기능은 자바보다 강력하다' 는 것이다.
'코틀린에서 컬렉션 만들기' 를 통해 책에서 전달하고자 하는 것을 포인트로 뽑아보면 아래와 같다.

<br>

**1) 코틀린에서는 새로운 컬렉션을 만들지 않았고 자바의 컬렉션을 그대로 사용했다.** <br>
```kotlin
fun main() {
    val set = hashSetOf(1, 2, 3)
    val list = arrayListOf(1, 2, 3)
    val map = hashMapOf(1 to "one", 2 to "two", 3 to "three")

    println(set.javaClass)
    println(list.javaClass)
    println(map.javaClass)
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=59474:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
class java.util.HashSet
class java.util.ArrayList
class java.util.HashMap

Process finished with exit code 0
```

.javaClass 를 이용해 확인해본 결과 모두 자바의 컬렉션을 그대로 사용하고 있음을 알 수 있다.
**<blue>책에서는 그 이유로 자바의 컬렉션을 그대로 사용해야 상호운용성이 더 좋기 때문이라는 말을 하고 있었다.</blue>**
사실 더 정확하게는 코틀린을 개발하면서 자바와의 상호운용성을 지키기 위해서는 자바의 컬렉션을 그대로 사용하는게 최선이기 때문이 아니었을까 싶다.

<br>

**2) 자바의 컬렉션을 그대로 사용하지만 더 많은 기능을 제공한다.**
```kotlin
fun main() {
    val numbers = setOf(1, 2, 3)
    println(numbers.maxOrNull())
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=59496:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
3

Process finished with exit code 0
```
위와 같이 자바클래스에 없는 메소드를 코틀린이 정의하고 제공해주고 있다. 차후(3, 4장에 걸쳐서)에 이러한 것들이 어떻게 동작하는지 알아보게 된다.

<br>

## 3.2 함수를 호출하기 쉽게 만들기
이번 소챕터에서는 코틀린에서 '어떻게 함수의 호출을 쉽게 하는지' 를 알아보기 위해서 먼저 가독성이 떨어지고
호출이 불편한(?) 함수를 직접 만드는 실습을 먼저 하게 되었다.

<br>

```kotlin
fun main() {
    val list = listOf(1, 2, 3)
    println(list)
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=59846:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
[1, 2, 3]

Process finished with exit code 0
```

<br>

여기서 출력 결과를 (1; 2; 3;) 으로 출력하기 위한 함수를 만들었는데 아래와 같이 만들었다.
```kotlin
import java.lang.StringBuilder

fun main() {
    val list = listOf(1, 2, 3)
    println(joinToString(list, ";", "(", ")"))
}

fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String,
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=59868:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
(1;2;3)

Process finished with exit code 0
```
책에서는 이 예제가 '함수를 호출하는 문장이 번잡하다' 고 말하고 있다. 그래서 이걸 코틀린에서 제공하는 여러 기능을 사용하여 리팩토링하면서
소챕터의 제목인 '함수를 호출하기 쉽게 만들기' 를 학습하는 것이 이번 소챕터의 학습 목표라고 할 수 있다.

<br>

### 3.2.1 이름 붙인 인자
실습하면서도 느꼈지만 파라미터가 4개라서 가독성이 상당히 좋지 않다고 생각했다. IDE 의 도움을 받으면 몇 번째 인자가 무엇인지를 알 수는 있으나
개발을 하는데 IDE 의 도움을 아예 전제로 보는 것은 좀 아닌 것 같다. 극단적으로 vim 으로 개발한다고 생각하고 언어의 특성을 학습하는게 좋지 않을까 싶다.

**<blue>'이름 붙인 인자' 를 통해서 책에서 말하고 싶었던 것은 코틀린에서는 메소드 호출시 파라미터에 이름을 써줄 수 있다는 것이었다.
아래 코드와 같이 메소드를 선언해줄 때에 선언해준 파라미터 명을 호출하는 곳에서 그대로 사용해서 가독성을 높혀줄 수 있다.</blue>**
```kotlin
import java.lang.StringBuilder

fun main() {
    val list = listOf(1, 2, 3)
    println(joinToString(list, ";", "(", ")"))
    println(joinToString(collection = list, separator = ";", prefix = "(", postfix = ")"))
}

fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String,
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

책에서는 호출부에서 하나의 인자라도 이름을 써준 경우에 다른 모든 인자들에 대한 이름도 명시해주라고 말하고 있다.
어떤 것은 이름을 명시해주고 어떤 것은 해주지 않으면 코드를 보는 사람이 혼란스럽기 때문이다.

**<blue>아무튼 IDE의 도움이 없다고 가정한다면 특정 메소드의 호출 파라미터가 많다던가 했을 때 코드의 가독성을 높히기 위해 사용할 수 있는 좋은 기능이라는
생각이 들었다. 혹은 파라미터 수가 적다고 해도 팀원들과 협의해서 팀의 컨벤션으로 사용해도 좋지 않을까 싶었다.</blue>**
단, 유지보수를 할때는 바꿔줘야할 포인트가 많아지긴 하지만 인텔리J 의 refactor 기능을 사용하면 한번에 다 바꿔주기 때문에 충분히 사용할 수는 있을 것이다.

**<blue>문제는 자바와의 호환 문제인데 안타깝게도 자바에서 만들어진 코드를 호출 할 때에는 호출부에 인자의 이름을 붙일 수 없다.</blue>**

<br>

그런데 이걸 하다가 매우 충격적이었던 것은 자바에서도 이런 비슷한 것이 가능했었다는 것이다.
책에서 '자바도 이런 비슷한게 가능하긴 하다' 라는 맥락에서 설명해주는 부분이 있었는데 나는 처음보는 것이었다.
```java
student.holdWritingInstrument(new Pen(/* color */"red"));
```
```java
    public Pen(String color) {
        this.color = color;
    }
```

위와 같이 /* */ 를 이용해서 호출부에 내가 원하는 문구를 쓸 수 있었는데, 이런 것이 가능한지 이번에 처음 알았다.
하지만 이건 휴먼에러를 발생 시킬 수 있기도 하고(파라미터 명을 이상하게 쓸 수 있으니까) 주석과도 문법이 같으니
코드가 여러모로 깔끔하지 못하게 느껴진다.

<br>

### 3.2.2 디폴트 파라미터
**<blue>코틀린에서는 함수를 선언할 때 파라미터의 디폴트 값을 지정해줄 수 있다. 이것이 주는 이점은 디폴트 값이 있는 경우
함수를 호출하는 곳에서 그 파라미터를 신경쓰지 않아도 되기에 가독성이 올라간다는 것이다.</blue>**

```kotlin
import java.lang.StringBuilder

fun main() {
    val list = listOf(1, 2, 3)
    println(joinToString(list))
}

fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=60151:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
1, 2, 3

Process finished with exit code 0
```

<br>

만약 선언부에서 특정한 파라미터 하나만 디폴트 값이 지정되지 않았다면,
'3.2.1 이름 붙인 인자' 에서 다룬 것처럼 아래와 같이 호출부에서 인자의 명을 밝혀주면 된다.
```kotlin
import java.lang.StringBuilder

fun main() {
    val list = listOf(1, 2, 3)
    println(joinToString(list, prefix = ""))
}

fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String,
    postfix: String = "",
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

<br>

**<red>자바에서는 디폴트 파라미터라는 개념이 없기 때문에 코틀린의 선언부에서 아무리 디폴트 값을 지정해 준다 한들
자바에서 이를 호출 할 때에는 파라미터를 넣어줘야한다.</red>**


**<red>만약 자바에서도 이를 편하게 호출하고 싶다면 코틀린 코드에서 @JvmOverloads 라는 어노테이션을 붙여주어서 컴파일시 해당 메소드에 대해서
오버로딩된 여러 바이트코드들이 더 만들어지게 만든다.</red>**
```kotlin
@JvmOverloads
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

<br>

그러고 생성된 바이트코드를 살펴보면 아래와 같이 오버로딩된 코드들을 볼 수 있다.
```java
   @JvmOverloads
   @NotNull
   public static final String joinToString(@NotNull Collection collection, @NotNull String separator, @NotNull String prefix) {
      return joinToString$default(collection, separator, prefix, (String)null, 8, (Object)null);
   }

   @JvmOverloads
   @NotNull
   public static final String joinToString(@NotNull Collection collection, @NotNull String separator) {
      return joinToString$default(collection, separator, (String)null, (String)null, 12, (Object)null);
   }

   @JvmOverloads
   @NotNull
   public static final String joinToString(@NotNull Collection collection) {
      return joinToString$default(collection, (String)null, (String)null, (String)null, 14, (Object)null);
   }
```

<br>

각각의 오버로딩된 함수들은 시그니처에서 생략된 파라미터에 대해서 코틀리 함수의 디폴트 파라미터 값을 사용하게 된다고 하는데,
@JvmOverloads 어노테이션에 대해서는 나중에 추가적으로 학습하게 될때 자세히 다뤄봐야겠고 이번엔 그냥 넘어가도록 한다.

<br>

### 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티
이번 소챕터의 학습 포인트는 아래 두 가지 였다.

**<blue>1) 함수를 최상위 선언함으로써 누릴 수 있는 이점은 무엇인가</blue>**
**<blue>2) 코틀린에서는 프로퍼티, 함수를 클래스에 속하게 하지 않고 최상위로 선언할 수 있는데 이것의 동작원리가 무엇인가</blue>**<br>

<br>

#### 함수를 최상위 선언함으로써 누릴 수 있는 이점은 무엇인가
자바에서는 어떤 함수든 특정한 클래스 내에 속하도록 만들었어야해서, 단순한 기능적인 필요에 의한 함수들을 사용하기 위해서
굳이 의미없는 클래스를 만들어서 그 안에 함수들을 넣어줬어야만 했다. 가장 간단한 예로 Utils 류의 클래스들이 있다.
**<blue>코틀린에서는 함수를 최상위 선언할 수 있으므로 이러한 무의미한 클래스를 만들어주지 않아도 된다.</blue>**

자바에서처럼 util 성 클래스에 필요한 함수를 넣고 이를 사용하는 곳에서 클래스를 만들어서 클래스의 함수를 호출하는 것을
코드로 나타내면 아래와 같다.
```kotlin
package utils

import java.lang.StringBuilder

class StringUtils {

    fun <T> joinToString(
        collection: Collection<T>,
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
    ): String {
        val result = StringBuilder(prefix)
        for ((index, element) in collection.withIndex()) {
            if (index > 0) result.append(separator)
            result.append(element)
        }
        result.append(postfix)
        return result.toString()
    }

}
```
```kotlin
import utils.*

fun main() {
    val stringUtils = StringUtils()

    val list = listOf(1, 2, 3)
    println(stringUtils.joinToString(list))
}
```

<br>

StringUtils 라는 클래스는 객체지향적 사고에서 보면 상당히 불필요하고 맥락에 맞지 않는 클래스다.
무슨 의미냐하면 객체지향적인 사고로 보았을 때 '클래스' 라는 것은 그 클래스가 대표할 수 있는,
대표 해야할 주체(추상적인 것이든 현실과 매칭 되는 것이든)의 상태값과 이것의 행동을 갖고 있는 것이어야 하는데
도구적인 기능을 위한 util 류의 객체는 엄연히 말하면 객체지향적이지 않기 때문이다.

단순한 정적인 기능을 하는 만큼 특정 클래스에 속하지 않고 함수만으로 작동해줄 수 있으면
조금더 원칙을 준수한 개발이 가능할 수 있고 코틀린은 이를 지원한다.
코틀린에서 함수를 최상위로 선언할 수 있다는 것을 이용해서 코드를 수정하면 아래와 같아진다.
```kotlin
package utils

fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
```kotlin
import utils.joinToString

fun main() {
    val list = listOf(1, 2, 3)
    println(joinToString(list))
}
```
코드가 매우 깔끔해졌다. 쓸데없는 클래스를 만들고 클래스에서 이를 꺼내 쓰는 불필요한 코드가 없어졌기 때문이다.

<br>

#### 코틀린에서는 프로퍼티, 함수를 클래스에 속하게 하지 않고 최상위로 선언할 수 있는데 이것의 동작원리가 무엇인가

원칙적으로 JVM 은 클래스 안에 들어있는 코드만을 실행할 수 있다. 그래서 코틀린에서 클래스 없이 함수를 최상위 선언해주면
**<blue>컴파일을 할 때 컴파일러가 코틀린 소스 파일의 이름을 이용해서 클래스를 만들어주고 그 안에 public static 하게 메소드를 만들어주기 때문에
함수를 바로 사용할 수 있는 것이다.</blue>**

자바로 디컴파일을 해보면 아래와 같이 소스가 나오는데 파일명을 이용해서 StringUtilsKt 라는 클래스를 만들어줬고,
그 내부에 public static final String joinToString 라는 함수가 만들어진 것을 확인할 수 있다.
```kotlin
package utils;

import java.util.Collection;
import java.util.Iterator;
import kotlin.Metadata;
import kotlin.jvm.internal.Intrinsics;
import org.jetbrains.annotations.NotNull;

@Metadata(
   mv = {1, 5, 1},
   k = 2,
   d1 = {"\u0000\u0012\n\u0000\n\u0002\u0010\u000e\n\u0002\b\u0002\n\u0002\u0010\u001e\n\u0002\b\u0004\u001a8\u0010\u0000\u001a\u00020\u0001\"\u0004\b\u0000\u0010\u00022\f\u0010\u0003\u001a\b\u0012\u0004\u0012\u0002H\u00020\u00042\b\b\u0002\u0010\u0005\u001a\u00020\u00012\b\b\u0002\u0010\u0006\u001a\u00020\u00012\b\b\u0002\u0010\u0007\u001a\u00020\u0001¨\u0006\b"},
   d2 = {"joinToString", "", "T", "collection", "", "separator", "prefix", "postfix", "kotlin-in-action"}
)
public final class StringUtilsKt {
   @NotNull
   public static final String joinToString(@NotNull Collection collection, @NotNull String separator, @NotNull String prefix, @NotNull String postfix) {
      Intrinsics.checkNotNullParameter(collection, "collection");
      Intrinsics.checkNotNullParameter(separator, "separator");
      Intrinsics.checkNotNullParameter(prefix, "prefix");
      Intrinsics.checkNotNullParameter(postfix, "postfix");
      StringBuilder result = new StringBuilder(prefix);
      int index = 0;

      for(Iterator var7 = ((Iterable)collection).iterator(); var7.hasNext(); ++index) {
         Object element = var7.next();
         if (index > 0) {
            result.append(separator);
         }

         result.append(element);
      }

      result.append(postfix);
      String var10000 = result.toString();
      Intrinsics.checkNotNullExpressionValue(var10000, "result.toString()");
      return var10000;
   }

   // $FF: synthetic method
   public static String joinToString$default(Collection var0, String var1, String var2, String var3, int var4, Object var5) {
      if ((var4 & 2) != 0) {
         var1 = ", ";
      }

      if ((var4 & 4) != 0) {
         var2 = "";
      }

      if ((var4 & 8) != 0) {
         var3 = "";
      }

      return joinToString(var0, var1, var2, var3);
   }
}
```

<br>

**<blue>프로퍼티는 const 를 붙여줘야 getter, setter 가 생기지 않아서 자바에서의 접근자 사용을 원천적으로 차단한다.</blue>**
진정한 필드로만 만들어주고 싶고 getter와 setter가 만들어지는 것을 원하지 않는다면 const 를 붙여주도록 한다.
```kotlin
package utils

const val week = 7
const val name = "fistkim101"
```
```java
package utils;

import kotlin.Metadata;
import org.jetbrains.annotations.NotNull;

@Metadata(
   mv = {1, 5, 1},
   k = 2,
   d1 = {"\u0000\u000e\n\u0000\n\u0002\u0010\u000e\n\u0000\n\u0002\u0010\b\n\u0000\"\u000e\u0010\u0000\u001a\u00020\u0001X\u0086T¢\u0006\u0002\n\u0000\"\u000e\u0010\u0002\u001a\u00020\u0003X\u0086T¢\u0006\u0002\n\u0000¨\u0006\u0004"},
   d2 = {"name", "", "week", "", "kotlin-in-action"}
)
public final class ConstantsKt {
   public static final int week = 7;
   @NotNull
   public static final String name = "fistkim101";
}
```

<br>

const 를 안붙이면 아래와 같이 getter 가 생성된다.(val 가 아니라 var 로 선언해주면 setter 도 만들어진다.)
```java
package utils;

import kotlin.Metadata;
import org.jetbrains.annotations.NotNull;

@Metadata(
   mv = {1, 5, 1},
   k = 2,
   d1 = {"\u0000\u0012\n\u0000\n\u0002\u0010\u000e\n\u0002\b\u0003\n\u0002\u0010\b\n\u0002\b\u0003\"\u0014\u0010\u0000\u001a\u00020\u0001X\u0086D¢\u0006\b\n\u0000\u001a\u0004\b\u0002\u0010\u0003\"\u0014\u0010\u0004\u001a\u00020\u0005X\u0086D¢\u0006\b\n\u0000\u001a\u0004\b\u0006\u0010\u0007¨\u0006\b"},
   d2 = {"name", "", "getName", "()Ljava/lang/String;", "week", "", "getWeek", "()I", "kotlin-in-action"}
)
public final class ConstantsKt {
   private static final int week = 7;
   @NotNull
   private static final String name = "fistkim101";

   public static final int getWeek() {
      return week;
   }

   @NotNull
   public static final String getName() {
      return name;
   }
}
```

<br>

## 3.3 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
'확장함수'는 예전에 flutter를 사용할 때 dart 에서 처음 사용해봤던 개념이다.
**<blue>'확장함수' 란 기존에 존재하는 클래스의 외부에 함수를 선언해주지만 마치 그 함수가 클래스의 내부에서 선언된 것처럼
기능을 사용할 수 있도록 해주는 기능이다.</blue>** 결국 이 확장함수는 호출하는 측에서는 확장함수로 선언된 메소드인지
원래 내부에서 선언된 함수인지 알 수도 없고 알 필요도 없이 편하게 사용할 수 있게 된다.

아래는 코틀린의 [공식 문서](https://kotlinlang.org/docs/extensions.html) 에 나온 확장함수에 대한 설명의 일부이다.
> Kotlin provides an ability to extend a class with new functionality without having to inherit from the class or use design patterns such as Decorator.
> This is done via special declarations called extensions.

'확장함수' 에는 **<red>수신 객체 타입(receiver type)</red>** 과 **<red>수신 객체(receiver object)</red>** 라는 개념이 존재한다.
타입은 말 그대로 '확장 함수' 가 추가될 타입을 의미한다. 수신 객체란 실제로 확장함수가 적용되는 구체적인 객체를 의미한다. 아래의 실습코드를 보자.
여기서 String 이 '수신 객체 타입' 이며, "Kotlin" 이 '수신 객체' 라고 할 수 있다.
```kotlin
package extention

fun String.lastChar(): Char = this.get(this.length - 1)
```
```kotlin
import extention.lastChar

fun main() {
    val lastChar = "Kotlin".lastChar()
    println(lastChar)
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=62265:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
n

Process finished with exit code 0
```

<br>

확장함수를 선언할 때 아래와 같이 this 를 생략할 수 있긴 한데 개인적으로 this 를 명시해주는 것이 가독성이 더 좋은 것 같다.
이건 작업자간에 약속을 해야 하는 것 같은데 나는 개인적으로 this 를 살리는 쪽으로 의견을 표현해볼 것 같다.
```kotlin
fun String.lastChar(): Char = get(length - 1)
```

<br>

책에서 강조하고 있는 것 중 하나는 **<red>확장함수가 캡슐화를 깨트리지 않는다는 것이다. 확장함수가 마치 내부에서 선언된 것 처럼
함수를 사용할 수 있게 해주는 기능이라 할지라도 본래 내부에 private 으로 선언되었거나 protected 로 선언된 자원은 사용하지 못하기 때문이다.</red>**

<br>

### 3.3.1 임포트와 확장 함수
import 시 as 를 이용해서 원하는 키워드로 변경이 가능하다.
```kotlin
import extention.lastChar as last

fun main() {
    val lastChar = "Kotlin".last()
    println(lastChar)
}
```

<br>

### 3.3.2 자바에서 확장 함수 호출
코틀린에서 선언한 확장함수를 자바에서 사용하려면 public static 함수를 쓰듯이 사용하면 된다.
위 '코틀린에서는 프로퍼티, 함수를 클래스에 속하게 하지 않고 최상위로 선언할 수 있는데 이것의 동작원리가 무엇인가' 에서 알아본 대로
확장함수를 선언해준 소스 코드 파일명이 클래스명이 되면서 클래스가 생성되고 내부에 메소드가 만들어지므로,
자바에서 사용을 할 때도 그에 맞게 동일하게 사용해주면 된다. 아래와 같이 StringKt.lastChar("Kotlin") 이런 식으로 사용이 가능하다.
```java
public final class MainKt {
   public static final void main() {
      char lastChar = StringKt.lastChar("Kotlin");
      boolean var1 = false;
      System.out.println(lastChar);
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```

<br>

### 3.3.3 확장 함수로 유틸리티 함수 정의
실습했던 joinToString 함수를 확장 함수로 만들어서 편하게 사용할 수 있도록 리팩토링을 해보았다.
```kotlin
package extention

fun <T> Collection<T>.joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String = "",
    postfix: String = "",
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

fun Collection<String>.join(
    collection: Collection<String>,
    separator: String,
): String = joinToString(collection, separator)
```
```kotlin
import extention.join

fun main() {
    val list = arrayListOf("apple", "banana", "orange")
    println(list.join(list, ","))
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=65091:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
apple,banana,orange

Process finished with exit code 0
```
그냥 단순하게 확장 함수를 호출해도 되지만 확장함수 내에서도 이미 선언한 확장함수를 이용해서 원하는 확장함수를 만드는 방식으로 코딩해보았다.
위 방법대로 하면 확장함수 내에서도 마치 private 을 사용하듯이 할 수 있어서 코드가 조금 더 유연해진다.

<br>

### 3.3.4 확장 함수는 오버라이드할 수 없다
확장 함수는 오버라이드할 수 없다는 말의 의미는 **<red>'어떤 클래스에 대해서 이름이 같은 확장함수와 멤버 함수 둘이 병존할 때
호출의 우선순위는 멤버 함수가 갖고 있다'</red>** 는 의미이다.

원리를 생각해보면 이해가 더 쉽다. 확장 함수는 컴파일 단계에서 소스코드의 파일명을 사용한 클래스의 public static 함수가 되어서
인자로 '수신 객체'를 받아서 호출되는 구조인데, 만약 이름이 같은 동일한 메소드를 호출하게 된다해도 이런 호출 원리를 생각해보면
'수신 객체 타입'이 호출하는 메소드는 오버라이드 된 메소드가 아니고 확장함수로 만들어준 그 함수인 것이다.

코드를 보면서 이해해보자.
```kotlin
open class Fruit {
}

class Apple : Fruit() {
}
```
```kotlin
package extention

import Apple
import Fruit

fun Fruit.printName() = println("I am fruit")
fun Apple.printName() = println("I am apple")
```
일단 여기까지만 보자. Fruit 클래스가 있고 Apple 은 Fruit 클래스를 상속한다. 그리고 확장함수로 똑같은 이름의 printName 을 각각 만들어줬다.
**<red>그러면 Apple 이 Fruit 함수의 printName 를 오버라이드 하는 것 같지만, 이게 오버라이드 되지 않는다는 것이다.</red>**

<br>

```kotlin
import extention.*

fun main() {
    val fruit: Fruit = Apple()
    fruit.printName()
}
```
```bash
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=65241:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/Desktop/lab/servers/kotlin-in-action/build/classes/kotlin/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.20/636c5653641cd956de9aee5792155b07ea49825e/kotlin-stdlib-jdk8-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.5.20/218b60e1d446d1e0a18bc7aa8663634b136fbcc5/kotlin-stdlib-jdk7-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.5.20/9de35cc611bcecec8edce1d56d8e659953806751/kotlin-stdlib-1.5.20.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.5.20/3d79dbd48bf605f4aac1e7028981a1953e245cbb/kotlin-stdlib-common-1.5.20.jar MainKt
I am fruit

Process finished with exit code 0
```
보다시피 Apple() 로 할당해줬지만 결과는 fruit 가 나왔다. **<red>확장 함수를 호출할 때의 기준은 결국 '수신 객체 타입'이 무엇인지가 기준이기 때문에
확장함수를 선언할 때의 형식인 receiverType.methodName() ~ 에 따라서 receiverType 에 맞는 함수가 호출 되는 것이다.</red>** 변수에 할당된
객체의 타입은 확장 함수의 호출에 영향을 줄 수 없다.

<br>

### 3.3.5 확장 프로퍼티
확장 함수를 쓰듯이 프로퍼티를 클래스의 외부에서 선언해서 사용할 수 있다는 내용이 전부였다.

확장 함수는 기존 클래스의 외부에 사용하고자 하는 함수를 선언하여 무의미한 클래스의 생성을 막아주고,
기존 클래스의 멤버 함수로 넣기엔 중요도가 다소 떨어지는 것을 밖으로 뺄수 있도록 해주는 등
유연하고 간결한 설계에 이바지 한다고 보는데 확장 프로퍼티는 글쎄..

애초에 프로퍼티라는 것이 '이러한 항목을 이 클래스의 상태값으로 관리할 것이다' 라는 생각이 받침이 된 개념인데
이걸 밖에 선언한다? 좀 이상한 것 같다. 개인적으로 안쓰는게 낫지 않을까 하는 생각이 들었다.

<br>
<br>

