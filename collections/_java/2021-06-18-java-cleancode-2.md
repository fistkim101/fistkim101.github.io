---
layout: article
title: 클린코드 숫자야구게임
tags: 클린코드 객체지향생활체조 숫자야구게임
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

웹만 다루다가 웹과 무관한 콘솔 프로그램을 만드는 경험이 꽤 오랜만이었다. 알고리즘 공부하느라 했던 것들은 원하는 값을 나오도록 하는 메소드를 만드는 것이었지
나름의 흐름을 갖고 있는 프로그램 느낌은 아니었던 것 같다. 아무튼 재미가 있었는데 예전에 처음 이런 미션을 수행하면서 힘들었던 부분들은 많이 해소가 되어 있었다.
왜냐하면 어느정도 원칙에 의거해서 코딩하는 습관이 길러져 있었기 때문이다.

하지만 아직 부족하다는 생각을 많이 했다. 나는 단순히 '이건 이래야해', '저건 저래야해' 와 같이 숙지하고 있던 원칙을 그대로 적용하는 것이 아니라
클린한 코드를 위해서 원칙을 적용하는 과정상에서의 노하우가 머릿속에 구체화 되어있으면 좋겠다는 생각을 했다.

**<blue>지금은 뭔가 원칙을 적용하기 위해 '노력하다보니' 모양새가 점점 원하는 형태로 잡히는 것이고, 내가 원하는 것은 '노력하다보니'의 과정에서 그 노력이 케이스에 따라
여러가지 구체적은 나의 방법론이 자리잡히길 원한다. 그래야 속도가 빨라질 수 있고 케이스마다 더 정확하게 클린코드를 적용할 수 있을 것이라는 생각이 든다.</blue>**

그리고 이번에 의도적으로 클린코드 원칙을 위배한 것이 있다. 나는 아직도 이 원칙이 맞는지 잘 모르겠다. 미션을 수행하는 중에 삼항연산자를 사용했는데, 이는 원칙에 위배된 코드다.
삼항연산자를 씀으로써 더 깔끔한 코드가 되는 것 같은데, 이 '깔끔함'이라는 것도 개인차가 있기 때문에 뭐가 무조건 맞다 하기가 좀 모호하다.
확실한 것은 틀은 삼항연산자 이지만 코드 자체가 엄청 길고 복잡하다면 삼항연산자가 적절하지 않은 것 같다.

**<blue>평소에도 느꼈지만 이번에 미션을 진행하면서 느꼈던 재밌는 점은 코드를 작성하면서 정한 변수명이나 네이밍이 작성 당시에는 굉장히 잘 만든듯 하다가도
잠시 다른 사람인 것처럼(?) 빙의를 하여서 코드를 처음보는 사람처럼 클래스명, 변수명, 메소드명 들에 집중하여 코드 흐름을 이해 하다보면 어울리지 않는 메소드명과 변수명이 많다는 것이다.
이건 당연히 실력의 문제이긴 하지만 반대로 이러한 방식으로 계속 자가점검을 하는 것이 좋겠다는 생각이 든다.</blue>**

**<red>메소드 하나 만들고서 코드 처음보는 사람처럼 점검하면서 '이해의 흐름에 메소드 명이나 변수명이 자연스러운가 방해가 되지는 않는가'와 같은 비판적인 눈으로 코드를 검사하는 습관을 들여야겠다.</red>**

<br>

### 요구사항
기본적으로 1부터 9까지 서로 다른 수로 이루어진 3자리의 수를 맞추는 게임이다.

* 같은 수가 같은 자리에 있으면 스트라이크, 다른 자리에 있으면 볼, 같은 수가 전혀 없으면 포볼 또는 낫싱이란 힌트를 얻고, 그 힌트를 이용해서 먼저 상대방(컴퓨터)의 수를 맞추면 승리한다.
    * e.g. 상대방(컴퓨터)의 수가 425일 때, 123을 제시한 경우 : 1스트라이크, 456을 제시한 경우 : 1볼 1스트라이크, 789를 제시한 경우 : 낫싱
* 위 숫자 야구 게임에서 상대방의 역할을 컴퓨터가 한다. 컴퓨터는 1에서 9까지 서로 다른 임의의 수 3개를 선택한다. 게 임 플레이어는 컴퓨터가 생각하고 있는 3개의 숫자를 입력하고, 컴퓨터는 입력한 숫자에 대한 결과를 출력한다.
* 이 같은 과정을 반복해 컴퓨터가 선택한 3개의 숫자를 모두 맞히면 게임이 종료된다.
* 게임을 종료한 후 게임을 다시 시작하거나 완전히 종료할 수 있다.

