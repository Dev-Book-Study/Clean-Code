## 3장 함수(Function)
---

<details>
  <summary>목록 3-1 HtmlUtil.java (FitNesse 20070619)</summary>

  ```java
  public static String testableHtml(
    PageData pageData;
    boolean includeSuiteSetup
  ) throws Exception {
    WikiPage wikiPage = pageData.getWikiPage();
    StringBuffer buffer = new String Buffer();
    if (pageData.hasAttribute("Test")) {
      if (includeSuiteSetup) {
        WikiPage suiteSetup = PageCrawlerImpl.getInheritedPage(
          SuiteResponder.SUITE_SETUP_NAME, wikiPage
        );
        if (suiteSetup != null) {
          WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
          String pagePathName = PathParser.render(pagePath);
          buffer.append("!include - setup .")
                .append(pagePathName)
                .append("\n");
        }
      }
      WikiPage setup = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
      if (setup != null) {
        WikiPagePath = setupPath = wikiPage.getPageCrawler().getFullPath(setup);
        String setupPathName = PathParser.render(setupPath);
        buffer.append("!include - setup .")
              .append(setupPathName)
              .append("\n")
      }
    }
    buffer.append(pageDate.getContent());
    if (pageData.hasAttribute("Test")) {
      WikiPage teardown = PageCrawlerImpl.getInheritedPage("Teardown", wikiPage);
      if (teardown != null) {
        WikiPage.getPageCrawler().getFullPath(teardown);
        String tearDownPathName = PathParser.render(tearDownPath);
        buffer.append("\n")
              .append("!include -teardown .")
              .append(tearDownPathName)
              .append("\n")
      }
      if (includeSuiteSetup) {
        WikiPage suiteTeardown = PageCrawlerImpl.getInheritedPage(
          SuiteResponder.SUITE_TEARDOWN_NAME,
          wikiPage
        );
        if (suiteTeardown != null) {
          WikiPagePath pagePath = suiteTeardown.getPageCrawler().getFullPath (suiteTeardown);
          String pagePathName = PathParser.render(pagePath);
          buffer.append("!include - teardown .")
                .append(pagePathName)
                .append("\n");
        }
      }
    }
    pageData.setContent(buffer.toString());
    return pageData.getHtml();
  }


  ```
> 이 코드의 문제는 추상화 수준이 높을 뿐만 아니라, 긴 코드, 여러겹의 if문, 이상한 문자열 등으로 코드를 이해하기 어렵다..

</details>


<details>
<summary>목록 3-2 HtmlUtil.java(리팩터링 버전)</summary>

```java
public static String renderPageWithSetupsAndTeardowns(
  PageData pageData, boolean isSuite
) throws Exception {
  boolean isTestPage = pageData.hasAttribute("Test");
  if (isTestPage) {
    WikiPage testPage = pageData.getWikiPage();
    StringBuffer newPageContent = new StringBuffer();
    includeSetupPages(testPage, newPageContent, isSuite);
    newPageContent.append(pageData.getContent());
    includeTeardownPages(testPage, newPageContent, isSuite);
    pageData.setContent(newPageContent.toString());
  }
  return pageData.getHtml();
}
```
> 100% 이해하기 어렵지만, 목록 3-1에서는 파악하기 어려웠던 정보를 쉽게 드러나는 것처럼 보인다. 그렇다면 그 이유는 무엇일까?

</details>

---

### 1. 작게 만들어라!

- 함수를 만드는 첫 번째 규칙은 __작게!__, 두 번째는 __더 작게!__

- 각 함수가 하나의 이야기 하나를 표현하듯이, 그 역할을 명백하게 구성하자.

- 얼마나 짧게라는 기준은 `목록 3-2` 보다 짧게, `목록 3-3`만큼 줄여보자.
  <details>
  <summary>목록 3-3 HtmlUtil.java (re-refactored)</summary>

  ```java
  public static String renderPageWithSetupsAndTeardowns(
    pageData pageData, boolean isSuite) throws Exception {
      if (isTestPage(pageData)) includeSetupAndTeardownPages(pageData, isSuite);
      return pageData,getHtml();
    }
  ```

  </details>
  
