## 4장 주석

---

- 잘달린 주석은 그 어떤 정보보다 유용하다. 그러나 경솔하고 근거 없는 주석은 코드를 이해하기 어렵게 만든다.
- 우리는 코드로 의도를 표현하지 못해, 실패를 만회하기 위해 주석을 사용한다.
  - 주석은 언제나 실패를 의미한다. 
  - 때때로 주석 없이는 자신을 표현할 방법을 찾지 못해 할 수 없이 주석을 사용한다.
  - 그렇기에 주석은 반겨 맞을 손님이 아니다..
- 주석은 오래될수록 코드에서 멀어진다. ➡ 그 이유는, 프로그래머들이 주석을 유지하고 보수하기란 현실적으로 불가능하기 때문이다.
- 부정확한 주석은 아예 없는 주석보다 훨씬 더 나쁘다. ➡ 진실은 __코드__ 에만 존재한다. 그러므로, 우리는 (간혹 필요할지라도) 주석을 가능한 줄이도록 꾸준히 노력해야 한다.

---

### 1. 주석은 나쁜 코드를 보완하지 못한다.

- 코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다.
- 표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가, 복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다.
  - 자신이 저지른 난장판을 주석으로 설명하려는 대신에, 그 난장판을 깨끗이 치우는 데 시간을 보내라!

---

### 2. 코드로 의도를 표현하라!

- 확실히 코드만으로 의도를 설명하기 어려운 경우가 존재하는데, 이를 많은 개발자들은 코드는 훌륭한 수단이 아니라는 의미로 해석한다. ➡ 이는 매우 잘못된 생각이다.

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if ((employee.flag & HOURLY_FLAG) && (employee.age > 65))
```

```java
if (employee.isEligibleForFullBenefits())
```

- 두 가지의 예제를 보더라도, 코드로 대다수의 의도를 표현할 수 있다. ➡ 주석으로 달려는 설명을 함수로 만들어 표현해도 충분하다.

---

### 3. 좋은주석

- 어떤 주석은 필요하거나 유익하다. 그러나 정말로 좋은 주석은 주석을 잘지 않을 방법을 찾아낸 주석이라는 것을 명심해라.

- __법적인 주석__

  - 때로는 회사가 정립 구현 표준에 맞춰 법적인 이유로 특정 주석을 넣으라고 명시한다.
    - 예를 들어, 각 소스 파일 첫머리에 주석으로 들어가는 저작권 정보와 소유권 정보는 필요하고도 타당하다.
    - `// Copyright (C) 2003, 2004, 2005, by Object Mentor, Inc. All rights reserved.`
    - `// GNU General Public License 버전 2 이상을 따르는 조건으로 배포한다.`

- __정보를 제공하는 주석__

  - 때로는 기본적인 정보를 주석으로 제공하면 편리하다.

    - 예를 들어, 다음 주석은 추상 메서드가 반환할 값을 설명한다.

      - `protected abstract Responder resonderInstance();  //테스트 중인 Responder 인스턴스를 반환한다. `

    - 위의 예시처럼 주석이 유용하다해도, 가능하다면 함수 이름에 정보를 담는 편이 좋다. (`responderBeingTested`)

    - 좀 더 나은 예제

      - ```java
        // kk:mm:ss EEE, MMM dd, yyyy 형식이다.
        Pattern timeMatcher = Pattern.compile("\\d*:\\d*:\\d* \\w*, \\w* \\d*, \\d*")
        ```

      - 이는, 코드에서 사용한 정규표현식이 시각과 날짜를 뜻한다고 설명하고 있다.

      - 이왕이면 시각과 날짜를 변환하는 클래스를 만들어 코드를 옮겨주면 더 좋고 깔끔하다 ➡ 주석도 필요 없어진다.

