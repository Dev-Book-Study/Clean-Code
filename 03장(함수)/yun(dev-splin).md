# 3장. 함수

어떤 프로그램이든 가장 기본적인 단위가 함수입니다. 때문에 함수를 잘 만드는 법을 아는 게 좋습니다.





## 3-1. 작게 만들어라!

함수를 만드는 첫 번째 규칙은 `무조건 작게`입니다. 큰 함수를 작게 쪼개면서 적절한 이름을 붙여주면 코드를 이해하기 쉬워집니다.



### 블록과 들여쓰기

**if문, else문, while문 등에 들어가는 블록은 한 줄**이어야 합니다. 이 말은 중첩 구조가 생길만큼 함수가 커져서는 안된다는 의미입니다.

그러므로, 함수에서 **들여쓰기 수준은 1단이나 2단을 넘어서는 안됩니다.** 그래야 함수를 읽고 이해하기 쉬워지기 때문입니다.





## 3-2. 한 가지만 해라!

**함수는 한 가지 일만** 해야합니다. 함수가 여러가지 일을 한다면 각 기능을 적절한 함수이름을 붙여 추상화 한 후, 여러 단계로 나누어 수행하여야 합니다.

한 가지 작업만하는 함수는 섹션으로 나누기 어렵습니다. 즉, **섹션으로 나눌 수 있다면 각 섹션을 함수로 나누어야 합니다.**





## 3-3. 함수 당 추상화 수준은 하나로!

`추상화 수준`이란 무엇일까요?? **해당 코드를 읽으면서 파악할 수 있는 정보의 수준**을 말합니다. 즉, 해당 코드로 더 자세한 정보를 알 수 있으면 추상화 더 낮아진다고 할 수 있습니다.

- `높은 추상화 수준` : `getHtml()`이라는 함수는 HTML을 가져오는 것은 알겠는데, 어떤 것이랑 연관되어 있는지 알 수 없기 때문에 추상화 수준이 높다고 할 수 있습니다.
- `보통 추상화 수준` : `String pagePathName = PathParser.render(pagepath);`는 PathParser객체의 render 함수를 이용해 pagePathName을 가져올 수 있다는 정보를 유추할 수 있기 때문에 추상화 수준이 보통이라고 할 수 있습니다.
- `낮은 추상화 수준` : `.append("\n")`는 바로 어떤 의미인지 유추 가능하기 때문에 추상화 수준이 낮다고 할 수 있습니다.

**한 함수 내에 여러 추상화 수준을 섞으면 특정 표현이 높은 추상화 수준인지 낮은 추상화 수준인지 구분하기 어렵기 때문에 코드를 읽는 사람이 헷갈립니다.**



### 내려가기 규칙

코드는 위에서 아래로 이야기처럼 읽혀야 좋습니다. 한 함수 다음에는 추상화 수준이 한 단계 더 낮은 함수가 와야합니다. 즉, **위에서 아래로 코드를 읽으면 함수 추상화 수준이 한 번에 한 단계씩 낮아집니다.**

여기서 **핵심은 짧으면서도 한 가지 일만 하는 함수가 좋다는 것**입니다.





## 3-4. Switch 문

어쩔 수 없이 Switch 문이나 if/eles가 여러 개 이어지는 구문은 작게 만들기 어렵습니다. **본질적으로 switch 문은 N가지를 처리하기 때문에 switch 문을 완전히 피할 방법은 없습니다.**

하지만 각 switch 문을 `다형성`을 이용해 반복하지 않는 방법은 있습니다. 아래의 예시를 보면서 알아보겠습니다.



### 개선 전 코드

직원 유형에 따른 다른 값을 계산해 반환하는 함수입니다.

```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
  switch (e.type) {
    case COMMISIONED :
      return calculateCommissionedPay(e);
    case HOURLY :
      return calculateHourlyPay(e);
    case SALARIED :
      return calculateSalariedPay(e);
    default :
      throw new InvalidEmployeeType(e.type);
  }
}
```

위 함수는 여러가지 문제를 가지고 있습니다.

1. 함수가 김. 새 직원 유형을 추가하면 더 길어짐
2. type에 따라 다른 함수들을 호출하기 때문에 한 가지 작업만 수행하지 않습니다. (`SRP : 단일 책임 원칙 위반`)
3. 새 직원 유형을 추가할 때마다 코드를 변경해야 하기 때문에 `OCP : 개방 폐쇄 원칙`을 위반합니다.,,
4. 위 함수와 구조가 동일한 함수가 무한정 존재하게 됩니다.
   - Ex) 직원 유형에 따른 급여 지급 날짜 등...