- `if / else / while` 등에 들어가는 블록은 한 줄로 줄여보자!
  
  - 이런 블록문에서 함수를 호출하자! 그렇다면, 바깥을 감싸는 함수(enclosing function)가 작아질 뿐 아니라 호출하는 함수의 이름이 적절할 경우 코드를 이해하기도 쉬워진다.
  
  ```java
  // 3-2
  boolean isTestPage = pageData.hasAttribute("Test");
  if (isTestPage) {		// # 변경전-1
    WikiPage testPage = pageData.getWikiPage();		// # 변경전-2
    StringBuffer newPageContent = new StringBuffer();		// # 변경전-2
    includeSetupPages(testPage, newPageContent, isSuite);		// # 변경전-2
    newPageContent.append(pageData.getContent());		// # 변경전-2
    includeTeardownPages(testPage, newPageContent, isSuite);	// # 변경전-2
    pageData.setContent(newPageConent.toString());		// # 변경전-2
  }
  // 3-3
  if (isTestPage(pageData)) {		// # 변경후-1
    includeSetupAndTeardownPages(pageData, isSuite);	// # 변경후-2
  }
  ```
  
  - 이 말은, 중첩 구좌 생길만큼 함수가 커져서는 안된다는 의미이다. ➡ 함수에서 들여쓰기 수준은 1단이나 2단을 넘어서지 말도록 하자.

---

### 2. 한가지만 해라!

- `목록 3-1`은 여러가지를 처리하느라 아주 바쁘다. 반면 `목록 3-3`은 한 가지만 처리한다.
- __함수는 한 가지를 해야한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.__ ➡ 여기서 "한 가지"란?, 지정된 함수 아래 추상화 수준이 하나라는 의미.
- `목록 3-3`은 지정된 함수 이름 아래에서 추상화 수준이 하나이다.
  - 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 한다. ➡ `목록 3-1` 은 추상화 수준에서 여러 단계를 처리, `목록 3-2` 은 추상화 수준이 둘
- 이 "한 가지"를 판단하는 또다른 방법 한가지는, 단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출가능 하다면 그 함수는 한 가지가 아닌 "여러 가지"를 수행하는 것이다.

---

### 3. 함수 당 추상화 수준은 하나로!

- `2.한가지만 해라!` 라는 규칙에서 처럼, 함수가 "한 가지" 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일 해야 한다.
- 추상화 수준이 섞이면 섞일 수록, 코드를 읽는 사람이 헷갈린다. ➡ 특정 표현이 '근본 개념'인지? '세부사항'인지? 햇갈리기 때문.
  - 만약 여기서, 근본 개념과 세부사항이 섞이기 시작하면 사람들이 함수에 세부사항을 점점 더 추가한다.
- __위에서 아래로 코드 읽기: 내려가기 규칙__
  - 코드는 위에서 아래로 이야기처럼 읽혀야 좋다.
  - 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다. ➡ 위에서 아래로 프로그램을 읽으면 함수 추상화 수준이 한 단계씩 낮아진다. (_내려가기 규칙_)
  - 즉, 일련의 문단을 읽듯이 프로그램이 읽혀야 한다는 의미..!
  - 그러나, 추상화 수준이 하나인 함수를 구현하기란 쉽지 않으며 많은 사람이 곤란을 겪는다. ➡ 그렇지만, 매우 중요한 규칙이며 핵심은 '짧으면서도 한 가지만'하는 함수이다.

---

### 4. Switch 문

- `switch` 문은 작게 만들기 어렵다.  (여럿 이어지는`if / else` 문 또한 포함) ➡ 본질적으로 switch문은 N가지를 처리하도록 되어있기 때문이다. 그렇기에 이 switch문을 완전히 피할 수 있는 방법은 없다.
- 그러나, 각 switch문을 저차원 클래스에 숨기고 반복하지 않는 방법이 있다. ➡ __다형성(polymorphism)__ 을 이용하는 방법.

<details>
<summary>목록 3-4 Payroll.java</summary>

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

> 이 코드에는 문제점이 존재한다.
> 1. 함수가 길다. 새 직원 유형을 추가하면 더 길어진다.
> 2. 한 가지 작업만 수행하지 않는다.
> 3. SRP(Single Responsibility Principle) 위반. 코드를 변경할 이유가 여럿이기 때문.
> 4. OCP(Open Closed Principle) 위반. 새 직원 유형을 추가할 때마다 코드를 변경하기 때문.
> 5. 위 함수와 구조가 동일한 함수가 무한정 존재할 가능성이 있다.

