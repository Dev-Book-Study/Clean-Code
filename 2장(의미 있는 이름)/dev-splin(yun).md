# 2장. 의미 있는 이름

소프트웨어에서 이름은 어디에서나 쓰입니다. 이름을 잘 지으면 여러모로 편하기 때문에 이 장에서는 이름을 잘 짓는 간단한 규칙 몇 가지를 소개한다고 합니다.





## 2-1. 의도를 분명히 밝혀라

**좋은 이름을 지으려면 시간이 걸리지만 좋은 이름으로 절약하는 시간이 훨씬 더 많습니다.** 때문에 변수나 함수 그리고 클래스 이름은 다음과 같은 굵직한 질문에 모두 답해야 합니다.

- **변수(혹은 함수나 클래스)의 존재 이유는?**
- **수행 기능은?**
- **사용 방법은?**

만약 따로 주석이 필요하다면 의도를 분명히 드러내지 못했다는 말입니다.



### 코드

```java
public List<int[]> getThen() {
  List<int[]> list1 = new ArrayList<int[]>();
  
  for (int[] x : theList)
    if (x[0] == 4)
      list1.add(x);
  
  return list1;
}
```

위의 코드는 복잡하진 않지만 짐작하기는 어렵습니다. 그 이유는 읽는 사람이 다음과 같은 정보를 안다고 가정하기 때문입니다.

1. theList에 무엇이 들었는가?
2. theList에서 0번째 값이 어째서 중요한가?
3. 값 4는 무슨 의미인가?
4. 함수가 반환하는 리스트 list1을 어떻게 사용하는가?

`지뢰찾기` 게임을 만든다고 가정하고 위의 코드를 조금 더 파악하기 쉽게 바꿔보겠습니다.

```java
public List<int[]> getFlaggedCells() {
  List<int[]> flaggedCells = new ArrayList<int[]>();
  
  for (int[] cell : gameBoard) // theList -> gameBoard
    if (cell[STATUS_VALUE] == FLAGGED) // 0을 STATUS_VALUE이라는 상수로 4를 FLAGGED라는 상수로 이름을 붙여줌
      flaggedCells.add(cell);
  
  return flaggedCells;
}
```

**각 개념에 이름만 붙여도 코드가 상당히 나아집니다.** 여기서 아래와 같이 int배열을 사용하는 대신 `Cell` 클래스를 만들고 `isFlagged`라는 좀 더 명시적인 함수를 사용해 상수 값들을 감춰도 좋습니다.

```java
// int[]을 Cell로 클래스로 변경
public List<Cell> getFlaggedCells() {
  List<Cell> flaggedCells = new ArrayList<Cell>();
  
  for (Cell cell : gameBoard)
    if (cell.isFlagged) // isFlagged를 만들어 상수 값을 감춤
      flaggedCells.add(cell);
  
  return flaggedCells;
}
```





## 2-2. 그릇된 정보를  피하라

- **약어를 사용하지 않습니다.**
  - ex) 직각삼각형의 빗변(hypotenuse)를 hp라 표현하는 등...
- **예약어를 사용하지 않습니다.**
  - ex) 실제 List가 아니라면 여러 여러 계정 그룹을 `accountList`라 명명하지 않고 `accountGroup` 등으로 표현
  - List인 경우라도 컨테이너 유형을 이름에 넣지 않는 편이 바람직하다고 합니다.
- **서로 흡사한 이름을 사용하지 않도록 주의합니다.**





## 2-3. 의미 있게 구분하라

동일한 범위 안에서는 다른 두 개념에 같은 이름을 사용하지 못합니다. 즉, **이름이 달라야 한다면 의미도 달라져야 한다는 것**입니다. `불용어`를 추가하는 방식은 적절하지 못합니다. **불용어는 a, an, the와 같이 분석에 큰 의미가 없는 단어를 지칭하기 때문에 아무런 정보도 제공하지 못합니다.** 예시는 아래와 같습니다.