- __의도를 설명하는 주석__

  - 때때로 주석은 구현을 이해하게 도와주는 선을 넘어 결정에 깔린 의도까지 설명한다.

  - ```java
    public int compareTo(Object o) {
      if (o instanceof WikiPagePath) {
        WikiPagePath p = (WikiPagePath) o;
        String compressedName = StringUtil.join(names, "");
        String compressedArgumentName = StringUtil.join(p.names, "");
        return compressedName.compareTo(compressedArgumentName);
      }
      return 1;		// 오른쪽 유형이므로 정렬 순위가 높다.
    }
    ```

  - ```java
    public void testConcurrentAddWidgets() throws Exception {
      WidgetBuilder widgetBuilder = new WidgetBuilder(new Class[]{BoldWidget.class});
      String text = "'''bold text'''";
      ParentWidget parent = new BoldWidget(new MockWidgetRoot(), "'''bold text'''");
      AtomicBoolean failFlag = new AtomicBoolean();
      failFlag.set(false);
      
      // 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
      for (int i=0; i <25000; i++) {
        WidgetBuilderThread widgetBuilderThread = new WidgetBuilderThread(widgetBuilderm text, parent, failFlag);
        Thread thread = new Thread(widgetBuilderThread);
        thread.start();
      }
      assertEquals(false, failFlag,get());
    }
    ```

- __의미를 명료하게 밝히는 주석__

  - 때때로 모호한 인수나 반환값은 그 의미를 읽기 좋게 표현하면 이해하기 쉬워진다.
  - 일반적으로 인수나 반환값 자체를 명확하게 만들면 더 좋겠지만, 인수나 반환값이 표준 라이브러리나 변경하지 못하는 코드에 속한다면 의미를 명료하게 밝히는 주석이 유용하다.
    - `assertTrue(a.compareTo(a) == 0); // a == a`
    - `assertTrue(a.compareTo(a) != 0); // a != a`
    - `assertTrue(a.compareTo(b) == -1); // a < b`
  - 물론 그릇된 주석을 달아놓을 위험은 상당히 높으며, 주석이 올바른지 검증하기는 쉽지 않다. ➡ 의미를 명료히 밝히는 주석이 필요한 이유인 동시에 주석이 위험한 이유이기도 하다.
  - 그러므로 주석을 달 떄는 더 나은 방법이 없는지 고민하고 정확히 달도록 각별히 주의한다.

- __결과를 경고하는 주석__

  - 때로 다른 프로그래머에게 결과를 경고할 목적으로 주석을 사용한다.

  - ```java
    // 여유 시간이 충분하지 않다면 실행하지 마십시오.
    public void _testWithRealllyBigFile() {
      writeLinesToFile(10000000);
      
      response.setBody(testFile);
      response.readyToSend(this);
      String responseString = output.toString();
      asserSubString("Content-Length: 10000000", responseString);
      assertTrue(byteSent > 10000000);
    }
    ```

  - 이는 특정 테스트 케이스를 꺼야하는 이유를 설명하는 주석이다. 

    - 요즘에는 `@Ignore` 속성을 이용하여 테스트케이스를 꺼버린다. ➡ `@Ignore("실행이 너무 오래 걸립니다.")`

  - 더 나은 주석의 예제로

  - ```java
    public static SimpleDateFormat makeStandartHttpDataFormat() {
    	// SimpleDateFormat은 스레드에 안전하지 못하다.
      // 따라서 각 인스턴스를 독립적으로 생성해야 한다.
      SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z");
      df.setTimeZone(TimeZone.getTimeZone("GMT"));
      return df;
    }
    ```

    - 프로그램의 효율을 높이기 위해 정적 초기화 함수를 사용하려던 열성적인 프로그래머가 주석 때문에 실수를 면할 수 있다.

- __TODO 주석__

  - 때로는 '앞으로 할 일'을 //TODO 주석으로 남겨두면 편하다

  - ```java
    // TODO-MdM 현재 필요하지 않다.
    // 체크다웃 모델을 도입하면 함수가 필요 없다.
    protected VersionInfo makeVersion() throws Exception {
    	return null;
    }
    ```

  - TODO 주석은 프로그래머가 필요하다 여기지만 당장 구현하기 어려운 업무를 기술한다.

  - 하지만, 어떤 용도로 사용하든 시스템에 나쁜 코드를 남겨 놓는 핑계가 되어서는 안된다.

  - TODO로 떡칠한 코드는 바람직하지 않다. ➡ 주기적으로 TODO 주석을 점검해 없애도 괜찮은 주석은 없애라고 권한다. 