</details>

<details>
<summary>목록 3-5 Employee and Factory</summary>

```java
public abstract class Employee {
  public abstract boolean isPayday();
  public abstract Money calculatePay();
  public abstract void deliverPay(Money pay);
}
//
public interface EmployeeFactory {
  public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
//
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

</details>

- 로버트 C.마틴은 일반적으로 다형성 객체를 생성하는 코드안에서 단 한 번의 switch문을 허락한다고 한다. ➡ 상속 관계로 숨긴 후에 절대로 다른 코드에 노출하지 않는다는 것!

---

### 5. 서술적인 이름을 사용하라!

- 함수가 하는 일을 좀 더 잘 표현하는 것이 훨씬 좋은 이름이다.
- 좋은 이름이 주는 가치는 아무리 강조해도 지나치지 않는다.

> "코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다." - 워드

- 한 가지만 하는 작은 함수 + 좋은 이름의 함수 = 원칙달성의 절반 성공
- 함수가 작고 단순할 수록 서술적인 이름은 고르기도 쉬워진다.
- 이름이 길어도 된다. 서술적인 이름이 짧고 어려운 이름보다 좋으며, 길고 서술적인 이름이 길고 서술적인 주석보다 좋다.
- 함수 이름을 정할 때는, 여러 단어가 쉽게 읽히는 명명법을 사용한다. 다음에는, 여러 단어를 사용해 함수 기능을 잘 표현하는 이름을 선택한다.
- 이름을 정하는 시간이 오래걸려도 좋다. 이런저런 이름을 시도한 후 최대한 서술적인 이름을 골라보면 좋겠다.
- 서술적인 이름을 사용하면 개발자 머릿속에서도 설계가 뚜렷해지므로 코드를 개선하기 쉬워진다.
- 이름을 붙일 때는 일관성이 있어야 한다. 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용한다.
  - `includeSetupAndTeardownPages` / `includeSetupPages` / `includeSuiteSetupPage` / `includeSetupPage` 등이 좋은 예.
- 문제가 비슷하면 이야기를 순차적으로 풀어가기도 쉬워진다. ➡ 위의 예시를 보면 짐작할 수 있는 질문들을 떠올릴 수도 있다.

---

### 6. 함수 인수

- 함수에서 이상적인 인수 개수는 0개(무항)이다. ➡ 다음은 1개(단항), 그 다음은 2개(이항), 3개(삼항)은 피하는 게 좋고, 4개(다항)은 특별한 이유가 필요하다. 특별한 이유가 있어도 사용하지 말도록 하자.
- 코드를 읽는 사람에게 `includeSetupPageInto(newPageContent)` 보다 `includeSetupPage()` 가 더 이해하기 쉽다. ➡ `includeSetupPageInto(newPageContent)`는 함수 이름과 인수 사이에 추상화 수준이 다르다. 게다가 현 시점에서 별로 중요하지 않은 세부사항(StringBuffer)을 알아야한다.
- 테스트 관점에서 인수는 더 어렵다. ➡ 인수가 없다면 테스트가 간단하지만 그 개수가 많아질수록 복잡해진다.
- 출력 인수는 입력 인수보다 이해하기 어렵다. ➡ 흔히 함수에다 인수로 입력을 넘기고 반환값으로 출력을 받는다는 개념으로 이해하지만, 대게 함수에서 인수로 결과를 받으리라 기대하지 않기 때문. ➡ 그렇기에 출력 인수는 독자로 하여금 코드를 재차 확인하게 하며 헷갈림의 요소가 될 수 있다.
- 가장 베스트는 입력 인수가 없는 경우이며, 차선으로는 1개인 경우이다.
- __많이 쓰는 단항 방식__
  - 함수에 인수 1개를 넘기는 이유로 가장 흔한 두 가지 경우가 있다.
    1. 인수에 질문을 던지는 경우. ➡ `boolean fileExists("MyFile")`
    2. 인수를 뭔가로 변환해 결과를 반환하는 경우. ➡ `InputStream fileOpen("MyFile")`은 String형의 파일 이름을 InputStream으로 변환
  - 다소 드물게 사용하지만, 단항 함수 형식이 이벤트다. ➡ 이벤트 함수는 입력 인수만 있으며 출력 인수는 없다.
    - 프로그램은 함수 호출을 이벤트로 해석해 입력 인수로 시스템 상태를 바꾼다. ➡ `passwordAttemptFailedNtimes(int attempts)` 가 좋은 예.
    - 이벤트 함수는 조심해서 사용해야하며, 이벤트라는 사실이 코드에 명확히 드러나야 한다.
    - 그러므로, 이름과 문맥을 주의해서 선택한다.
  - 위의 세가지 경우가 아니라면 단항 함수는 가급적 피한다.
    - 예를 들어, `void includeSetupPageInto(StringBuffer pageText)` 는 피한다.
    - 입력 인수를 변환하는 함수라면 변환 결과는 반환값으로 돌려준다. `void transform(StringBuffer out)` ❌ / `StringBuffer transform(StringBuffer in)` ✅
- __플래그 인수__
  - 플래그 인수는 추하다.
  - 함수로 부울 값을 넘기는 관계는 끔직다. ➡ 함수가 한번에 여러 가지를 처리한다고 대놓고 알리는 꼴.
- __이항 함수__
  - 인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다.
    - 예를 들어, `writeField(name)` 는 `writeField(outputStream, name)` 보다 이해하기 쉽다.
  - 이항 항수가 더 적절한 경우가 있는데, 바로 직표 좌표계이다 ➡ `point p = new Point(0, 0)` 가 좋은 예.
    - 직표 좌표계 점은 일반적으로 인수 2개를 취한다. 여기서 인수 1개라면 더 당황스러울 것이다.
  - 당연하게 여겨지는 이항함수, `assertEquals(expected, actual)` 에도 문제가 있다.
    - 두 인수에 자연적인 순서가 없다. expected 다음에 actual이 온다는 순서를 인위적으로 기억해야 한다.
  - 이항 함수가 무조건 나쁘다는 건 아니지만, 그만큼 위험이 따른다는 사실을 이해하고 가능하다면 단항 함수로 바꾸도록 하자.
- __삼항 함수__
  - 인수가 3개인 함수는 인수가 2개인 함수보다 훨씬 이해하기 어렵다.
    - 순서, 주춤, 무시로 야기되는 문제가 두 배 이상 늘어난다. ➡ 그래서, 삼항 함수를 만들 때는 신중히 고려하자.
  - 예를 들어, `assertEquals(message, expected, actual)` 에서는 매번 함수를 볼 때마다 예상을 빗나갈 수 있다.
- __인수 객체__
  - 만약 인수가 2~3개 필요하다면, 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어 보자.
    - `Circle makeCircle(double x, double y, double radius)`
    - `Circle makeCircle(point center, double radius)`
- __인수 목록__
  - 때로는 인수 개수가 가변적인 함수도 필요하다.
  - `String.format` 메서드가 좋은 예이다. ➡ `String.format("%s worked %.2f" hours, name, hours)`
    - 위의 예처럼 가변 인수 전부를 동등하게 취급하면 List형 인수 하나로 취급할 수 있다. ➡ `public String format(String format, Object... args)`
  - 가변 인수를 취하는 함수는 단항, 이항, 삼항 함수로 취급할 수 있지만, 이를 넘어서는 인수를 사용할 경우에 문제가 있다.
- __동사와 키워드__
  - 함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필수다.
    - 단항 함수는 함수/인수가 동사/명사 쌍을 이뤄야 한다. ➡ ex) `writeField(name)`
  - 함수에 키워드를 넣으면 인수 순서를 기억할 필요가 없어진다.
    - `assertEquals` 보다 `assertExpectedEqualsActual(expected, actural)` 이 더 좋다.

---

### 7. 부수효과를 일으키지 마라!

- 부수효과는 좋지않다. ➡ 함수에서 한 가지를 하겠다고 하고선, 다른 짓도 하는 것이다.
- 때로는, 예상치 못하게 클래스 변수를 수정하며, 함수로 넘어온 인수나 시스템 전역 변수를 수정한다. ➡ 많을 경우, 시간적인 결합이나(temporal coupling)이나 순서 종속성(order dependency)을 초래한다.

<details>
<summary>목록 3-6 UserValidator.java</summary>

```java
public class UserValidator {
  private Cryptographer cryptographer;

