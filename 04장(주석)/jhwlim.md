# 📖 4장. 주석

## 요약
- 코드를 깔끔하게 정리하고 표현력을 강화하여 주석을 가능한 줄이도록 꾸준히 노력해야 한다.
- 주석으로 달려는 설명을 함수로 만들어 표현하라.
- 좋은 주석은 코드만 보고는 이해하기 어렵거나 코드만으로는 제공할 수 없는 유용한 정보를 제공한다. 
    (의도, 의미, 경고, 중요성, TODO 등)
- 주석의 내용이 코드와 동일한 정보를 제공하거나 부정확하거나 이해하기 어렵다면 나쁜 주석이다.

## 🔖 목차

1. [주석은 나쁜 코드를 보완하지 못한다](#01-주석은-나쁜-코드를-보완하지-못한다)
2. [코드로 의도를 표현하라!](#02-코드로-의도를-표현하라)
3. [좋은 주석](#03-좋은-주석)
4. [나쁜 주석](#04-나쁜-주석)

## 들어가기 전에

- 잘 달린 주석은 그 어떤 정보보다 유용하다.
- 경솔하고 근거 없는 주석은 코드를 이해하기 어렵게 만든다. 오래되고 조잡한 주석은 거짓과 잘못된 정보를 퍼뜨려 해악을 미친다.
- 주석은 기껏해야 필요악이다.
- _**우리는 코드로 의도를 표현하지 못해, 그러니까 실패를 만회하기 위해 주석을 사용한다.**_
- 주석은 언제나 실패를 의미한다. 때때로 주석 없이는 자신을 표현할 방법을 찾지 못해 할 수 없이 주석을 사용한다.
- 주석은 오래될수록 코드에서 멀어진다. 오래될수록 완전히 그릇될 가능성도 커진다. 주석을 유지하고 보수하기란 현실적으로 불가능하기 때문이다.
- 코드는 변화하고 진화한다. 불행하게도 주석이 언제나 코드를 따라가지 못한다.
- _**코드를 깔끔하게 정리하고 표현력을 강화하는 방향으로, 그래서 애초에 주석이 필요 없는 방향으로 에너지를 쏟아라.**_
- 부정확한 주석은 아예 없는 주석보다 훨씬 더 나쁘다.
- _**코드만이 정확한 정보를 제공하는 유일한 출처다. 그러므로 주석을 가능한 줄이도록 꾸준히 노력해야 한다.**_

## 01. 주석은 나쁜 코드를 보완하지 못한다.

- 코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다.
- _**표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가, 복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다.**_

## 02. 코드로 의도를 표현하라!

나쁜 예

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))
```

좋은 예

```java
if (employee.isEligibleForFullBenefits())
```

- _**많은 경우 주석으로 달려는 설명을 함수로 만들어 표현해도 충분하다.**_

## 03. 좋은 주석

- 어떤 주석은 필요하거나 유익하다.
- _**정말로 좋은 주석은, 주석을 달지 않을 방법을 찾아낸 주석이다.**_

### (1) 법적인 주석

- 때로는 회사가 정리한 구현 표준에 맞춰 법적인 이유로 특정 주석을 넣으라고 명시한다.
  예) 각 소스 파일 첫머리에 주석으로 들어가는 저작권 정보와 소유권 정보는 필요하고도 타당하다.
- 소스 파일 첫머리에 들어가는 주석이 반드시 계약 조건이나 법적인 정보일 필요는 없다. 모든 조항과 조건을 열거하는 대신에, 가능하다면, 표준 라이선스나 오부 문서를 참조해도 되겠다.

### (2) 정보를 제공하는 주석

- 때로는 기본적인 정보를 주석으로 제공하면 편리하다.
  예) 추상 메서드가 반환할 값을 설명한다.

  ```java
  // 테스트 중인 Responder 인스턴스를 반환한다.
  protected abstract Responder responderInstance();
  ```

  - 가능하면, 함수 이름에 정보를 담는 편이 더 좋다. 함수 이름을 responderBeingTested()로 바꾸면 주석이 필요없어 진다.

  예) 코드에서 사용한 정규표현식이 시각과 날짜를 뜻한다고 설명한다. 구체적으로는 주어진 형식 문자열을 사용해 SimpleDateFormat.format 함수가 반환하는 시각과 날짜를 뜻한다.

  ```java
  // kk:mm:ss EEE, MMM dd, yyyy 형식이다.
  Pattern timeMatcher = Pattern.comiple("\\d*:\\d*:\\d*, \\w*, \\w*, \\d*, \\d*");
  ```

  - 시각과 날짜를 변환하는 클래스를 만들어 코드를 옮겨주면 더 좋고 더 깔끔하다.

### (3) 의도를 설명하는 주석

- 때때로 주석은 구현을 _**이해하게 도와주는**_ 선을 넘어 _**결정에 깔린 의도까지 설명한다.**_
  좋은 예)

  ```java
  public int compareTo(Object o) {
      if (o instanceof WikiPagePath) {
          WikiPagePath p = (WikiPagePath) o;
          String compressedName = StringUtil.join(names, "");
          String compressedArgumentName = StringUtil.join(p.names, "");
          return compressedName.compareTo(compressedArgumentName);
      }
      return 1; // 오른쪽 유형이므로 정렬 순위가 더 높다.
  }
  ```

  좋은 예)

  ```java
  public void testConcurrentAddWidgets() throws Exception {
      WidgetBuilder widgetBuilder = new WidgetBuilder(new Class[]{BoldWidget.class});
      String text = "'''bold test'''";
      ParentWidget parent = new BoldWidget(new MockWidgetRoot(), "'''bold text'''");
      AtomicBoolean failFlag = new AtomicBoolean();
      failFlag.set(false);

      // 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
      for (int i = 0; i < 25000; i++) {
          WidgetBuilderThread widgetBuilderThread = new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
          Thread thread = new Thread(widgetBuilderThread);
          thread.start();
      }
      assertEquals(false, failFlag.get());
  }
  ```

### (4) 의미를 명료하게 밝히는 주석

- _**때때로 모호한 인수나 반환값은 그 의미를 읽기 좋게 표현하면 이해하기 쉬워진다.**_
- 일반적으로는 인수나 반환값 자체를 명확하게 만들면 더 좋겠지만, 인수나 반환값이 표준 라이브러리나 변경하지 못하는 코드에 속한다면 의미를 명료하게 밝히는 주석이 유용하다.

```java
public void testCompareTo() throws Exception {
    WikiPagePath a = PathParser.parse("PageA");
    WikiPagePath ab = PathParser.parse("PageA.PageB");
    WikiPagePath b = PathParser.parse("PageB");
    WikiPagePath aa = PathParser.parse("PageA.PageA");
    WikiPagePath bb = PathParser.parse("PageB.PageB");
    WikiPagePath ba = PathParser.parse("PageB.PageA");

    assertTrue(a.compareTo(a) == 0);    // a == a
    assertTrue(a.compareTo(b) != 0);    // a != b
    assertTrue(ab.compareTo(ab) == 0);  // ab == ab
    assertTrue(a.compareTo(b) ==  -1);  // a < b
    assertTrue(aa.compareTo(ab) == -1); // aa < ab
    assertTrue(ba.compareTo(bb) == -1); // ba < bb
    assertTrue(b.compareTo(a) == 1);    // b > a
    assertTrue(ab.compareTo(aa) == 1);  // ab > aa
    assertTrue(bb.compareTo(ba) == 1);  // bb > ba
}
```

- 물론 그릇된 주석을 달아놓을 위험은 상당히 높다. _**주석이 올바른지 검증하기 쉽지 않다.**_ 이것이 의미를 명료히 밝히는 주석이 필요한 이유인 동시에 주석이 위험한 이유이기도 하다.
- 그러므로 위와 같은 주석을 달 때는 _**더 나은 방법이 없는지 고민하고 정확히 달도록 각별히 주의한다.**_

### (5) 결과를 경고하는 주석

- 때로는 다른 프로그래머에게 결과를 경고할 목적으로 주석을 사용한다.
  좋은 예) 특정 테스트 케이스를 꺼야하는 이유를 설명하는 주석

  ```java
  // 여유 시간이 충분하지 않다면 실행하지 마십시오.
  public void _testWithReallyBigFile()
  ```

  - `_` 기호를 붙이는 방법이 일반적인 관례였음.

  좋은 예) 프로그램 효율을 높이기 위해 정적 초기화 함수를 사용하려던 열성적인 프로그래머가 주석 때문에 실수를 면한다.

  ```java
  public static SimpleDateFormat makeStandardHttpDateFormat() {
      // SimpleDateFormat은 스레드에 안전하지 못하다.
      // 따라서 각 인스턴스를 독립적으로 생성해야 한다.
      SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z");
      df.setTimeZone(Timezone.getTimeZone("GMT"));
      return df;
  }
  ```

### (6) TODO 주석

- 때로는 '앞으로 할 일'을 `//TODO` 주석으로 남겨두면 편하다
  예) 함수를 구현하지 않은 이유와 미래 모습을 `//TODO` 주석으로 설명
  ```java
  // TODO-MdM 현재 필요하지 않다.
  // 체크아웃 모델을 도입하면 함수가 필요 없다.
  protected VersionInfo makeVersion() throws Exception {
      return null;
  }
  ```
- TODO 주석은 _**프로그래머가 필요하다 여기지만 당장 구현하기 어려운 업무를 기술한다. 더 이상 필요 없는 기능을 삭제하라는 알림, 누군가에게 문제를 봐달라는 요청, 더 좋은 이름을 떠올려달라는 부탁, 앞으로 발생할 이벤트에 맞춰 코드를 고치라는 주의 등에 유용하다.**_
- 그래도 TODO로 떡칠한 코드는 바람직하지 않다. 그러므로 주기적으로 TODO 주석을 점검해 없애도 괜찮은 주석은 없애라고 권한다.

### (7) 중요성을 강조하는 주석

- 자칫 대수롭지 않다고 여겨질 뭔가의 중요성을 강조하기 위해서도 주석을 사용한다.
  예)
  ```java
  String listItemContent = match.group(3).trim();
  // 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
  // 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
  new ListItemWidget(this, listItemContent, this.level + 1);
  return buildList(text.substring(match.end()));
  ```