- __중요성을 강조하는 주석__

  - 자칫 대수롭지 않다고 여겨질 뭔가의 중요성을 강조하기 위해서도 주석을 사용한다.

  - ```java
    String listItemContent = match.group(3).trim();
    // 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
    // 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
    new ListItemWidget(this, listItemContent, this.level +1);
    return buildList(text.subString(match.end()));
    ```

- __공개 API에서 Javadocs__

  - 설명이 잘 된 공개 API는 참으로 유용하고 만족스럽다. 표준 자바 라이브러리에서 사용한 Javadocs가 좋은 예! ➡ Javadocs가 없다면 자바 프로그램을 짜기 어려웠을 것이다.
  - 공개 API를 구현한다면 반드시 훌륭한 Javadocs를 작성한다.
  - 하지만, 여느 주석과 마찬가지로 Javadocs 역시 독자를 오도하거나, 잘못 위치하거나, 그릇된 정보를 전달할 가능성이 존재함을 명심하자.

---

### 4. 나쁜 주석

- 대다수의 주석이 이 범주에 속한다.

- 일반적으로 대다수의 주석은 허술한 코드를 지탱하거나, 엉성한 코드를 변명하거나, 미숙한 결정을 합리화하는 등 프로그래머가 주절거리는 독백에서 크게 벗어나지 못한다.

- __주절거리는 주석__

  - 특별한 이유 없이 의무감으로 혹은 프로세스에서 하라고 하니까 마지못해 주석을 다는 것은 시간낭비이다. ➡ 주석을 달기로 결정했다면 충분한 시간을 들여 최고의 주석을 달도록 노력한다.

  - ```java
    public void loadProperties() {
    	try {
        String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
        FileInputStream propertiesStream = new FileInputStream(propertiesPath);
        loadedProperties,load(propertiesStream);
      }
      catch (IOException e) {
        // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미이다.
      }
    }
    ```

  - 여기의 catch 블록에 있는 주석은 무슨 뜻일까? ➡ 저자에게는 의미가 있겠지만, 독자에게 그 의미가 전해지지 않는다.

  - IOException이 발생하면 속성 파일이 없다는 뜻이다. 근데 모든 기본값을 메모리로 읽어 들였다? ➡ 무슨 의미인지 모호하다.

  - 여기서의 답을 알아내려면 다른 코드를 뒤져보는 수밖에 없다. ➡ 이해가 안 되어 다른 모듈까지 뒤져야 하는 주석이므로 독자와 제대로 소통하지 못한다.

- __같은 이야기를 중복하는 주석__

<details>
<summary>목록 4-1 waitForClose</summary>

```java
// this.closed가 true일 때 반환되는 유틸리티 메서드이다.
// 타임아웃에 도달하면 예외를 던진다.
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
  if (!closed) {
    wait(timeoutMillis);
    if (!closed) throw new Exception("MockResponseSender could not be closed");
  }
}
```
> 위의 주석은 코드를 정당화하는 주석도 아니고, 의도나 근거를 설명하는 주석도 아니다. 코드보다 읽기가 쉽지도 않다. ➡ 실제로 코드보다 부정확해 독자가 함수를 대충 이해하고 넘어가게 만든다.

</details>

- __오해할 여지가 있는 주석__
  - 때때로 의도는 좋았으나 프로그래머가 딱 맞을 정도록 엄밀하게는 주석을 달지 못하기도 한다.
  - 목록 4-1 에서는 this.closed가 true로 변하는 순간 메서드는 반환되지 않는다. this.closed가 true여야 메서드는 반환된다. 아니면 무조건 타임아웃을 기다렸다 this.closed가 그래도 true가 아니면 예외를 던진다. ➡ (코드보다 읽기도 어려운) 주석에 담긴 '살짝 잘못된 정보'이다.
- __의무적으로 다는 주석__
  - 모든 함수에 Javadocs를 달거나 모든 변수에 주석을 달아야 한다는 규칙은 매우 어리석다. ➡ 이런 주석은 코드를 복잡하게 만들며, 거짓이 포함되고, 혼동과 무질서를 초래한다.


