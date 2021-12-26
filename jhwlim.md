# 📖 2장. 의미 있는 이름

## 🔖 목차

1. [의도를 분명히 밝혀라](#01-의도를-분명히-밝혀라)
2. [그릇된 정보를 피하라](#02-그릇된-정보를-피하라)
3. [의미 있게 구분하라](#03-의미-있게-구분하라)
4. [발음하기 쉬운 이름을 사용하라](#04-발음하기-쉬운-이름을-사용하라)
5. [검색하기 쉬운 이름을 사용하라](#05-검색하기-쉬운-이름을-사용하라)
6. [인코딩을 피하라](#06-인코딩을-피하라)
7. [자신의 기억력을 자랑하지 미라](#07-자신의-기억력을-자랑하지-마라)
8. [클래스 이름](#08-클래스-이름)
9. [메서드 이름](#09-메서드-이름)
10. [기발한 이름은 피하라](#10-기발한-이름은-피하라)
11. [한 개념에 한 단어를 사용하라](#11-한-개념에-한-단어를-사용하라)
12. [말장난을 하지 마라](#12-말장난을-하지-마라)
13. [해법 영역에서 가져온 이름을 사용하라](#13-해법-영역에서-가져온-이름을-사용하라)
14. [문제 영역에서 가져온 이름을 사용하라](#14-문제-영역에서-가져온-이름을-사용하라)
15. [의미 있는 맥락을 추가하라](#15-의미-있는-맥락을-추가하라)
16. [불필요한 맥락을 없애라](#16-불필요한-맥락을-없애라)

## 01. 의도를 분명히 밝혀라

> _의도가 분명한 이름은 정말로 중요하다는. 좋은 이름을 지으려면 시간이 걸리지만 좋은 이름으로 절약하는 시간이 훨씬 더 많다._

### 예시 : 지뢰찾기

아래의 코드는 코드 맥락이 코드 자체에 명시적으로 드러나지 않는 문제가 있다. (**코드의 함축성 문제**)

```java
public List<int[]> getThem() {
	List<int[]> list1 = new ArrayList<int[]>(); // (1)
	for (int[] x : theList) // (2)
		if (x[0] == 4) // (3), (4)
			list1.add(x);
	return list1;
}
```

### 개선하기 (1)

(1) `list1` : 깃발이 꽃힌 상태의 게임판의 칸 (→ `flaggedCells`) <br>
(2) `theList` : 게임판 (→ `gameBoard`) <br>
(3) `0` : 배열에서 인덱스 0의 값은 칸 상태 (→ `STATUS_VALUE`) <br>
(4) `4` : 배열에서 인덱스 4의 값은 깃발이 꽃힌 상태 (→ `FLAGGED`)

```java
public List<int[]> getFlaggedCells() {
	List<int[]> flaggedCells = new ArrayList<int[]>();
	for (int[] cell : gameBoard)
		if (x[STATUS_VALUE] == FLAGGED)
			list1.add(cell);
	return flaggedCells;
}
```

### 개선하기 (2)

한 걸음 더 나아가, `int[]` 배열 대신에 게임판 각 칸을 간단한 클래스로 만들어도 된다. (→ `Cell` 클래스)

```java
public List<Cell> getFlaggedCells() {
	List<Cell> flaggedCells = new ArrayList<Cell>();
	for (Cell cell : gameBoard)
		if (x[STATUS_VALUE] == FLAGGED)
			list1.add(cell);
	return flaggedCells;
}
```

## 02. 그릇된 정보를 피하라

> _코드에 그릇된 단서를 남겨서는 안된다. 그릇된 단서는 코드 의미를 흐린다._

- 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용해도 안된다.
- 일관성이 떨어지는 표기법은 그릇된 정보다.

## 03. 의미 있게 구분하라

> _연속된 숫자를 덧붙이거나 불용어(noise word)를 추가하는 방식은 적절하지 못하다. **이름이 달라야 한다면 의미도 달라져야 한다.** 읽는 사람이 차이를 알도록 이름을 지어라._

- 컴파일러나 인터프리터만 통과하려는 생각으로 코드를 구현하지 말자.
- 연속적인 숫자를 덧붙인 이름(`a1`, `a2`, ...)은 그릇된 정보를 제공하는 이름도 아니며, 아무런 정보를 제공하지 못하는 이름일 뿐이다.
- 불용어를 추가한 이름 역시 아무런 정보도 제공하지 못한다. <br> 나쁜 예) `Product`, `ProductInfo`, `ProductData` 는 개념을 구분하지 않은 채 이름만 달리한 경우이다. `Info`나 `Data`는 의미가 불분명한 불용어이다.
- 불용어는 중복이다. 변수 이름에 `variable`이라는 단어는 단연코 금물이다. 표 이름에 table이라는 단어도 마찬가지이다.

## 04. 발음하기 쉬운 이름을 사용하라

> _발음하기 쉬운 이름을 선택한다._

- 지적인 대화가 가능해진다.
- 반면에, 발음하기 어려운 이름은 토론하기 어렵다.
- 나쁜 예) `genymdhms` (generate date, year, month, day, hour, minute, second)

## 05. 검색하기 쉬운 이름을 사용하라

> _이름 길이는 범위 크기에 비례해야 한다. 변수나 상수를 코드 여러 곳에서 사용한다면 검색하기 쉬운 이름이 바람직하다._

- 문자 하나를 사용하는 이름과 상수는 텍스트 코드에서 쉽게 눈에 띄지 않는다는 문제점이 있다.
- 이런 관점에서 긴 이름이 짧은 이름보다 좋다. 검색하기 쉬운 이름이 상수보다 좋다.
- 이름을 의미있게 지으면 함수가 길어지지만, 검색이 쉬워진다.

## 06. 인코딩을 피하라

> _유형이나 범위 정보까지 인코딩에 넣으면 그만큼 이름을 해독하기 어려워진다._

- 개발자에게 인코딩은 불필요한 정신적 부담이다. 인코딩한 이름은 거의가 발음하기 어려우며 오타가 생기기도 쉽다.
- 변수, 함수, 클래스 이름이나 타입을 바꾸기가 어려워지며, 읽기도 어려워진다. 독자가 오도할 가능성도 커진다.
- 멤버 변수에 `m_` 접두어를 붙일 필요가 없다. 클래스와 함수는 접두어가 필요없을 정도로 작아야 마땅하다. 코드를 읽을수록 접두어는 관심 밖으로 밀려난다.
- 인터페이스 이름은 접두어를 붙이지 않는 편이 좋다. 만약 인터페이스 클래스 이름과 구현 클래스 이름 중 하나를 인코딩해야 한다면 구현 클래스 이름을 택하겠다.

## 07. 자신의 기억력을 자랑하지 마라

> _**명료함이 최고다.** 자신의 능력을 좋은 방향으로 사용해 남들이 이해하는 코드를 내놓아라._

- 일반적으로 문제 영역이나 해법 영역에서 사용하는 이름(단어)를 선택해라.
- 문자 하나만 사용하는 변수 이름은 문제가 있다.
- 예외적으로, 루프에서 반복 횟수를 세는 변수 `i`, `j`, `k`는 괜찮다. (`l`은 절대 안된다.) 단, 루프 범위가 아주 작고 다른 이름과 충돌하지 않을 때만 괜찮다. 루프에서 반복 횟수 변수는 전통적으로 한 글자를 사용하기 때문이다. 그 외에는 대부분 적절하지 못하다.

## 08. 클래스 이름

> _클래스 이름과 객체 이름은 명사나 명사구가 적합하다._

- 동사는 사용하지 않는다.
- 좋은 예) `Customer`, `WikiPage`, `Account`, `AddressParser` 등
- 나쁜 예) `Manager`, `Processor`, `Data`, `Info`

## 09. 메서드 이름

> _메서드 이름은 동사나 동사구가 적합하다._

- 좋은 예 : `postPayment`, `deletePage`, `save` 등
- 접근자(Accessor), 변경자(Mutator), 조건자(Predicate)는 javabean 표준에 따라 값 앞에 `get`, `set`, `is`를 붙인다. (예: `getName()`, `setName()`, `isPosted()` 등)
- 생성자를 중복정의(Overload)할 때는 <u>_정적 팩토리 메서드_</u>를 사용한다. <br> 정적 팩토리 메소드란? 객체를 생성하는 메소드를 만들고 `static`으로 선언하는, 객체 생성을 캡슐화하는 기법

### 정적 팩토리 메서드 사용 예시

```java
Complex fulcrumPoint = new Complex(23.0); // bad

Complex fulcrumPoint = Complex.FromRealNumber(23.0); // recommended
```

## 10. 기발한 이름은 피하라

> _재미난 이름보다 명료한 이름을 선택하라._

## 11. 한 개념에 한 단어를 사용하라

> _추상적인 개념 하나에 단어를 하나를 선택해 이를 고수한다. **일관성 있는 어휘**는 코드를 사용할 프로그래머가 반갑게 여길 선물이다._

- 메서드 이름은 독자적이고 일관적이어야 한다. 그래야 주석을 뒤져보지 않고도 프로그래머가 올바른 메서드를 선택한다.
- 이름이 다르면 독자는 당연히 클래스도 다르고 타입도 다르리라 생각한다. <br> 예) controller, manager, driver

## 12. 말장난을 하지 마라

> _한 단어를 두 가지 목적으로 사용하지 마라._

### 예시

- 지금까지 구현한 `add` 메서드를 두 개를 더하거나 이어서 새로운 값을 만든다고 가정하자.
- 새로 작성하는 메서드는 집합에 값 하나를 추가하려고 할때, 기존의 `add` 메서드와 맥락이 다르다.
- 그러므로 `insert`나 `append`라는 이름이 적당하다.

## 13. 해법 영역에서 가져온 이름을 사용하라

> _모든 이름을 문제 영역(domain)에서 가져오는 정책은 현명하지 못하다. 기술 개념에는 기술 이름이 가장 적합한 선택이다._

- 코드를 읽는 사람도 프로그래머이기 때문에 전산 용어, 알고리즘 이름, 패턴 이름, 수학 용어 등을 사용해도 괜찮다.

## 14. 문제 영역에서 가져온 이름을 사용하라

> _적절한 '프로그래머 용어'가 없다면 문제 영역에서 이름을 가져온다. 문제 영역 개념과 관련이 깊은 코드라면 문제 영역에서 이름을 가져와야 한다._

## 15. 의미 있는 맥락을 추가하라

> _스스로 의미가 분명한 이름이 아니라면 클래스, 함수, 이름 공간에 넣어 맥락을 부여한다. 모든 방법이 실패하면 마지막 수단으로 접두어를 붙인다._

### 예시 : 맥락이 불분명한 변수

아래의 코드에서 함수 이름은 맥락 일부만 제공하며, 알고리즘이 나머지 맥락을 제공한다. <br> → **독자가 맥락을 유추해야만 한다.** 그냥 메서드만 훑어서는 `number`, `verb`, `pluraModifier`라는 세 변수의 의미가 불분명하다.

```java
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluraModifier;
    if (count == 0) {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    } else if (count == 1) {
        number = "1";
        verb = "is";
        pluraModifier = "";
    } else {
        number = Integer.toString(count);
        verb = "are";
        pluraModifier = "s";
    }
    String guessMessage = String.format(
        "There %s %s %s%s", verb, number, candidate, pluralModifier
    );
    print(guessMessage);
}
```

### 개선하기

`GuessStaticsMessager` 라는 클래스를 만든 후 세 변수를 클래스에 넣는다. <br> → 세 변수의 맥락이 분명해진다. 맥락을 개선하면 함수가 쪼개기가 쉬워지므로 알고리즘도 좀 더 명확해진다.

```java
public class GuessStaticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char chandidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format(
            "There %s %s %s%s", verb, number, candidate, pluralModifier
        );
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

## 16. 불필요한 맥락을 없애라

> _일반적으로는 짧은 이름이 긴 이름보다 좋다. 단, 의미가 분명한 경우에 한해서다. 이름에 불필요한 맥락을 추가하지 않도록 주의한다._

- 나쁜 예) '고급 휘발유 충전소(Gas Station Deluxe)'라는 애플리케이션에서 모든 클래스 이름을 `GSD`로 시작

## References

- [https://velog.io/@bosl95/자바의-Static-Factory-Method](https://velog.io/@bosl95/%EC%9E%90%EB%B0%94%EC%9D%98-Static-Factory-Method)