### (8) 공개 API에서 Javadocs

- 공개 API를 구현한다면 반드시 훌륭한 Javadocs를 작성한다.
- 하지만 여느 주석과 마찬가지로 Javadocs 역시 _**독자를 오도하거나, 잘못 위치하거나, 그릇된 정보를 전달할 가능성이 존재한다.**_

## 04. 나쁜 주석

- 일반적으로 대다수 주석은 허술한 코드를 지탱하거나, 엉성한 코드를 변명하거나, 미숙한 결정을 합리화하는 등 프로그래머가 주절거리는 독백에서 크게 벗어나지 못한다.

### (1) 주절거리는 주석

- 특별한 이유 없이 의무감으로 혹은 프로세스에서 하라고 하니까 ***마지못해 주석을 단다***면 전적으로 시간낭비다.
- _**주석을 달기로 결정했다면 충분한 시간을 들여 최고의 주석을 달도록 노력한다.**_
- _**이해가 안 되어 다른 모듈까지 뒤져야 하는 주석은 독자와 제대로 소통하지 못하는 주석이다.**_ 그런 주석은 바이트만 낭비할 뿐이다.

### (2) 같은 이야기를 중복하는 주석

- 헤더에 달린 주석이 같은 코드 내용을 그대로 중복하는 경우
- _**코드보다 부정확해 독자가 함수를 대충 이해하고 넘어가게 만들 수도 있다.**_