<details>
<summary>목록 4-3</summary>

```java
/**
 *
 * @param title cd 제목
 * @param author cd 저자
 * @param tracks cd 트랙 숫자
 * @param durationInMinutes cd 길이(단위: 분)
 */
public void addCD(String title, String author, int tracks, int durationInMinutes) {
  CD cd = new CD();
  cd.title = title;
  cd.author = author;
  cd.tracks = tracks;
  cd.duration = durationInMinutes;
  cdList.add(cd);
}
```
> 이는 모든 함수에 Javadocs를 넣으라는 규칙이 낳은 괴물이다. 아래와 같은 주석은 코드만 헷갈리게 만들며, 거짓이 포함될 가능성과, 잘못된 정보를 제공할 여지를 남긴다.

</details>

- __이력을 기록하는 주석__

  - 때때로 사람들은 모듈을 편집할 때마다 모듈 첫머리에 주석을 추가한다. ➡ 그리하여 모듈 첫머리 주석은 지금까지 모듈에 가한 변경을 모두 기록하는 일종의 일지 혹은 로그가 된다.
  - 예전에는 모든 모듈 첫머리에 변경 이력을 기록하고 관리하는 관례가 바람했지만, 이제는 혼란만 가중할 뿐이므로, 완전히 제거하는 편이 좋다.

- __있으나 마나 한 주석__

  - 때때로 있으나 마나 한 주석을 접한다. ➡ 너무 당연한 사실을 언급하며 새로운 정보를 제공하지 못하는 주석이다.
    - `protected AnnualDateRule();	// 기본 생성자`
    - `private int dayOfMonth;	// 월 중 일자`
  - 예시와 같은 주석은 지나친 참견이라, 개발자가 주석을 무시하는 습관으로 이어진다. ➡ 코드를 읽으며 자동으로 주석을 건너뛴다. 결국은 코드가 바뀌면서 주석은 거짓말로 변한다.
  - 있으나 마나한 주석을 달기보다, 코드를 정리하는 습관을 들여 행복한 프로그래머가 되어라.!

- __무서운 잡음__

  - 떄로는 Javadocs도 잡음이다.

  - ```java
    /** The name. */
    private String name;
    
    /** The version. */
    private String version;
    
    /** The licenceName. */
    private String licenseName;
    
    /** The version. */
    private String info;
    ```

  - 위에 나타난 주석은 어떤 목적을 수행할까? 없다. ➡ 단지 문서를 제공해야 한다는 잘못된 욕심으로 탄생한 잡음이다.

- __함수나 변수로 표현할 수 있다면 주석을 달지 마라__

  - ```java
    // 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
    if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem()))
    ```

  - 위의 예제에서 주석을 없애고 다시 표현하면

  - ```java
    ArrayList moduleDependees = smodule.getDependSubsystem();
    String outSubSystem = subSysMod.getSubSystem();
    if (mouduleDependees.contains(ourSubSystem))
    ```

  - 이렇게 주석이 필요하지 않도록 코드를 개선할 수 있으며, 더 좋다.

- __위치를 표시하는 주석__

  - 때때로 프로그래머는 소스파일에서 특정 위치를 표시하려 주석을 사용한다.
  - 극히 드물지만, `//Actions ///////////////////////` 이와 같은 배너 아래 특정 기능을 모아놓으면 유용한 경우도 있긴하다.
  - 그.러.나 위와 같은 주석은 가독성만 낮추므로 제거해야 마땅하다. ➡ 특히, 뒷부분에 슬래시(/)로 이어지는 잡음은 제거하는 편이 좋다.
    - 너무 자주 사용하지 않는 배너는 눈에 띄며 주의를 환기한다.
    - 그러므로 반드시 필요할 때만, 아주 드물게 사용하는 편이 좋다. 
    - 배너를 남용하면 독자가 흔한 잡음으로 여겨 무시할 수 있다.

- __닫는 괄호에 다는 주석__

  - 때로는 프로그래머들이 닫는 괄호에 특수한 주석을 달아놓는다. (목록 4-6)
  - 중첩이 심하고 장황한 함수라면 의미가 있을지도 모르지만, 작고 캡슐화된 함수에는 잡음일 뿐이다. ➡ 그러므로 닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신에 함수를 줄이려 시도하자.