- a1, a2 등등...
- ProductInfo 혹은 ProductData
  - `Info, Data`는 a, an, the와 마찬가지로 의미가 불분명하다고 합니다.

그렇다고 a, the 같은 접두어를 사용하지 말라는 소리는 아닙니다. 의미가 분명히 다르다면 사용해도 무방합니다.

### 나의 생각

기존에 불용어를 사용하면 안된다는 것은 알았지만, `Info, Data`는 처음 알았는데, 충분히 오해할만한 소지가 있다고 생각합니다.





## 2-4. 발음하기 쉬운 이름을 사용하라

발음하기 어려운 이름은 사람마다 발음하는 방식도 다르고 토론을 할 때나 새로운 개발자에게 설명이 필요하기 때문에 좋지 않습니다.





## 2-5. 검색하기 쉬운 이름을 사용하라

`MAX_CLASSES_PER_STUDENT`는 검색으로 찾기 쉽지만, 숫자 7은 파일 이름이나 수식이 모두 검색되기 때문에 검색으로 찾기가 쉽지 않습니다. 아래의 두 예제를 비교해 보겠습니다.

```java
for (int j=0; j<34; j++) {
	s += (t[j]*4)/5;
}
```

첫 번째 예제입니다. 각 변수가 어떤 걸 의미하는지도 모르겠고 검색할 때도 너무 힘들 것 같습니다.

```java
int realDayPerIdealDay = 4;
const int WORK_DAYS_PER_WEEK = 5;
int sum = 0;
for (int j=0; j < NUMBER_OF_TASKS; j++) {
  int realTaskDays = taskEstimate[j] * realDayPerIdealDay;
  int realTaskWeeks = (realTaskDays / WORK_DAYS_PER_WEEK);
  sum += realTaskWeeks;
}
```

두 번째 예제입니다. 코드가 조금 길어졌지만, 검색하기도 용이하고 변수명으로 어떤 걸 의미하는지 파악할 수 있습니다.





## 2-6. 인코딩을 피하라

이름에는 인코딩할 정보가 아주 많은데, 여기에 유형이나 범위 정보까지 인코딩에 넣으면 그만큼 이름을 해독하기 어려워집니다. 아래의 몇 가지 예시를 보겠습니다.

- `헝가리식  표기법`
  - 과거에는 컴파일러가 타입을 점검하지 않았으므로 프로그래머에게 타입을 기억할 단서가 필요했습니다.
  - 현재는 **IDE가 코드를 컴파일하지 않고도 타입 오류를 감지할 정도로 발전했기 때문에 헝가리 표기법이나 기타 인코딩 방식은 오히려 방해**가 됩니다.
- `멤버 변수 접두어`
  - 멤버 변수를 다른 색상으로 표시하거나 눈에 띄게 보여주는 IDE가 많기 때문에 굳이 사용할 필요가 없습니다.
- `인터페이스 클래스와 구현 클래스`
  - 인터페이스와 구현 클래스를 작성할 때 인코딩이 필요한 경우가 있는데, 이런 경우에는 인터페이스에 I를 붙여주거나, 구현 클래스에 Imp를 붙여줄 수 있는데 저자는 Imp가 더 좋다고 합니다.
  - ex) 도형을 생성하는 `ShapeFactory`와 `ShapeFactoryImp`



### 나의 생각

인터페이스 클래스와 구현 클래스 부분은 스타일 차이라고 생각은 되지만, **Imp가 I를 붙이는 것 보다는 가독성이 좋다고 생각**합니다.





## 2-7. 자신의 기억력을 자랑하지 마라

코드를 분석하는 사람이 코드를 읽으면서 변수 이름을 자신이 아는 이름으로 변환해야 한다면 그 변수 이름은 바람직하지 못합니다. 예를 들면, **r이라는 변수가 호스트와 프로토콜을 제외한 소문자 URL이라는 사실을 자기만 기억하고 사용하지 말라는 의미**입니다. (루프에서 사용하는 단일 이름으로 된 변수는 괜찮다고 합니다. 가독성이 좋지 않은 소문자 L 제외)