<br>

### 실행결과
```bash
숫자를 입력해 주세요 : 123
1볼 1스트라이크
숫자를 입력해 주세요 : 145
1볼
숫자를 입력해 주세요 : 671
2볼
숫자를 입력해 주세요 : 216
1스트라이크
숫자를 입력해 주세요 : 713
3스트라이크
3개의 숫자를 모두 맞히셨습니다! 게임 종료
게임을 새로 시작하려면 1, 종료하려면 2를 입력하세요.
1
숫자를 입력해 주세요 : 123
1볼 1스트라이크
…
```

<br>

### 학습목표
* 자바 코드 컨벤션을 지키면서 프로그래밍한다.
    * 기본적으로 Google Java Style Guide을 원칙으로 한다.
    * 단, 들여쓰기는 '2 spaces'가 아닌 '4 spaces'로 한다.
* indent(인덴트, 들여쓰기) depth를 2가 넘지 않도록 구현한다. 1까지만 허용한다.
    * 예를 들어 while문 안에 if문이 있으면 들여쓰기는 2이다.
    * 힌트: indent(인덴트, 들여쓰기) depth를 줄이는 좋은 방법은 함수(또는 메소드)를 분리하면 된다.
* else 예약어를 쓰지 않는다.
    * 힌트: if 조건절에서 값을 return하는 방식으로 구현하면 else를 사용하지 않아도 된다.
    * else를 쓰지 말라고 하니 switch/case로 구현하는 경우가 있는데 switch/case도 허용하지 않는다.
* 모든 로직에 단위 테스트를 구현한다. 단, UI(System.out, System.in) 로직은 제외
    * 핵심 로직을 구현하는 코드와 UI를 담당하는 로직을 구분한다.
    * UI 로직을 InputView, ResultView와 같은 클래스를 추가해 분리한다.
* 3항 연산자를 쓰지 않는다.
* 함수(또는 메소드)가 한 가지 일만 하도록 최대한 작게 만들어라.

<br>

### [실습코드](https://github.com/fistkim101/java-baseball-playground)

**Judge**
```java
package baseball.domain;

import baseball.common.Constant;

import java.util.HashSet;
import java.util.List;
import java.util.Random;
import java.util.Set;

public class Judge {

    public void generateAnswers(List<String> answers) {
        Random random = new Random();
        Set<Integer> answersSet = new HashSet<>();

        this.collectNumbers(random, answersSet);
        answersSet.forEach(answer -> answers.add(answer.toString()));
    }

    public int getStrikeCount(List<String> inputs, List<String> answers) {
        int count = Constant.ZERO;
        for (String input : inputs) {
            count += isStrike(inputs.indexOf(input), input, answers) ? 1 : 0;
        }

        return count;
    }

    public int getBallCount(List<String> inputs, List<String> answers) {
        int count = Constant.ZERO;
        for (String input : inputs) {
            count += isBall(inputs.indexOf(input), input, answers) ? 1 : 0;
        }

        return count;
    }

    private boolean isStrike(int index, String input, List<String> answers) {
        return input.equals(answers.get(index));
    }

    private boolean isBall(int index, String input, List<String> answers) {
        return !input.equals(answers.get(index)) && answers.contains(input);
    }

    private void collectNumbers(Random random, Set<Integer> answersSet) {
        while (answersSet.size() != Constant.CORRECT_ANSWER_STRIKE_COUNT) {
            answersSet.add(random.nextInt(Constant.MAX_NUMBER) + 1);
        }
    }

}
```

일단 제일 신경쓴 것은 최대한 객체의 상태값으로 가지고 있을 무엇인가가 없도록 하는 것이었다.
이를 위해서 각 기능에 필요한 것을 파라미터로 받아서 필요한 처리를 한 후 다시 return을 해주는 식으로 작성하고자 노력했다.

예전에도 피드백을 받을 때 클래스 내의 지역변수를 최대한 없애도록 피드백을 받았었는데, 이건 결국 '과연 그 상태값을 이 클래스가 책임지고 관리해야 되는 것이냐', 혹은
'이 클래스의 책임과 역할에 이 지역변수가 과연 적절한 것이냐'의 질문으로 이어졌었다.

이번에 다시 이걸 해보는 과정에서도 위 두 질문을 스스로에게 하면서 작성하다 보니 실제로 내가 선언하고자 하는 지역변수가
내가 실제로 부여하고자 하는 책임과 역할을 생각하면 이 클래스가 관리해야할 상태값으로 적절하지 않았다.