  public boolean checkPassword(string userName, String password) {
    User user = UserGateway.findByName(userName);
    if (user != User.NULL) {
      String codedPhrase = user.getPhraseEncodedByPassword();
      String phrase = cryptographer.decrypt(codedPhrase, password);
      if ("Valid Password".equals(phrase)) {
        Session.initialize();
        return true;
      }
    }
    return false;
  }
}
```
> 함수는 표준 알고리즘을 사용해 userName과 password를 확인한다. 두 인수가 올바르면 true를 반환하고 아니면 false를 반환한다.
> 결과적으로 이 함수는 부수효과를 일으킨다.

</details>

- 위 코드의 부수 효과는 `Session.initialize()`의 호출이다. 
- checkPassword라는 함수는 이름 드대로 암호를 확인하는 것이다. 그러나 암호를 확인하면서 세션을 초기화한다는 의미를 찾기 힘들다. ➡ 이런 효과가 _시간적인 결합_을 초래한다. 즉, 세션을 초기화해도 괜찮은 경우에서만 호출이 가능하다는 것.
- 만약, 시간적인 결합이 필요하다면 함수 이름에 분명히 명시한다. ➡ `checkpPassword` 보다, `checkPasswordAndInitializeSession`이 더 좋아 보인다. ➡ 물론, 이는 함수가 '한 가지'만 한다는 규칙을 위반하는 것..
- __출력 인수__
  - 일반적으로 우리는 인수를 함수 __입력__으로 해석한다. 
  - `appendFooter(s)` 는 무언가에 s를 바닥글로 첨부할까? s에 바닥글을 첨부할까? / 입력일까? 출력일까?
    - 함수 선언부에서는 `public void appendFooter(StringBuffer report)` 로 표현하고 있다.
      - 이를 보면 인수 s가 출력 인수로 사용되고 있음을 알 수 있다. ➡ 이렇게 함수 선언부를 통해 비로소 그 역할을 이해할 수 있었으므로 인지적으로 거슬린다 표현 할 수 있겠다. (피해야 한다.)

  - 하지만, 객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다.
    - 출력 인수로 사용하라고 설계한 변수가 바로 this이기 때문!
    - 즉, 위의 예시에서 `appendFooter` 는 `report.appendFooter()` 표현하는 것 이 좋다.

  - 일반적으로 출력 인수는 피해야 한다. 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택한다.


---

### 8. 명령과 조회를 분리하라!

- 함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다. 둘 다 하면 안됀다. ➡ 객체 상태를 변경하거나 아니면 객체 정보를 반환하거나 둘 중 하나이다.
- `public boolean set(String attribute, String value);` ➡ 이 함수는, 이름이 attribute인 속성을 찾아 값을 value 설정한 후 성공하면 true를 반환하고, 실패하면 false를 반환한다. ➡ 즉, `if (set("username", "unclebob"))...` 로 나타낼 수 있다.
  - 독자 입장에서 이는 set이라는 단어가 동사인지 형용사인지 분간하기 어렵기 떄문에 헷갈릴 수 있다.
- 이에 대한 해결책은 __명령과 조회를 분리__해 혼란을 애초에 뿌리뽑는 방법이다.

```java
if (attributeExists("username")) {		// 조회
  setAttribute("username", "unclebob");		// 명령
  ...
}
```

---

### 9. 오류 코드보다 예외를 사용하라!

- 명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다. ➡ 자칫하면 if 문에서 명령을 표현식으로 사용하기 쉽기 때문이다. ➡ `if (deletePage(page) == E_OK)`
  - 위 코드는 동사/형용사 혼란을 일으키진 않지만, 여러 단계로 중첩되는 코드를 야기한다.
  - 오류 코드를 반환하면 호출자는 오류 코드를 곧바로 처리해야한다는 문제까지 부딪힐 수 있다.
- 반면, 오류 코드 대신에 예외를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로, 코드가 깔끔해진다.

```java
// 오류 코드
if (deletePage(page) == E_OK) {
  if (registry.deleteReference(page.name) == E_OK) {
    if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
      logger.log("page deleted");
    } else {
      logger.log("configKey not deleted");
    }
  } else {
    logger.log("deleteReference from registry failed");
  }
} else {
  logger.log("delete failed");
  return E_ERROR;
}

