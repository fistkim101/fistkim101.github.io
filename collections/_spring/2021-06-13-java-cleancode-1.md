---
layout: article
title: 클린코드 문자열계산기
tags: 클린코드 객체지향생활체조 enum 문자열계산기
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
문자열 계산기는 nextstep에서 클린코드 수업에 들어가기 전에 단위 테스트에 익숙하지 않은 사람들을 위해서 만들어둔 튜토리얼 성격의 과정이라고 생각되었다.
처음 수업을 받은 이후로 꽤나 시간이 지났고, 그간 실무에서 테스트 코드를 많이 작성했던터라 이번에 미션을 수행하면서 익숙함을 많이 느낄 수 있어서 좋았다.
반면에 그 익숙함 만큼이나 성장의 폭은 적을 것이라는 생각도 들었다.

<br>
<br>

### 요구사항
* 사용자가 입력한 문자열 값에 따라 사칙연산을 수행할 수 있는 계산기를 구현해야 한다.
* 문자열 계산기는 사칙연산의 계산 우선순위가 아닌 입력 값에 따라 계산 순서가 결정된다. 즉, 수학에서는 곱셈, 나눗셈이 덧셈, 뺄셈 보다 먼저 계산해야 하지만 이를 무시한다.
* 예를 들어 "2 + 3 * 4 / 2"와 같은 문자열을 입력할 경우 2 + 3 * 4 / 2 실행 결과인 10을 출력해야 한다.

<br>
<br>

### 학습목표
* JUnit 을 활용한 단위 테스트 적용

<br>
<br>

### [실습코드](https://github.com/fistkim101/java-baseball-playground)

요구사항을 보면 결국 '약속된 사칙연산(문자열)에 대해서 전, 후의 인자를 계산해주고 그것을 연속으로 처리' 해주면 될 것으로 판단된다.
그리고 사칙연산(문자열)에 따른 연산을 분기해줘야 해서 enum으로 처리해주었다. 물론, 이걸 일반적인 클래스로 만들어서 처리해줄 수 있지만
입력을 받아서 적절한 연산을 찾아 처리해주는 것보다는 입력된 사칙연산(문자열) 자체가 enum으로서 연산이 가능한 개념이 더 직관적이기 때문에
enum이 좋겠다고 판단했다. **<red>애초에 한정된 데이터 타입들이 정해져있고, 이에 따른 연산이 각각 정해져 있으므로 이럴때 쓰라고 둔 것이
열거형이기 때문에 이런 경우 enum이 좋은 선택지가 될 수 있는 것 같다.</red>**

그리고 이번 미션을 수행하면서 enum에 대해서 그간 원리를 모르고 기계적으로만 써왔다는 생각이 들어서 enum에 대해서 좀 찾아보았는데,
찾아보는 과정에서 내가 정말 enum에 대해서 잘 모르고 썼다는 것을 알게되었다. 이번 미션이 사실 튜토리얼 성격의 미션이고 이미 진행 했던 미션이라서
스킵을 하려 했었는데 스킵하지 않고 해보길 잘했다는 생각이 든다.

<br>

#### java의 enum은 메모리에 언제, 어떻게 할당되는가
생각해보면 new를 통해서 만들어준 적도 없는데 냅다 갖다 쓰기만 했다. 그 때부터 이상하다고 생각하고 찾아볼 생각을 했어야했다.
아무튼, 생성되기 전의 enum은 private static final 로 선언만 된 채로 jvm의 method area(= static)에 상주해 있다가
처음 만나게 되면 heap에 이를 생성해주고 생성된 heap의 주소값을 열거형으로 선언해준 상수들이 할당 받게 된다.
그래서 동일한 enum 타입을 각기 다른 변수에 할당해주고 비교해줘도 같다는 판단을 할 수 있는 것이다.

[공식문서에서도 생성자에 관해서](https://docs.oracle.com/javase/7/docs/api/java/lang/Enum.html) 아래와 같이 서술되어 있다.
> Sole constructor. Programmers cannot invoke this constructor. It is for use by code emitted by the compiler in response to enum type declarations.

**<red>정리하자면 enum에 정의해주는 value들은 enum 을 선언한 곳에서 모두 그 때 생성이 되게 된다.
그래서 value에 파라미터를 설정할 경우 이에 맞게 사용될 생성자 역시 만들어 주어야 하는 것이고, 이 때 만들어준 생성자가 call 되는 것이다.</red>**

아주 간단한 원리인데 이걸 알고 나니까 enum에 대한 활용도가 매우 높아짐을 느낄 수 있었다.

```java
package calculator.entity;

import common.Constant;

import java.util.Arrays;
import java.util.function.BiFunction;

public enum OperationType {

    PLUS(Constant.OPERATION_PLUS, (param1, param2) -> param1 + param2),
    MINUS(Constant.OPERATION_MINUS, (param1, param2) -> param1 - param2),
    MULTIPLE(Constant.OPERATION_MULTIPLE, (param1, param2) -> param1 * param2),
    DIVIDE(Constant.OPERATION_DIVIDE, (param1, param2) -> param1 / param2);

    private String operator;
    private BiFunction<Long, Long, Long> formula;

    OperationType(String operator, BiFunction<Long, Long, Long> formula) {
        this.operator = operator;
        this.formula = formula;
    }

    public static Long calculate(String operator, Long parma1, Long param2) {
        return getOperationType(operator)
                .formula
                .apply(parma1, param2);
    }

    private static OperationType getOperationType(String operator) {
        return Arrays.stream(values())
                .filter(value -> value.operator.equals(operator))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException(Constant.NOT_EXIST_OPERATOR_TYPE_MESSAGE));
    }

}
```

```java
package calculator.entity;

import common.Constant;

import java.util.Arrays;
import java.util.List;
import java.util.Scanner;
import java.util.stream.Collectors;

public class Calculator {

    public String getFormula() {
        Scanner scanner = new Scanner(System.in);
        return scanner.nextLine();
    }

    public Long getResult(String input) {
        Long result = 0L;
        List<String> formula = Arrays.stream(input.split(Constant.SPACE)).collect(Collectors.toList());
        this.addDefaultElement(formula);

        for (int i = 1; i < formula.size(); i += 2) {
            result = OperationType.calculate(formula.get(i), result, Long.parseLong(formula.get(i + 1)));
        }

        return result;
    }

    private void addDefaultElement(List<String> formula) {
        formula.add(0, Constant.FORMULA_FIRST_ELEMENT);
        formula.add(1, Constant.OPERATION_PLUS);
    }

}

```

```java
package common;

public class Constant {

    // common
    public static String SPACE = " ";

    // calculator
    public static String FORMULA_FIRST_ELEMENT = "0";
    public static String OPERATION_PLUS = "+";
    public static String OPERATION_MINUS = "-";
    public static String OPERATION_DIVIDE = "/";
    public static String OPERATION_MULTIPLE = "*";

    // error message
    public static String NOT_EXIST_OPERATOR_TYPE_MESSAGE = "연산은 +, -, *, / 중 하나만 가능합니다.";

}
```

<br>

이번에는 객체지향생활체조에 관한 항목들을 적용하는 것을 학습목표로 삼지 않아서 의식적으로 이를 적용하지는 않았지만,
평소 코딩 할때 지키고자 하는 습관 그대로를 적용해서 결과적으로는 객체지향생활체조 원칙들을 위배하지 않기 위해서 노력했다.