### (3) 오해할 여지가 있는 주석

- _**잘못된 정보로 인해 경솔하게 함수를 호출할 수도 있다.**_

### (4) 의무적으로 다는 주석

- 모든 함수에 javadocs를 달거나 모든 변수에 주석을 달아야 한다는 규칙은 어리석기 그지없다.
- 이런 주석은 코드를 복잡하게 만들며, 거짓말을 퍼뜨리고, 혼동과 무질서를 초래한다.

### (5) 이력을 기록하는 주석

- 소스 코드 관리 시스템을 통해 이력에 대한 관리를 하면 된다.
- 혼란만 가중할 뿐이다. 완전히 제거하는 편이 좋다.

### (6) 있으나 마나 한 주석

- 너무나 당연한 사실을 언급하며 _**새로운 정보를 제공하지 못하는 주석**_
  예)

  ```java
  /**
   * 기본 생성자
   */
  protected AnnualDateRul() {
  }

  /** 월 중 일자 */
  private int dayOfMonth;

  /**
   * 월 중 일자를 반환한다.
   *
   * return 월 중 일자
   */
  public int getDayOfMonth() {
      return dayOfMonth;
  }
  ```

- _**개발자가 주석을 무시하는 습관에 빠진다.**_
- 있으나 마나 한 주석을 달려는 유혹에서 벗어나 코드를 정리하라. 더 낫고, 행복한 프로그래머가 되는 지름길이다.