// 예외 코드
try {
  deletePage(page);
  registry.deleteReference(page.name);
  configKeys.deleteKey(page.name.makeKey());
}
catch (Exception e) {
  logger.log(e.getMessage());
}
```

- __Try/Catch 블록 뽑아내기__
  - try/catch 블록은 코드 구조에 혼란을 일으키며, 정상 동작과 오류 처리 동작을 섞는다. ➡ 그러므로, try/catch 블록은 별도의 함수로 뽑아내는 편이 좋다.
  - 아래에서 delete 함수는 모든 오류를 처리하며, 페이지를 제거하는 함수는 deletePageAndReference이다. ➡ 이렇게 정상 동작과 동류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.

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

- __오류 처리도 한 가지 작업이다__

  - 함수는 '한 가지' 작업만 해야 하며, 오류 처리도 '한 가지' 작업에 속한다. ➡ 그러므로 오류를 처리하는 함수는 오류만 처리해야 한다.
    - 즉, 함수에 try 키워드가 있다면, 그 함수는 try 문으로 시작해 catch/finally 문으로 끝나야 한다.

- __Error.java 의존성 자석__

  - 오류 코드를 반환한다는 이야기는, 클래스든 열거형 변수든, 어디선가 오류 코드를 정의한다는 뜻이다.

  - ```java
    public enum Error {
      OK,
      INVALID,
      NO_SUCH,
      LOCKED,
      OUT_OF_RESOURCES,
      WAITING_FOR_EVENT;
    }
    ```

  - 위와 같은 클래스는 의존성 자석(magnet)이다. 다른 클래스에서 Error enum을 import해 사용해야 하므로. ➡ 즉, Error enum이 변한다면 Error enum을 사용하는 클래스를 다시 컴파일하고 다시 배치해야 한다. 그러므로, Error의 변경이 어려워 진다.

  - 이럴 경우, 오류 코드 대신 예외를 사용하면 새 예외는 Exception 클래스에서 파생된다. ➡ 따라서, 재컴파일/재배치 없이도 새 예외 클래스를 추가 할 수 있다. (OCP를 보여주는 예.)

---

### 10. 반복하지 마라. (DRY, Don't Repeat Yourself)

- 중복은 코드 길이가 늘어날 뿐 아니라 알고리즘이 변하면 여러 곳이나 수정해야 한다. 게다가, 어느 한 곳이라도 빠뜨리는 경우 오류가 발생할 확률도 배로 높아진다.
- 중복은 소프트웨어에서 모든 악의 근원이며, 많은 원칙과 기법이 중복을 없애거나 제어할 목적으로 나왔다.
  - _E.F. 커드_는 자료에서 중복을 제거할 목적으로 관계형 데이터베이스에 정규 형식을 만들었다.
  - 객체지향 프로그래밍은 코드를 부모 클래스로 몰아 중복을 없앤다.
  - 구조적 프로그래밍, AOP, COP 모두 중복 제거 전략이다.

---

### 11. 구조적 프로그래밍

- 어떤 프로그래머는 _에츠허르 데이크스트라_ 의 구조적 프로그래밍 원칙을 따른다. ➡ 모든 함수와 함수 내 모든 블록의 입구와 출구가 하나만 존재해야 한다고 말한다. 즉, return 문이 하나여야 한다는 말!
- 루프 안에서 break나 continue를 사용해선 안 되며, goto는 절대절대 사용하면 안된다.
- 구조적 프로그래밍의 목표와 규율은 공감하지만 함수가 작다면 위 규칙은 큰 이익을 제공하지 않는다. ➡ 함수가 아주 클 때만 상당한 이익을 제공한다.
- 만약 함수를 작게 만든다면, return / break / continue를 여러 차례 사용해도 괜찮다. ➡ 이는 때로 단일 입/출구 규칙보다 의도를 표현하기 쉬울 수도 있다.
- 그러나 goto 문은 큰 함수에서만 의미가 있으므로, 작은 함수에서는 지양하여라.

---

### 12. 함수를 어떻게 짜죠?

- 소프트웨어를 짜는 행위는 여느 글짓기와 비슷하다. ➡ 초안은 대개 서투르고 어수선하므로 원하는 대로 읽힐 때까지 말을 다듬고 문장을 고치고 문단을 정리한다.
- 함수를 짜는 것도 마찬가지 ➡ 처음에는 길고 복잡하며 들여쓰기 단계도 많고 중복된 루프와 인수목록이 길다. --> 그러나 이 서투른 코드에서도 빠짐없이 테스트하는 단위 테스트는 놓치지 말고 수행한다. --> 다음으로 코드를 다듬고 함수를 만들고 중복을 제거하고 클래스를 쪼갠다. 이 모든 과정에서도 항상 단위테스트를 통과한다. --> 최종적으로 원하는 코드가 만들어 진다.

---

### 13. 결론

- 모든 시스템은 특정 응용 분야 시스템을 기술할 목적으로 프로그래머가 설계한 __도메인 특화 언어 (DSL, Domain Specific Language)__ 로 만들어 진다.
- 함수는 그 언어에서 동사며, 클래스는 명사이다. 
- 대가의 프로그래머는 시스템을 (구현할) 프로그램이 아니라, (풀어갈) 이야기로 여긴다. 즉, 프로그래밍 언어라는 수단을 이용해 좀 더 풍부하고 좀 더 표현력이 강한 언어를 만들어 이야기를 풀어나가는 것.
- 진짜 목표는 시스템이라는 이야기를 풀어가는 데 있다는 사실을 명심하라.

---

<details>
<summary>목록 3-7 SetupTeardownIncluder.java</summary>

```java
package fitnesse.html;