### 개선 후 코드

위와 같은 문제들을 `추상 팩토리 패턴`을 이용해 해결할 수 있습니다. 간단히 설명하자면 **Employee를 상속하는 다양한 직원 유형을 만들고 Employee의 추상화 함수를 구현한 다양힌 직원 유형에서 각 함수들을 호출하는 방법**입니다.

**참고 링크**

- [추상 팩토리 패턴 - 낼름 디자인패턴 스터디](https://github.com/Nelmm/DesignPattern/blob/main/객체생성/2주차-추상팩토리/README.md)

```java
// 추상 클래스 Employee
public abstract class Employee {
  public abstract boolean isPayday();
  public abstract boolean calculatePay();
  public abstract boolean deliverPay(Money pay);
}

// Employee를 상속 받는 객체들을 생성할 수 있는 EmployeeFactory 인터페이스
public interface EmployeeFactory {
  public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}

// EmployeeFactory를 구현한 클래스
public class EmployeeFactoryImpl implements EmployeeFactory {
  public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
    switch (r.type) {
      case COMMISIONED :
        return new CommissionedEmployee(r);
      case HOURLY :
        return new HourlyEmployee(r);
      case SALARIED :
        return new SalariedEmployee(r);
      default :
        throw new InvalidEmployeeType(r.type);
    }
  }
}
```

위와 같이 Employee를 구현한 다양한 직원 유형 객체를 생성해줌으로써, 개선 전의 문제들을 해결할 수 있습니다. (물론.. 새 직원 유형을 추가한다면 똑같이 함수가 길어지긴 합니다)

저자는 **다형적 객체를 생성하는 코드의 switch 문 한 번만은 참아준다**고 합니다.





## 3-5. 서술적인 이름을 사용하라!

좋은 이름이 주는 가치는 아무리 강조해도 지나치지 않습니다. **길고 서술적인 이름이 짧고 어려운 이름보다 좋습니다.** (함수가 작고 단순할수록 서술적인 이름을 고르기 쉬워집니다.)

특히, **이름을 붙일 때는 일관성이 있어야 합니다.** 예를 들어, `includeA, includeB`가 있다면 `includeC`도 있을 수 있다고 짐작할 수 있기 때문입니다.





## 3-6. 함수 인수

함수에서 가장 이상적인 인수 개수는 0개(무항) 입니다. **인수 개수(항)가 늘어날 수록 함수를 이해하기 어렵게 만들기 때문에 4개 이상은 특별한 이유가 있어도 사용하지 않는 게 좋습니다.**

테스트 관점에서 본다면, 갖가지 인수 조합으로 함수를 검증하게 되는데, 이 때 인수가 늘어나면 늘어날수록 조합이 복잡해집니다.

**최선은 입력 인수가 없는 경우이며, 차선은 입력 인수가 1개뿐인 경우**입니다.



### 많이 쓰는 단항 형식

함수에 인수 1개를 넘기는 이유로 가장 흔한 경우는 아래의 두 가지 입니다.

- 인수에 질문을 던지는 경우
  - ex) boolean fileExists("MyFile")
- 인수를 변환해 결과를 반환하는 경우
  - ex) InputStream fileOpen("MyFile")

드물게 사용하지만 이벤트 함수도 단항 함수 형식으로 이루어져있습니다. 다만, 이벤트 함수는 이벤트라는 사실이 코드에 명확히 드러나야 합니다.

위의 경우가 아니라면 단항 함수는 가급적 피합니다.



### 플래그 인수

플래그 인수는 함수가 한꺼번에 여러 가지를 처리하는 것을 유추할 수 있기 때문에 쓰지 않는 게 좋습니다.



### 이항 함수

인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵고 헷갈리기 쉽습니다. **Point p = new Point(0, 0) 처럼 인수 2개가 한 값을 자연스럽게 표현하는 경우가 아니라면 불가피한 경우가 아닌 경우는 사용을 지양**해야합니다.



### 삼항 함수

인수가 3개인 함수는 인수가 2개인 함수보다 훨씬 더 이해하기 어렵습니다. 때문에 삼항 함수를 만들 때는 신중히 고려해야합니다.