<details>
<summary>목록 4-6 wc.java</summary>

```java
public class wc {
  public static void main(String[] args) {
    BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
    String line;
    int lineCount = 0;
    int charCount = 0;
    int wordCount = 0;
    try {
      while ((line == in.readLine()) != null) {
        lineCount ++;
        charCount += line.length();
        String words[] = line.split("\\W");
        wordCount += words.length;
      } // while
      System.out.println("wordCount = " + wordCount);
      System.out.println("lineCount = " + lineCount);
      System.out.println("charCount = " + charCount);
    }   // try
    catch (IOException e) {
      System.err.println("Error: " + e.getMessage());
    }   // catch
  }     // main
}
```

</details>

- __공로를 돌리거나 저자를 표시하는 주석__

  - 소스 코드 관리 시스템은 누가 언제 무엇을 추가했는지 기억한다. ➡ 굳이 저자 이름으로 주석코드를 달 필요가 없다.

- __주석으로 처리한 코드__

  - 주석으로 처리한 코드만큼 밉살드런 관행도 드물다. ➡ 주석으로 코드는 작성하지 마라.
  - 주석으로 처리된 코드는 다른 사람들이 지우기를 주저한다. ➡ 이유가 있어 남겨놓았으리라고, 중요하니까 지우면 안 된다고 생각한다.
  - 어차피 소스 코드 관리 시스템이 우리를 대신해 코드를 기억해주므로 과감하게 코드를 삭제해라!

- __HTML 주석__

  - 소스 코드에서 HTML 주석은 혐오 그 자체이다. (HTML 주석은 편집기/IDE에서조차 읽기 어렵다.)
  - 도구로 주석을 뽑다 웹 페이지에 올릴 작정이라면 주석에 HTML 태그를 삽입해야하는 책임은 프로그래머가 아니라 도구가 져야한다.

- __전역 정보__

  - 주석을 달아야 한다면 근처에 있는 코드만 기술하라. ➡ 코드 일부에 주석을 달면서 시스템의 전반적인 정보를 기술하지 마라.

  - ```java
    /**
     * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
     *
     * @param fitnessePort
     */
    public void setFitnessePort(int fitnessePort) {
      this.fitnessePort = fitnessePort;
    }
    ```

  - 위의 주석은 바로 아래에 있는 함수가 아니라 시스템 어딘가에 있는 다른 함수를 설명한다. ➡ 즉, 포트 기본값을 설정하는 코드가 변해도 아래 주석이 변하리라는 보장은 없다.

- __너무 많은 정보__

  - 주석에다 흥미로운 역사나 관련 없는 정보를 장황하게 늘어놓지 마라. ➡ 독자에게 불필요하며 불가사의한 정보를 제공하여 혼란만 야기한다.

- __모호한 관계__

  - 주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다. ➡ 이왕 공들여 주석을 달았다면 적어도 독자가 주석과 코드를 읽었을 때 이해할 수 있게 하자

  - ```java
    /*
     * 모든 픽셀을 담을 만큼 충분한 배열로 시작한다 (여기에 필터 바이트를 더한다).
     * 그리고 헤더 정보를 위해 200바이트를 더한다.
     */
    this.pngBytes = new byte[((this.width +1) * this.height * 3) + 200];
    ```

  - 여기서 필터 바이트란 무엇일까? ➡ 주석을 다는 목적은 코드만으로 설명이 부족해서인데, 여기선 주석에 또 다시 의문을 가질 여지를 주고 있다.

- __함수 헤더__

  - 짧은 함수는 긴 설명이 필요 없다. 짧고 한 가지만 수행하며 이름을 잘 붙인 함수가 주석으로 헤더를 추가한 함수보다 훨씬 좋다.

- __비공개 코드에서 Javadocs__

  - 공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 Javadocs는 쓸모가 없다. ➡ 유용하지 않을 뿐만 아니라 Javadocs 주석이 요구하는 형식으로 인해 코드만 보기 싫고 산만해질 수 있다.

---