import fitnesse.responders.run.SuiteResponder;
import fitnesse.wiki.*;

public class SetupTeardownIncluder {
  private PageData pageData;
  private boolean isSuite;
  private WikiPage testPage;
  private StringBuffer newPageContent;
  private PageCrawler pageCrawler;

  public static String render(PageData pageData) throws Exception {
    return render(pageData, false);
  }

  public static String render(PageData pageData, boolean isSuite) throws Exception {
    return new SetupTeardownIncluder(pageData).render(isSuite);
  }

  private SetupTeardownIncluder(pageData pageData) {
    this.pageData = pageData;
    testPage = pageData.getWikiPage();
    pageCrawler = testPage.getPageCrawler();
    newPageContent = new StringBuffer();
  }

  private String render(boolean isSuite) throws Exception {
    this.isSuite = isSuite;
    if (isTestPage()) includeSetupAndTeardownPages();
    return pageData.getHtml();
  }

  private void includeSetupAndTeardownPages() throws Exception {
    includeSetupPages();
    includePageContent();
    includeTeardownPages();
    updatePageContent();
  }

  private void includeSetupPages() throws Exception {
    if (isSuite) includeSuiteSetupPage();
    includeSetupPage();
  }

  private void includeSuiteSetupPage() throws Exception {
    include(SuiteResponder.SUITE_SETUP_NAME, "-setup");
  }