### 인수 객체

인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 살펴보아야 합니다. 

`Circle makeCircle(double x, double y, double radius) -> Circle makeCircle(Point center, double radius)`

위와 같이 이항 함수 부분에서 Point 부분을 예로 들 수 있습니다.

**객체를 생성해 인수를 줄이는 방법이 눈속임 처럼 보일 수 있지만, x,y를 묶었듯이 변수를 넘기려면 이름을 붙여야 하기 때문에 결국은 개념을 표현하게 됩니다.**



### 인수 목록

`String.format` 처럼 때로는 인수 개수가 가변적인 함수도 필요합니다. 이 때, **가변 인수들은 List형 인수 하나로 취급**할 수 있습니다.

`public String format(String format, Object... args)`

위의 **String.format의 구조를 보면 이항 함수라는 사실**을 알 수 있습니다. 하지만 가변 인수를 포함해 인수 개수가 3개가 넘어가면 안됩니다.



### 동사와 키워드

좋은 함수 이름을 정할 때, 단항 함수라면 함수와 인수가 동사/명사 쌍을 이루는 게 좋습니다. 그렇게 하면 `write(name)`처럼 이해하기 쉬운 함수가 됩니다. `writeField(name)`처럼 name이 필드라는 사실을 분명히 드러나게 해도 좋습니다.

또, `assertEquals`보다 `assertExpectedEqualsActual(expected, actual)`처럼 함수 이름에 키워드를 넣음으로써 인수 순서를 기억하지 않아도 되게 만드는 방법도 좋습니다.





## 3-7. 부수 효과를 일으키지 마라!

**함수 이름으로 명시된 한 가지 일 외에 다른 일이 포함되면 부수 효과(Side Effect)가 발생하는 경우를 조심**해야 합니다. 함수가 한 가지일을 하게 만드는 게 좋지만 어쩔 수 없다면 차라리 함수 이름에 다른 일에 대한 것을 분명히 명시하는 게 낫습니다.



### 출력 인수

일반적으로 인수를 함수의 입력, 반환값을 출력으로 생각하기 때문에 인수를 출력으로 사용하지 않게 조심해야 합니다.





## 3-8. 명령과 조회를 분리하라!

함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 하지, 둘 다 하면 안 됩니다.

예를 들어, 속성을 찾아 값을 value로 설정한 후 성공하면 true, 실패하면 false를 반환하는 함수 `public boolean set(String attribute, String value)`가 있다고 했을 때,

`if (set("username", "unclebob")) ...` 같은 경우는`username 속성이 unclebob으로 설정되어 있다면...` 으로 오해할 가능성이 있습니다. 때문에 set이 설정과 체크 두 개의 일을 하는 것 보다 `attributeExists, setAttribute`로 일을 나누는 것이 낫습니다.



### 나의 생각

이 부분을 읽고 보통 웹에서 데이터 update를 할 때 사용하는 update 함수들은 인자로 받은 값을 update 후, 성공과 실패를 확인할 수 있게 값을 반환해주는 경우가 생각났습니다.

저자의 말대로라면 값을 update하고 다시 DB를 조회하여 값을 확인해야 하는데, 이런 경우는 DB를 한 번 더 조회하는 것이 너무 비효율적이기 때문에 어쩔 수 없는 경우라고 생각합니다.





## 3-9. 오류 코드보다 예외를 사용하라!

함수에서 오류 코드를 사용하게 되면 **오류 코드에 관련된 처리를 추가적으로 해주어야 합니다.** 때문에 오류 코드 대신 예외를 사용하면 오류 처리 코드가 분리되므로 코드가 더 깔끔해집니다.



### Try/Catch 블록 뽑아내기

try/catch 블록은 정상 동작과 오류 처리 동작이 섞이게 됩니다.(정상 동작 할 수도 있고, 오류가 날 수도 있어서 이렇게 말한 거 같음..) 때문에 try/catch 블록을 아래와 같이 별도 함수로 분리하는 편이 좋습니다.

```java
// try catch 블록을 포함하는 delete 함수
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    } catch (Exception e) {
        logError(e);
    }
}

// 실질 적인 로직을 모아 놓은 deletePageAndAllReferences 함수
private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}

// 로그를 보여주는 logError 함수
private void logError(Exception e) {
    logger.log(e.getMessage());
}
```