## 2-8. 클래스 이름

클래스 이름과 객체 이름은 명사나 명사구가 적합합니다. `Customer, WikiPage, Account` 등등..

`Manager, Processor, Data, Info` 같은 단어를 피해야 합니다. 



### 나의 생각

`Data, Info`는 너무 추상적이니 이해는 가지만 `Manager, Processor`는 좀 애매모호할 수 있지만, 객체를 관리한다는 의미로 `Manager`를 붙일 수 있기 때문에 피해야 한다고 하는 것 같습니다.



## 2-9. 메서드 이름

- 메서드 이름은 동사나 동사구가 적합합니다. `postPayment, deletePage` 등등...

- `접근자, 변경자, 조건자`는 javabean 표준에 따라 값 앞에 `get(접근자), set(변경자), is(조건자)`를 붙입니다.
- 생성자를 중복정의(overload) 할 때는 정적 팩토리 메서드를 사용합니다. (메서드는 인수를 설명하는 이름을 사용합니다.)
  - `Complex fulcrumPoint = Complex.FromRealNumber(23.0);`의 코드가 아래의 코드보다 좋습니다.
  - `Complex fulcrumPoint = new Complex(23.0);`
- 생성자 사용을 제한하려면 해당 생성자를 private으로 선언합니다.





## 2-10. 기발한 이름을 피하라

당연한 소리긴 하지만.. 이름이 너무 기발하면 알아듣지 못 할수도 있기 때문에 농담식의 이름보다는 의도를 분명하고 솔직하게 표현하는 것이 좋습니다.





## 2-11. 한 개념에 한 단어를 사용하라

추상적인 개념 하나에 단어 하나를 선택해 이를 고수합니다. 예를 들어, 똑같은 메서드를 클래스마다 `fetch, retrieve, get`으로 제각각 부르면 혼란스럽습니다. 마찬가지로 동일 코드 기반에 `Controller, Manager, Driver`같이 비슷한 의미를 섞어 쓰면 혼란스럽습니다.





## 2-12. 말장난을 하지 마라

한 단어를 두 가지 목적으로 사용하지 말아야 합니다. 다른 개념에 같은 단어를 사용한다면 그것은 말장난에 불과합니다.

여러 클래스에 `add`라는 메서드가 있을 때, 모든 `add` 메서드의 매개변수와 반환값이 의미적으로 똑같다면 문제 없습니다. 하지만 종종 프로그래머들은 같은 맥락이 아닌데도 `일관성`을 고려해 `add`라는 단어를 선택합니다. 아래와 같은 두 개의 `add` 메서드가 있다고 가정하겠습니다.

- `기존의 add 메서드` : 기존 값 두 개를 더하거나 이어서 새로운 값을 만드는 메서드
- `두 번째 add 메서드` : 집합에 값을 하나 추가하는 메서드

`두 번째 add 메서드`를 add라 불러도 괜찮을까요?? 일관성을 위해 add라 부를수도 있지만, **두 번째 add 메서드는 기존의 add 메서드와 맥락이 다르기 때문에 insert나 append라는 이름이 적합**합니다. 즉, **두 번째 메서드를 add라고 부른다면 말장난인 것**입니다.





## 2-13. 해법 영역에서 가져온 이름을 사용하라

모든 이름을 도메인 영역에서 가져오는 정책은 같은 개념을 다른 이름으로 이해할 수 있어 매번 고객에게 의미를 물어봐야 하기 때문에 좋지 않습니다. **코드를 읽을 사람도 프로그래머이기 때문에 전산 용어, 알고리즘 이름, 패턴 이름, 수학 용어등을 사용해도 괜찮습니다.** 프로그래머에게 익숙한 기술 개념은 아주 많기 때문에 **기술 개념에는 기술 이름이 가장 적합한 선택**입니다.





