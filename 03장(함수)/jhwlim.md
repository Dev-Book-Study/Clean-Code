# 📖 3장. 함수

## 요약

- 함수는 한 가지만 해야 한다.
- 서술적인 이름을 사용하라
- 함수 인수는 적어야 한다.
- 부수효과를 발생시키면 안된다.
- 함수는 명령(상태 변경) 또는 조회(정보 반환) 중 하나만 해야 한다.
- 예외를 사용하라

## 🔖 목차

1. [작게 만들어라](#01-작게-만들어라)
2. [한 가지만 해라!](#02-한-가지만-해라)
3. [함수 당 추상화 수준은 하나로!](#03-함수-당-추상화-수준은-하나로)
4. [Switch 문](#04-switch-문)
5. [서술적인 이름을 사용하라](#05-서술적인-이름을-사용하라)
6. [함수 인수](#06-함수-인수)
7. [부수 효과를 일으키지 마라!](#07-부수-효과를-일으키지-마라)
8. [명령과 조회를 분리하라!](#08-명령과-조회를-분리하라)
9. [오류 코드보다 예외를 사용하라!](#09-오류-코드보다-예외를-사용하라)
10. [반복하지 마라!](#10-반복하지-마라)
11. [구조적 프로그래밍](#11-구조적-프로그래밍)
12. [함수를 어떻게 짜죠?](#12-함수를-어떻게-짜죠)
13. [결론](#13-결론)

## 01. 작게 만들어라

- 함수를 만드는 첫째 규칙은 '작게!'다. 함수를 만드는 둘째 규칙은 '더 작게!'다.

### (1) 블록과 들여쓰기

- if 문/else 문/while 문 등에 들어가는 _**블록은 한 줄이어야 한다.**_
- 함수에서 _**들여쓰기 수준은 1단이나 2단을 넘어서면 안 된다.**_

## 02. 한 가지만 해라!

- _**함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.**_
- 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 한다.
- 함수를 만드는 이유는 큰 개념(함수 이름)을 다음 추상화 수준에서 여러 단계로 나눠 수행하기 위해서이다.
- 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.
- 한 가지 작업만 하는 함수는 자연스럽게 섹션으로 나누기 어렵다.

> 추상화 수준이 하나인 단계만 수행한다는 말의 의미는?

> 단계와 섹션의 차이는?

## 03. 함수 당 추상화 수준은 하나로!

- 함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.
- 한 함수 내에 추상화 수준을 섞으면 코드를 읽는 사람이 헷갈린다. 특정 표현이 근본 개념인지 아니면 세부사항인지 구분하기 어려운 탓이다.
- 근본 개념과 세부사항을 뒤섞기 시작하면, 깨어진 창문처럼 사람들이 함수에 세부사항을 점점 더 추가한다.

### (1) 위에서 아래로 코드 읽기: **내려가기** 규칙

- 코드는 위에서 아래로 이야기처럼 읽혀야 좋다.
- 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다. 즉, 위에서 아래로 프로그램을 읽으면 함수 추상화 수준이 한 번에 한 단계씩 낮아진다. (**내려가기 규칙**)
- 핵심은 짧으면서도 '한 가지'만 하는 함수다. 위에서 아래로 읽어내려 가듯이 코드를 구현하면 추상화 수준을 일관되게 유지하기 쉬워진다.

## 04. Switch 문

- '한 가지' 작업만 하는 switch 문도 만들기 어렵다. 본질적으로 switch 문은 N가지를 처리한다.
- 다형적 객체를 생성하는 것 이외에 switch 문은 사용하지 않도록 한다.
  - 다형적 객체를 생성하는 코드 안에서 상속 관계로 숨긴 후에 절대로 다른 코드에 노출하지 않도록 한다.

### 예시

직원 유형에 따라 다른 값을 계산해 반환하는 함수

```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
    switch (e.type) {
        case COMMISSIONED:
            return calculateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            throw new InvalidEmployeeType(e.type);
    }
}
```

위 코드의 문제점

- 함수가 길다. 새 직원을 추가하면 더 길어진다.
- '한 가지' 작업만 수행하지 않는다.
- SRP를 위반한다. 코드를 변경할 이유가 여럿이기 때문이다.
- OCP를 위반한다. 새 직원 유형을 추가할 때마다 코드를 변경하기 때문이다.
- 위 함수와 구조가 동일한 함수가 무한정 존재한다.
  예) 직원 유형에 따라 switch 문을 사용하는 `isPayday(Employee e, Date date)`(급여일자인지 확인), `deliverPay(Employee e, Money pay)`(급여를 전달)하는 메서드가 존재할 수 있다.

> SRP란?

    단일책임의 원칙 (Single Responsibility Principle), 한 클래스는 하나의 책임만 가져야 한다. 어떤 클래스를 변경해야 하는 이유는 오직 하나 뿐이어야 한다.

> OCP란?

    개방-폐쇄 원칙 (Open/Close Principle), 자신의 확장에는 열려 있고, 주변의 변화에 대해서는 닫혀 있어야 한다. 인터페이스를 통해 구현하여 해결한다.

### 문제 개선하기

switch 문을 추상 팩토리에 숨긴다. 팩토리는 switch 문을 사용해 적절한 `Employee` 파생 클래스의 인스턴스를 생성한다.

```java
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}
```

```java
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
```

```java
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r);
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmployee(r);
            default:
                throw new InvalidEmployeeType(r.type);
        }
    }
}
```

## 05. 서술적인 이름을 사용하라!

- "코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다."(워드)
- 한 가지만 하는 작은 함수에 좋은 이름을 붙인다면 이런 원칙을 달성함에 있어 이미 절반은 성공했다. 함수가 작고 단순할수록 서술적인 이름을 고르기도 쉬워진다.
- 이름이 길어도 괜찮다. 길고 서술적인 이름이 짧고 어려운 이름보다 좋다. 길고 서술적인 이름이 길고 서술적인 주석보다 좋다.
- 함수 이름을 정할 때는 여러 단어가 쉽게 읽히는 명명법을 사용한다. 그런 다음, 여러 단어를 사용해 함수 기능을 잘 표현하는 이름을 선택한다.
- 이름을 정하느라 시간을 들여도 괜찮다. 이런저런 이름을 넣어 코드를 읽어보면 더 좋다.
- 서술적인 이름을 사용하면 개발자 머릿속에서도 설계가 뚜렷해지므로 코드를 개선하기 쉬워진다.
- 이름을 붙일 때는 일관성이 있어야 한다. 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용한다.
  좋은 예) `includeSetupAndTeardownPages`, `includeSetupPages`, `includeSuiteSetupPage`, `includeSetupPage` 등
- 문체가 비슷하면 이야기를 순차적으로 풀어가기도 쉬워진다.

## 06. 함수 인수

- 함수에서 이상적인 인수 개수는 0개(무항)다. 다음은 1개(단항)고, 다음은 2개(이항)다.
- 3개(삼항)는 가능한 피하는 편이 좋다.
- 4개(다항)은 특별한 이유가 필요하다. 특별한 이유가 있어도 사용하면 안된다.
- 인수는 개념을 어렵게 만든다.
- 테스트 관점에서 보면 인수는 더 어렵다. 인수마다 유효한 값으로 모든 조합을 구성해 테스트하기가 상당히 부담스러워진다.
- 출력 인수는 입력 인수보다 이해하기 어렵다.
- _**최선은 입력 인수가 없는 경우이며, 차선은 입력 인수가 1개뿐인 경우다.**_

> 출력 인수란? 파라미터로 넘기고 파라미터로 결과값을 저장한다?

### (1) 많이 쓰는 단항 형식

- 함수에 인수가 1개를 넘기는 이유로 가장 흔한 경우는 두 가지다.
  1. _**하나는 인수에 질문을 던지는 경우다.**_
     좋은 예) `boolean fileExists("MyFile")`
  2. _**다른 하나는 인수를 뭔가로 변환해 결과를 반환하는 경우다.**_
     좋은 예) `InputStream fileOpen("MyFile") : String 형의 파일 이름을 InputStream으로 변환한다.
- 함수 이름을 지을 때는 두 경우를 분명히 구분한다.
- 또한 언제나 일관적인 방식으로 두 형식을 사용한다.

- _**다소 드물게 사용하지만 그래도 아주 유용한 단항 함수 형식이 이벤트다.**_
- 이벤트 함수는 입력 인수만 있다. 출력 인수는 없다. 프로그램은 함수 호출을 이벤트로 해석해 입력 인수로 시스템 상태를 바꾼다.
  좋은 예) `passwordAttemptFailedNtimes(int attempts)`
- 이벤트라는 사실이 코드에 명확히 드러나야 한다. 그러므로 이름과 문맥을 주의해서 선택한다.

- 지금까지 설명한 경우가 아니라면 단항 함수는 가급적 피한다.
  나쁜 예) `void includeSetupPageInfo(StringBuffer pageText)`
  _**변환 함수에서 출력 인수를 사용하면 혼란을 일으킨다. 입력인수를 반환하는 함수라면 변환 결과는 반환값으로 돌려준다.**_
  좋은 예) `StringBuffer transform(StringBuilder in)`
  입력 인수를 그대로 돌려주는 함수라 할지라도 변환 함수 형식을 따르는 편이 좋다. 적어도 변환 형태는 유지하기 때문이다.

### (2) 플래그 인수

- _**플래그 인수는 추하다. 플래그 인수가 참이면 이걸 하고 거짓이면 저걸 한다는 말이다.**_
  나쁜 예) `render(boolean isSuite)` : 헷갈리기 쉽다.
  좋은 예) `renderForSuite()`, `renderForSingleTest()`

### (3) 이항 함수

- 인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다.
  - 좋은 예) `writeField(name)`
  - 나쁜 예) `writeField(outputStream, name)`
- 이항 함수가 적절한 경우도 있다.
  좋은 예) 직교 좌표계 점은 일반적으로 인수 2개를 취한다.
  ```java
  Point p = new Point(0, 0);
  ```
  나쁜 예) `assertEquals(expected, actual)` : 두 인수는 자연적인 순서가 없다. expected 다음에 actual이 온다는 순서를 인위적으로 기억해야 한다.
- _**이항 함수가 무조건 나쁘다는 소리는 아니다. 불가피한 경우도 생긴다. 하지만 그만큼 위험이 따른다는 사실을 이해하고 가능하면 단항 함수로 바꾸도록 애써야 한다.**_
  방법 1) writeField 메서드를 outputStream 클래스 구성원으로 만들어 `outputStream.writeField(name)`
  방법 2) FieldWriter라는 새 클래스를 만들어 구성자에서 outputStream을 받고 write메서드를 구현한다.

### (4) 삼항 함수

- 인수가 3개인 함수는 인수가 2개인 함수보다 휠썬 더 이해하기 어렵다. 순서, 주춤, 무시로 야기되는 문제가 두 배 이상 늘어난다. 그래서 삼항 함수를 만들 때는 신중히 고려하라 권고한다.
  나쁜 예) `assertEquals(message, expected, actual)` : 첫 인수가 expected라고 예상된다. 매번 함수를 볼 때마다 주춤했다가 message를 무시해야 한다는 사실을 상기한다.
  좋은 예) `assertEquals(1.0, amount, .001)` : 여전히 주춤하게 되지만 그만한 가치가 충분하다. 부동소수점 비교가 상대적이라는 사실은 언제든 주지할 중요한 사항이다.

### (5) 인수 객체

- _**인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어본다.**_

#### 예제

```java
Circle makeCircle(double x, double y, double radius); // bad

Circle makeCircle(Point center, double radius); // better
```

변수를 묶어 넘기려면 이름을 붙여야 하므로 결국은 개념을 표현하게 된다.

### (6) 인수 목록

- 때로는 인수 개수가 가변적인 함수도 필요하다.
  좋은 예) `String.format()`
  ```
  public String format(String format. Object... args)
  ```
- 가변 인수를 취하는 모든 함수에 같은 원리가 적용된다. 가변 인수를 취하는 함수는 단항, 이항, 삼항 함수로 취급할 수 있다. 가변 인수를 취하는 함수는 단항, 이항, 삼항 함수로 취급할 수 있다.
- 하지만 이를 넘어서는 인수를 사용할 경우에는 문제가 있다.

### (7) 동사와 키워드

- _**함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필수다.**_
- _**단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다.**_
  좋은 예) `writeField(name)`
- 함수 이름에 키워드를 추가하는 형식이다. 즉, _**함수 이름에 인수 이름을 넣는다.**_
  나쁜 예) `assertEquals(expected, actual)`
  좋은 예) `assertExpectedEqualsActual(expected, actual)`

## 07. 부수 효과를 일으키지 마라!

- _**함수에서 한 가지를 하겠다고 약속하고선 남몰래 다른 짓도 하니까 문제이다.**_
- _**많은 경우 시간적인 결합(temporal coupling)이나 순서 종속성(order dependency)을 초래한다.**_

### 예시

```java
public class UserValidator {
    private Cryptographer cryptographer;

    public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);
        if (user != User.NULL) {
            String codePhrase = user.getPhraseEncodedByPassword();
            String phrase = cryptographer.decrypt(codedPhrase, password);
            if ("Valid Password".equals(phrase)) {
                Session.initialized();
                return true;
            }
        }
        return false;
    }
}
```

위 코드의 문제점

- 여기서, 함수가 일으키는 부수 효과는 Session.initialize() 호출이다.
- 이름만 봐서는 세션을 초기화한다는 사실이 드러나지 않는다. 그래서 함수 이름만 보고 함수를 호출하는 사용자는 사용자를 인증하면서 기존 세션 정보를 지워버릴 위험에 처한다.
- 이런 _**부수 효과가 시간적인 결합을 초래한다.**_ 즉, checkPassword 함수는 _**특정 상황에서만 호출이 가능하다.**_ 다시 말해, 세션을 초기화해도 괜찮은 경우에만 호출이 가능하다. 자칫 잘못 호출하면 의도하지 않게 세션 정보가 날아간다.
- 시간적인 결합은 혼란을 일으킨다. 특히 부수 효과로 숨겨진 경우에는 더더욱 혼란이 커진다. 만약 _**시간적인 결합이 필요하다면 함수 이름에 분명히 명시한다.**_
- `checkPasswordAndInitializeSession`이라는 이름이 훨씬 좋다. 물론 함수가 '한 가지'만 한다는 규칙을 위반한다.

> 그러면 어떻게 코드를 바꿔야 할까요?

```java
public void login(userName, password) {
    if (!checkPassword(userName, password)) {
        Session.initialize();
    }
}
```

### 출력 인수

- 일반적으로 우리는 인수를 함수 입력으로 해석한다.
- 함수 선언부를 찾아보는 행위는 코드를 보다가 주춤하는 행위와 동급이다. _**인지적으로 거슬린다는 뜻이므로 피해야 한다.**_
- 객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다. 출력 인수로 사용하라고 설계한 변수가 바로 `this`이기 때문이다.
  좋은 예) `report.appendFooter()`
- _**일반적으로 출력 인수는 피해야 한다. 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택한다.**_

## 08. 명령과 조회를 분리하라!

- 함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다. 둘 다 하면 안된다.
- _**객체 상태를 변경하거나 아니면 객체 정보를 반환하거나 둘 중 하나다.**_ 둘 다 하면 혼란을 초래한다.

### 예시

```java
// 이름이 'attribute'인 속성을 찾아 값을 value로 설정한 후 성공하면 true, 실패하면 false를 반환한다.
public boolean set(String attribute, String value);
```

위 코드의 문제점

- 독자 입장에서는 "username"이 "unclebob"으로 설정되어 있는지 확인하는 코드인지, 아니면 "username"을 "unclebob"으로 설정하는 코드인지 함수를 호출하는 코드만 봐서는 의미가 모호하다. "set"이라는 단어가 동사인지 형용사인지 분간하기 어려운 탓이다.

```java
if (set("username", "unclebob"))
```

### 문제 개선하기

명령과 조회를 분리한다.

```java
if (attributeExists("username")) {
    setAttribute("username", "unclebob")
}
```

## 09. 오류 코드보다 예외를 사용하라!

- 명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다. 자칫하면 if 문에서 명령을 표현식으로 사용하기 쉬운 탓이다.

### 예시

```java
if (deletePage(page) == E_OK)
```

```java
if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
            logger.log("page delete");
        } else {
            logger.log("configKey not delete");
        }
    } else {
        logger.log("deleteReference from registry failed");
    }
} else {
    logger.log("delete failed");
    return E_ERROR;
}
```

위 코드의 문제점

- 위 코드는 동사/형용사 혼란을 일으키지 않는 대신 여러 단계로 중첩되는 코드를 야기한다.
- 오류 코드를 반환하면 호출자는 오류 코드를 곧바로 처리해야 한다는 문제에 부딪힌다.

### 문제 개선하기

오류 코드 대신 예외를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해진다.

```java
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}
catch (Exception e) {
    logger.log(e.getMessage());
}
```

### (1) Try/Catch 블록 뽑아내기

- try/catch 블록은 원래 추하다. 코드 구조에 혼란을 일으키며, 정상 동작과 오류 처리 동작을 뒤섞는다.
- 그러므로 _**try/catch 블록을 별도 함수로 뽑아내는 편이 좋다.**_
- _**정상 동작과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.**_

```java
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    }
    catch (Exception e) {
        logError(e);
    }
}

private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) {
    logger.log(e.getMessage());
}
```

### (2) 오류 처리도 한 가지 작업이다.

- 함수는 '한 가지' 작업만 해야 한다. 오류 처리도 '한 가지' 작업에 속한다.
- 그러므로 _**오류를 처리하는 함수는 오류만 처리해야 마땅하다.**_
- _**함수에 키워드 try가 있다면 함수는 try 문으로 시작해 catch/finally 문으로 끝나야 한다**_

### (3) Error.java 의존성 자석

- 오류 코드를 반환한다는 이야기는, 클래스든 열거형 변수든, 어디선가 오류 코드를 정의한다는 뜻이다.

```java
// 의존성 자석 예시
public enum Error {
    OK,
    INVALID,
    NO_SUCH,
    LOCKED,
    OUT_OF_RESOURCES,
    WAITING_FOR_EVENT;
}
```

- 의존성 자석의 문제점
  - Error enum이 변한다면 Error enum을 사용하는 클래스 전부를 다시 컴파일하고 다시 배치해야 한다.
  - 그래서 Error 클래스 변경이 어려워진다.
  - 그래서 새 오류 코드를 추가하는 대신 기존 오류 코드를 재사용한다.
- 오류 코드 대신 예외를 사용하면 새 예외는 Exception 클래스에서 파생된다. 따라서 재컴파일/재배치 없이도 새 예외 클래스를 추가할 수 있다.

## 10. 반복하지 마라!

- 중복은 문제다. 코드 길이가 늘어날 뿐 아니라 알고리즘이 변하면 같은 알고리즘을 사용하는 곳도 손봐야 한다. 게다가 어느 한곳이라도 빠뜨리는 바람에 오류가 발생할 확률도 그만큼 높다.
- 중복은 소프트웨어에서 모든 악의 근원이다. 많은 원칙과 기법이 중복을 없애거나 제어할 목적으로 나왔다.
  - 관계형 데이터베이스의 정규 형식
  - 객체지향 프로그래밍은 코드를 부모클래스로 몰아 중복을 없앤다.
  - 구조적 프로그래밍, AOP, COP 모두 어떤 면에서 중복 제거 전략이다.

## 11. 구조적 프로그래밍

- 함수는 return 문이 하나여야 한다. 루프 안에서 break나 continue를 사용해선 안 되며 goto는 절대로 절대로 안된다. (구조적 프로그래밍 원칙, 단일 입/출구 규칙)
- 함수를 작게 만든다면 위 규칙은 별 이익을 제공하지 못한다. 함수가 아주 클 때만 상당한 이익을 제공한다.
- 그러므로 _**함수를 작게 만든다면 간혹 return, break, continue를 여러 차례 사용해도 괜찮다. 오히려 단일 입/출구 규칙보다 의도를 표현하기 쉬워진다.**_
- 반면, _**goto 문은 큰 함수에서만 의미가 있으므로, 작은 함수에서는 피해야만 한다.**_

## 12. 함수를 어떻게 짜죠?

- 처음에는 길고 복잡하다. 들여쓰기 단계도 많고 중복된 루프도 많다. 인수 목록도 아주 길다. 이름은 즉흥적이고 코드는 중복된다. 하지만 나는 그 서투른 코드를 빠짐없이 테스트하는 단위 테스트 케이스도 만든다.
- 그런 다음 나는 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. 메서드를 줄이고 순서를 바꾼다. 때로는 전체 클래스를 쪼개기도 한다. 이 와중에도 코드는 항상 단위 테스트를 통과한다.

## 13. 결론

- 모든 시스템은 특정 응용 분야 시스템을 기술할 목적으로 프로그래머가 설계한다 도메인 특화 언어(Domain Specific Language, DSL)로 만들어진다. 함수는 그 언어에서 동사며, 클래스는 명사다.
- 마스터 프로그래머는 시스템을 구현할 프로그램이 아니라 풀어갈 이야기로 여긴다. 프로그래밍 언어라는 수단을 사용해 좀 더 풍부하고 좀 더 표현력이 강한 언어를 만들어 이야기를 풀어간다. 시스템에서 발생하는 모든 동작을 설명하는 함수 계층이 바로 그 언어에 속한다.
- 진짜 목표는 시스템이라는 이야기를 풀어가는 데 있다는 사실을 명심하기 바란다.
- _**함수는 분명하고 정확한 언어로 깔끔하게 같이 맞아떨어져야 이야기를 풀어가기가 쉬워진다**_ 는 사실을 기억하기 바란다.