#### 나의 생각

사실 뭔가 애매모호한 느낌이 드는 부분이긴 합니다. 제 생각으로는 `delete` 함수를 읽을 때 `deletePageAndAllReferences 함수에서 오류가 발생할 수도 있구나`라고 한 번에 파악하기 쉽게 만들기 위함이 아닐까 싶습니다.



### 오류 처리도 한 가지 작업이다.

**함수는 한 가지 일만해야 하는데, 오류 처리도 한 가지 작업에 속합니다.** 때문에 위의 예시와 같이 오류를 처리하는 함수는 오류만 처리해야 한다고 합니다.



### Error.java 의존성

만약 자바에서 Error 코드를 위한 Enum을 사용한다고 했을 떄, 다른 클래스에서 Error enum을 사용해야 하므로 Error enum이 변경된다면, Error enum을 사용하는 클래스 전부를 다시 컴파일하고 다시 배치해야 합니다. 하지만 **예외를 사용한다면 재컴파일/재배치 없이도 새 예외 클래스를 추가해서 사용할 수 있기 때문에 예외를 권장하는 것**입니다.



### 오류 코드와 예외에 대한 나의 생각

오류 코드 처리를 위한 작업이 생기기 때문에 예외 처리가 더 좋은 것에는 동감합니다. 하지만 웹 프로그래머 입장에서는 백엔드에서 프론트엔드와의 통신을 위한 오류 코드는 백엔드와 프론트엔드 간의 약속이기 때문에 괜찮다고 생각합니다.





## 3-10. 반복하지 마라!

많은 원칙과 기법들이 중복을 없애거나 제어할 목적으로 나왔습니다. **중복이 발생한다면 코드 길이가 늘어날 뿐 아니라 중복 코드가 변하면 중복된 부분 모두를 수정해야 하기 때문에 오류가 발생할 확률도 높아지게 됩니다.**

`구조적 프로그래밍`, `AOP(Aspect Oriented Programming)`, `COP(Component Oriented Programming)` 모두 어떤 면에서는 중복 제거 전략입니다.

> **AOP(Aspect Oriented Programming)** : 관점 지향 프로그래밍이라고 합니다. 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어서 보고 그 관점을 기준으로 각각 묘듈화 하는 것을 말합니다.
>
> **COP(Component Oriented Programming)** : Vue, React, Angular 등.. 에서 사용하는 컴포넌트 지향 프로그래밍입니다. 프론트 엔드에서 반복되는 요소들을 컴포넌트로 분리하여 애플리케이션을 더 빠르게 구축할 수 있게 해주는 것을 말합니다.





## 3-11. 구조적 프로그래밍

모든 함수와 함수 내 모든 블록에는 입구와 출구가 하나만 존재해야 한다고 합니다.(`입/출구 규칙`) 때문에  **break, continue는 사용해선 안 되고 goto는 절대로 사용하지 말라고 합니다.**

하지만 함수를 작게 만든다면 때로는 `입/출구 규칙` 보다 아래의 예시 처럼 `return, break ,continue`를 사용하는 게 의도를 표현하기 쉬워지기 때문에 적절히만 사용한다면 괜찮다고 합니다.

```java
public boolean findNum(int target) {
	for(int num : nums) {
		if (num == target) {
			return true;	// 해당 숫자를 찾는다면, 함수를 종료하겠다는 의도를 확실히 표현
		}
	}
	
	return false;
}
```





## 3-12. 함수를 어떻게 짜죠?

프로그램을 만들 때 처음부터 함수를 분리하는 것은 매우 어렵습니다. 때문에 **처음에는 길고 복잡하게 만들더라도 단위 테스트를 통해 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거하는 과정이 필요**하다고 합니다.





## 3-13. 3장을 마치며..

마지막 `3-12. 함수를 어떻게 짜죠?` 부분에서 나왔던 것처럼 함수를 처음부터 완벽하게 분리해서 작성하는 것은 거의 불가능하다고 생각합니다. 때문에 지금까지 알아본 3장을 토대로 **항상 좋은 함수를 만들기 위한 고민과 노력을 끊임없이 해야하고 단위 테스트를 통해 검증하고 리팩토링하는 과정이 필요**하다고 생각됩니다.



---

참고 : Clean Code - 로버트 C. 마틴