위 코드중 마음에 안드는 부분은 getStrikeCount와 getBallCount이 내부에서 분기만 해주면 하나의 메소드로 처리가 가능하다는 것인데,
이걸 분기하고자 boolean 값으로 isBall 혹은 isStrike를 받아서 내부에서 분기를 해주면 이를 받아주는 메소드의 파라미터가 너무 많아져서
일단 거의 같은 로직인데도 메소드를 따로 두게 되었다.

<br>

**Moderator**
```java
package baseball.domain;

import baseball.common.Constant;
import baseball.ui.InputView;
import baseball.ui.ResultView;

import java.util.ArrayList;
import java.util.List;

public class Moderator {

    private final Judge judge;

    public Moderator(Judge judge) {
        this.judge = judge;
    }

    public void startBaseBall() {
        List<String> answers = new ArrayList<>();
        judge.generateAnswers(answers);
        boolean allInputsCorrect = false;

        while (!allInputsCorrect) {
            List<String> inputs = InputView.getUserInputNumber();
            allInputsCorrect = this.isAllInputsCorrect(inputs, answers);
        }
    }

    private boolean isAllInputsCorrect(List<String> inputs, List<String> answers) {
        int ballCount = judge.getBallCount(inputs, answers);
        int strikeCount = judge.getStrikeCount(inputs, answers);
        ResultView.showResult(ballCount, strikeCount);

        if (strikeCount != Constant.CORRECT_ANSWER_STRIKE_COUNT) {
            return false;
        }

        if (ResultView.isNewGameStart()) {
            answers.clear();
            judge.generateAnswers(answers);
            return false;
        }

        return true;
    }
}
```

원래 main 메소드에게 이 역할을 주려고 했으나, Judge 객체를 구분 해준 것 처럼 '게임을 진행'해주는 하나의 존재가 있는 것이
코드의 구조를 이해하기에 더 용이한 것 같아서 Moderator라는 클래스를 만들어서 이 역할을 부여했다.

특별한 점은 없지만 정답을 모두 맞춘 경우에 게임을 더 할지 말지를 물어보고 새 게임을 진행 혹은 종료를 시키는 부분에서
continue와 break을 사용하게 되었었는데, 가독성이 떨어지는 것 같아서 조건에 맞는 조기종결들을 사용함으로써
기존에 사용한 continue와 break을 없애고 위와 같은 코드로 수정하게 되었다.

<br>

**JudgeTest**
```java
package baseball.domain;

import baseball.common.Constant;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;

class JudgeTest {

    Judge judge = new Judge();

    @DisplayName("1 ~ 9 사이의 서로 다른 임의의 수를 이용하여 정답 생성 하는지 테스트")
    @Test
    void generateAnswerTest() {
        List<String> answers = new ArrayList<>();
        judge.generateAnswers(answers);

        assertEquals(Constant.CORRECT_ANSWER_STRIKE_COUNT, answers.size());
    }

    @DisplayName("ball 갯수 판단 테스트")
    @ParameterizedTest(name = "{displayName} : {index} // {0}")
    @ValueSource(strings = {"123-389", "734-741", "927-129", "365-135"})
    void getBallCountTest(String param) {
        List<String> inputs = this.convertStringToArray(param, true);
        List<String> answers = this.convertStringToArray(param, false);
        assertEquals(1, judge.getBallCount(inputs, answers));
    }

    @DisplayName("strike 갯수 판단 테스트")
    @ParameterizedTest(name = "{displayName} : {index} // {0}")
    @ValueSource(strings = {"123-126", "423-823", "187-987", "365-369"})
    void getStrikeCountTest(String param) {
        List<String> inputs = this.convertStringToArray(param, true);
        List<String> answers = this.convertStringToArray(param, false);
        assertEquals(2, judge.getStrikeCount(inputs, answers));
    }

    private List<String> convertStringToArray(String param, boolean isInput) {
        int index = isInput ? 0 : 1;
        String target = param.split(Constant.SEPERATOR)[index];
        return Arrays.asList(target.split(Constant.EMPTY));
    }

}
```

이건 피드백을 받았다면 테스트케이스가 적다고 분명 피드백을 받을 테스트 케이스 커버리지다.
실무에서 테스트 코드로 동료들과 티키타카를 해본 경험이 적어서 보통 다른 회사에서는 테스트 케이스를 작성할 때에
얼마만큼 감안을 해놓는지가 좀 궁금하다.