## 2-14. 문제(도메인) 영역에서 가져온 이름을 사용하라

하지만 `적절한 프로그래머 용어`가 없다면 도메인 영역에서 이름을 가져옵니다. 그러면 **코드를 보수하는 프로그래머가 도메인 전문가에게 의미를 물어 파악**할 수 있습니다. 다만 **우수한 프로그래머와 설계자라면 해법 영역과 문제(도메인) 영역을 구분할 줄 알아야 합니다.**





## 2-15. 의미 있는 맥락을 추가하라

모든 이름은 스스로 의미를 가지지만 대다수 이름은 그렇지 못합니다. 때문에 **클래스, 함수, 이름 공간에 넣어 맥락을 부여합니다. 이 모든 방법이 실패했을 때, 마지막 수단으로 접두어를 붙입니다.**
예를 들어, 주소에 속하는 `addrFirstName, addrLastName, addrState`가 있습니다. 접두어 `addr`을 추가하므로써 코드를 읽는 사람이 주소에 속한다는 것을 알 수있습니다. 물론 Adress라는 클래스를 생성하면 더 좋습니다. 아래의 두 예제를 비교하면서 좀 더 확실히 알아보겠습니다.



### 예제1

해당 예제는 메서드 이름에서 통계 추측 메세지를 출력한다는 것을 알수는 있지만, 함수를 끝까지 읽어보고 나서야 `number, verb, pluralModifier`라는 변수 세 개가 `통계 추측 메세지`에 사용된다는 사실을 알 수 있습니다.

```java
private void printGuessStatistics(char candidate, int count) {
	String number;
	String verb;
	String pluralModifier;
	
	if (count == 0) {
		number = "no";
		verb = "are";
        pluralModifier = "s";
	} else if (count == 1) {
        number = "1";
        verb = "is";
        pluralModifier = "";
	} else {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
    
    String guessMessage = String.format(
        "There %s %s %s%s", verb, number, candidate, pluralModifier
    );
}
```



### 예제2

위의 함수를 작은 조각으로 쪼개어 세 변수를 가지는 `GuessStatisticsMessage`라는 클래스를 만듭니다. 그러면 **세 변수는 맥락이 분명해지고 함수를 쪼개기가 쉬워지므로 알고리즘도 좀 더 명확**해집니다.

```java
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;
    
    public String make(char candidate, int count) {
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





## 2-16. 불필요한 맥락을 없애라

`(Gas Station Deluxe)고급 휘발유 충전소`라는 애플리케이션을 짠다고 가정합니다. 

만약 모든 클래스 이름을 `GSD`로 시작한다면 IDE에서 G를 입력하고 자동 완성 키를 누르면 IDE는 모든 클래스를 열거하게 되므로 좋지 않습니다. 또 다른 프로그램에서 해당 객체를 사용하는 경우가 생기면 `GSD`가 붙는 객체는 사용하기에 적절하지 않습니다.

일반적으로는 의미가 분명한 경우에 짧은 이름이 긴 이름보다 좋습니다. 때문에 불필요한 맥락을 제거해야하고 이름을 정할 때 의미가 분명하게 만들어야합니다.





## 2-17. 2장을 마치며...

우리들 대다수는 자신이 작성한 클래스 이름과 메서드 이름을 모두 암기하지 못합니다. 때문에 **우리는 문장이나 문단처럼 읽히는 코드나 표나 자료 구조처럼 읽히는 코드를 작성하는데 집중**해야 합니다. 만약 코드 개선 노력과 마찬가지로 이름 역시 나름대로 바꿨다가는 누군가의 질책을 받을지도 모르지만 그렇다고 **코드를 개선하려는 노력을 중단해서는 안 됩니다.**



---

참고 : Clean Code - 로버트 C. 마틴