  private void includeSetupPage() throws Exception {
    include("Setup", "-setup");
  }

  private void includePageContent() throws Exception {
    newPageContent.append(pageData.getContent());
  }

  private void includeTeardownPages() throws Exception {
    includeTeardownPage();
    if (isSuite) includeSuiteTeardownPage();
  }

  private void includeTeardownPage() throws Exception {
    include("Teardown", "-teardown");
  }

  private void updatePageContent() throws Exception {
    pageData.setContent(newPageContent.toString());
  }

  private void include(String pageName, String arg) throws Exception {
    WikiPage inheritedPage = findInheritedPage(pageName);
    if (inheritedPage != null) {
      String pagePathName = getPathNameForPage(inheritedPage);
      buildIncludeDirective(pagePathName, arg);
    }
  }

  private WikiPage findInheritedPage(String pageName) throws Exception {
    return PageCrawlerImpl.getInheritedPage(pageName, testPage);
  }

  private String getPathNameForPage(WikiPage page) throws Exception {
    WikiPagePath pagePath = pageCrawler.getFullPath(page);
    return PathParser.render(pagePath);
  }

  private void buildIncludeDirective(String pagePathName, String arg) {
    newPageContent
      .append("\n!include ")
      .append(arg)
      .append(" .")
      .append(pagePathName)
      .append("\n");
  }
}
```

</details>


---

__추가 학습 개념__

- SRP
- OCP
- 시간적인 결합?
- 출력 인수? [참고](https://velog.io/@sansam202/Output-Parameter%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B3%A0%EC%B0%B0)
- AOP
- COP
- DSL