### (7) 무서운 잡음

- 때로는 Javadocs도 잡음이다.
- 주석을 작성한 저자가 주의를 기울이지 않았다면 독자가 여기서 무슨 이익을 얻겠는가?

### (8) 함수나 변수로 표현할 수 있다면 주석을 달지 마라

- 주석이 필요하지 않도록 코드를 개선하는 편이 좋다..
  나쁜 예)
  ```java
  // 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
  if (smodule.getDependSubsystem().contains(subSysMod.getSubSystem()))
  ```
  좋은 예)
  ```java
  ArrayList moduleDependees = smodule.getDependSubsystem();
  String ourSubSystem = subSysMod.getSubSystem();
  if (moduleDependees.contains(ourSubSystem))
  ```

### (9) 위치를 표시하는 주석

- 소스 파일에서 특정 위치를 표시하려 사용하는 주석

```java
// Actions ///////////////////
```

- 위와 같은 배너 아래 특정 기능을 모아놓으면 유용한 경우도 있긴 있다.
- 하지만 일반적으로 위와 같은 주석은 가독성만 낮추므로 제거해야 마땅하다. 특히 뒷부분에 슬래시(`/`)로 이어지는 잡음은 제거하는 편이 좋다.
- 너***무 자주 사용하지 않는다면 배너는 눈에 띄며 주의를 환기한다. 그러므로 반드시 필요할 때만, 아주 드믈게 사용하는 편이 좋다. 배너를 남용하면 독자가 흔한 잡음으로 여겨 무시한다.***

### (10) 닫는 괄호에 다는 주석

- 중첩이 심하고 장황한 함수라면 의미가 있을지도 모르지만, 작고 캡슐화된 함수에는 잡음일 뿐이다.
- 그러므로 _**닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신에 함수를 줄이려 시도하자.**_

### (11) 공로를 돌리거나 저자를 표시하는 주석

- 현실적으로 이런 주석은 그냥 오랫동안 코드에 방치되어 점차 부정확하고 쓸모없는 정보로 변하기 쉽다.
- 이러한 정보는 소스 코드 관리 시스템에 저장하는 편이 좋다.

### (12) 주석으로 처리한 코드

- 주석으로 처리된 코드는 다른 사람들이 지우기를 주저한다.
- 소스 코드 관리 시스템이 우리를 대신해 코드를 기억해준다. 이제는 주석으로 처리할 필요가 없다. 그냥 코드를 삭제하라. 잃어버릴 염려는 없다.

### (13) HTML 주석

- HTML 주석은 (주석을 읽기 쉬워야 하는) 편집기/IDE에서조차 읽기가 어렵다. (Javadocs와 같은) 도구로 주석을 뽑아 웹 페이지에 올릴 작정이라면 주석에 HTML 태그를 삽입해야 하는 책임은 프로그래머가 아니라 도구가 져야 한다.

### (14) 전역 정보

- 주석을 달아야 한다면 근처에 있는 코드만 기술하라.
- _**코드 일부에 주석을 달면서 시스템의 전반적인 정보를 기술하지 마라.**_

### (15) 너무 많은 정보

- _**주석에다 흥미로운 역사나 관련 없는 정보를 장황하게 늘어놓지 마라.**_

### (16) 모호한 관계

- 주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다. 이왕 공들여 _**주석을 달았다면 적어도 독자가 주석과 코드를 읽어보고 무슨 소린지 알아야 한다.**_
- 주석을 다는 목적은 코드만으로 설명이 부족해서다. 주석 자체가 다시 설명을 요구하니 안타깝기 그지않다.

### (17) 함수 헤더

- _**주석으로 헤더를 추가하기보다 짧고 한가지만 수행하며 이름을 잘 붙인 함수를 만들도록 해야한다.**_

### (18) 비공개 코드에서 Javadocs

- 공개 API는 Javadocs가 유용하지만 _**공개하지 않을 코드라면 Javadocs는 쓸모가 없다.**_
- 시스템 내부에 속한 클래스와 함수에 Javadocs를 생성할 필요는 없다. 유용하지 않을 뿐만 아니라 Javadocs 주석이 요구하는 형식으로 인해 코드만 보기 싫고 산만해질 뿐이다.
