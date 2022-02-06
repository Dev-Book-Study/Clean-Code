# 10장. 클래스

이제까지 코드 행과 코드 블록을 올바르게 작성하는 방법에 초점을 맞췄습니다. 

함수를 올바르게 구현하는 방법과 함수가 서로 관련을 맺는 

방식등을 공부했지만, 좀 더 높은 단계(클래스)까지 신경 쓰지 않으면 깨끗한 코드를 작성하기에 어렵기 때문에 이 장에서는 깨끗한 코드를 위한 클래스 작성법에 대해 나와있습니다.





## 10-0. 들어가기 전

이 장에서는 객체 지향의 5대 원칙(`SOLID`)에 대해 주로 나오기 때문에 들어가기 전, 한 번 짚고 가겠습니다.



### 객체 지향적 설계 원칙 (SOLID)

1. `SRP(Single Responsibility Principle)` : 단일 책임 원칙

   **클래스**는 단 **하나의 책임**을 가져야 하며 클래스를 변경하는 이유는 단 하나의 이유이어야 합니다.

2. `OCP(Open-Closed Principle)` : 개방-폐쇄 원칙

   **확장에는 열려** 있어야 하고 **변경에는 닫혀** 있어야 합니다.

3. `LSP(Liskov Substitution Principle)` : 리스코프 치환 원칙

   **상위 타입**의 객체를 **하위 타입**의 객체로 **치환**해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 합니다.

4. `ISP(Interface Segregation Principle)` : 인터페이스 분리 원칙

   **인터페이스**는 그 인터페이스를 사용하는 **클라이언트를 기준**으로 분리해야 합니다.

5. `DIP(Dependency Inversion Principle)` : 의존 역전 원칙

   **고수준 모듈**은 **저수준 모듈**의 **구현에 의존해서는 안됩니다**.





## 10-1. 클래스 체계

클래스를 정의하는 표준 자바 관례의 순서는 다음과 같습니다.

1. `static public 상수`
2. `static private 변수`
3. `private 변수`
   - 공개 변수가 필요한 경우는 거의 없음
4. `public 함수`
5. `private 함수`는 자신을 호출하는 public 함수 직후

이렇게 추상화 수준이 단계적으로 내려가게 프로그래밍 해야 합니다.



### 캡슐화

변수와 유틸리티 함수는 가능한 공개하지 않는편이 낫습니다.

하지만 때로는 변수나 유틸리티 함수를 protected로 선언해 테스트 코드에 접근을 허용하기도 합니다. :arrow_right: 같은 패키지 안에서 테스트 코드가 함수를 호출하거나 변수를 사용해야 한다면 그 함수나 변수를 protected로 선언하거나 패키지 전체로 공개합니다.

다만 이런 **캡슐화를 해제하는 결정은 최후의 수단**입니다.





## 10-2 클래스는 작아야 한다!

3장 **함수에서 나왔던 것 처럼 클래스도 작아야합니다.** 다만, 함수를 물리적인 행 수로 크기를 측정했다면, 클래스는 **클래스가 맡은 책임을 기준으로 측정**해야합니다.

클래스 이름은 해당 클래스 책임을 기술해야 합니다. 때문에 **작명하면서 클래스 크기를 줄이는 것이 첫 번째 관문**이 되며, **간결한 클래스 이름이 떠오르지 않거나 클래스 이름이 모호하다면 클래스 책임이 너무 많은 것**입니다.

예를 들어, 클래스 이름에 **Processor, Manager, Super 등과 같이 모호한 단어가 있다면 클래스가 여러 책임을 가지고 있다는 증거**라고 합니다.



### 단일 책임 원칙

단일 책임 원칙은 클래스를 변경할 이유가 하나여야 한다는 의미입니다.

```java
public class SuperDashboard extends JFrame implements MetaDataUser
    public Component getLastFocusedComponent()
    public void setLastFocused(Component lastFocused)
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()
}
```

위와 같이 짧아보이는 클래스라도 변경할 이유가 두 가지 있습니다.

1. 소프트웨어 버전 정보를 추적 :arrow_right: 버전 정보는 소프트웨어를 출시할 때마다 달라짐
2. JFrame을 상속 받기 때문에 Java의 Swing 컴포넌트를 관리합니다. :arrow_right: Swing의 코드를 변경할 때 마다 해당 클래스도 변경해야 함

때문에 위의 클래스에서 버전에 관련된 내용은 아래 처럼 독자적인 클래스로 만들 수 있습니다.

```java
public class Version {
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()
}
```

이렇게 **단일 책임 원칙은 이해하고 지키기 수월해 보이지만, 실제로는 프로그램이 돌아가면 끝났다고 여기는 경우가 많아 설계 시 가장 무시하게 되는 규칙 중 하나**라고 합니다.

물론 수많은 클래스를 넘나 드는 것이 복잡하다고 생각할 수 있지만, 클래스가 적든 많든 해당 애플리케이션이 작동하는 방식은 비슷합니다. 때문에 **규모가 큰 애플리케이션의 복잡성 문제는 체계적인 정리가 필수인 것이지 클래스가 많아져 복잡해진다는 것이 아닙니다.** :arrow_right: 오히려 클래스나 함수의 이름으로써 해당 로직을 설명할 수 있기 때문에 유지 보수에 더 좋다고 생각합니다.



#### 나의 생각

> 도구 상자를 어떻게 관리하고 싶은가? 작은 서랍을 많이 두고 기능과 이름이 명확한 컴포넌트를 나눠 넣고 싶은가?
> 아니면 큰 서랍 몇개를 두고 모두를 던져 넣고 싶은가?

위의 질문에 크게 공감합니다. 클래스는 될 수 있으면 세세히 쪼개는 것이 추후의 유지 보수는 물론이고 자신의 코드를 읽는 사람에게도 좋다고 생각합니다. :arrow_right: 어플리케이션 규모가 커져 생기는 복잡도는 위의 말 처럼 **체계적인 정리가 더 중요**하다고 생각됩니다.



### 응집도

클래스는 인스턴스 변수 수가 작아야 합니다. 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 합니다. 메서드가 변수를 더 많이 사용할 수록 메서드와 클래스의 응집도가 더 높다고 말합니다.

일반적으로 응집도가 높은 클래스는 바람직하진 않지만, **응집도가 높다는 말은 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미이기 때문에 응집도가 높은 클래스를 선호**하게 됩니다.

3장 함수에서 나온 `함수를 작게, 매개변수 목록을 짧게` 전략을 따르다 보면 때때로 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아집니다. 이런 경우는 새로운 클래스로 쪼개야 한다는 신호입니다. 응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두 세개로 쪼개줍니다.



### 응집도를 유지하면 작은 클래스 여럿이 나온다

큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아집니다. 큰 함수 일부를 작은 함수로 빼내는 경우를 가정할 때, **빼내려는 코드가 큰 함수에 정의된 변수 4개를 사용한다면 변수 4개를 새 함수에 인수로 넘기는 것이 아니라, 4개의 변수를 클래스 인스턴스로 사용함으로써, 클래스를 쪼개면 함수의 인수도 줄일 수 있습니다.**

큰 함수를 작은 함수 여럿으로 쪼개다 보면 프로그램에 점점 더 체계가 잡히고 구조가 투명해집니다.



#### 예시 코드

```java
public class PrintPrimes {

    public static void main(String[] args) {
        final int M = 1000;
        final int RR = 50;
        final int CC = 4;
        final int WW = 10;
        final int ORDMAX = 30;
        int P[] = new int[M + 1];
        int PAGENUMBER;
        int PAGEOFFSET;
        int ROWOFFSET;
        int C;
        int J;
        int K;
        boolean JPRIME;
        int ORD;
        int SQUARE;
        int N;
        int MULT[] = new int[ORDMAX + 1];
        J = 1;
        K = 1;
        P[1] = 2;
        ORD = 2;
        SQUARE = 9;
        
        while (K < M) {
            do {
                J = J + 2;
                if (J == SQUARE) {
                    ORD = ORD + 1;
                    SQUARE = P[ORD] * P[ORD];
                    MULT[ORD - 1] = J;
                }
                N = 2;
                JPRIME = true;
                while (N < ORD && JPRIME) {
                    while (MULT[N] < J)
                        MULT[N] = MULT[N] + P[N] + P[N];
                    if (MULT[N] == J)
                        JPRIME = false;
                    N = N + 1;
                }
            } while (!JPRIME);
            K = K + 1;
            P[K] = J;
        }
        {
            PAGENUMBER = 1;
            PAGEOFFSET = 1;
            while (PAGEOFFSET <= M) {
                System.out.println("The First " + M +
                                   " Prime Numbers --- Page " + PAGENUMBER);
                System.out.println("");
                for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++){
                    for (C = 0; C < CC;C++)
                        if (ROWOFFSET + C * RR <= M)
                            System.out.format("%10d", P[ROWOFFSET + C * RR]);
                    System.out.println("");
                }
                System.out.println("\f");
                PAGENUMBER = PAGENUMBER + 1;
                PAGEOFFSET = PAGEOFFSET + RR * CC;
            }
        }
    }
}
```

위와 같은 딱 봐도 읽기 싫은 복잡한 코드를 아래와 같이 바꿀 수 있습니다.

```java
package literatePrimes;
public class PrimePrinter {
    public static void main(String[] args) {
        final int NUMBER_OF_PRIMES = 1000;
        int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);
        
        final int ROWS_PER_PAGE = 50;
        final int COLUMNS_PER_PAGE = 4;
        RowColumnPagePrinter tablePrinter =
            new RowColumnPagePrinter(ROWS_PER_PAGE,
                                     COLUMNS_PER_PAGE,
                                     "The First " + NUMBER_OF_PRIMES + " Prime Numbers");
        
        tablePrinter.print(primes);
    }
}
```

`PrintPrimes`에서 사용된 다양한 상수들과 변수, 로직들을 `RowColumnPagePrinter`와 `PrimeGenerator`로 분리하였습니다.

`RowColumnPagePrinter`와 `PrimeGenerator` 클래스까지 살펴본다면, 클래스가 늘어나고 코드의 전체적인 양이 아래와 같은 이유로 많아졌지만 이점이 있습니다.

1. 좀 더 길고 서술적인 변수 이름을 사용해 코드의 양이 늘어남 :arrow_right: 서술적은 변수 이름을 통해 코드 해석에 도움을 줌
2. 코드에 주석을 추가하는 수단으로 함수 선언과 클래스 선언을 활용하여 파일이나 코드의 양이 늘어남 :arrow_right: 마찬가지로 해당 로직은 해당 함수나 클래스만 파악하면 되기 때문에 코드 해석에 도움을 줌
3. 가독성을 높이고자 공백 추가하여 코드의 양이 늘어남 :arrow_right: 마찬가지로 코드 해석에 도움을 줌

만약 로직이나 서비스를 바꾸어야 한다면 이전에는 PrintPrimes에서 하나 하나 찾아 바꿨다면, **위와 같이 클래스와 함수를 나눠 책임을 분할하게 되면 연관된 해당 클래스나 함수만을 변경하면 되기 때문에 유지보수에 엄청난 도움**이 됩니다.



#### 나의 생각

아래와 같은 DTO 객체의 사용도 비슷한 경우가 되지 않을까 싶습니다.

```java
public class TestController {
    final private testService;
    
    ...
    
    // 기존 사용법
    public class Test() {
        return testService.test(1, 2, 3, 4);
    }
    
    // DTO 객체로 메서드의 인수를 줄임
    public class Test() {
        NumDto numDto = new NumDto(1, 2, 3, 4);
        
        return testService.test(numDto);
    }
}

```





## 10-3. 변경하기 쉬운 클래스

대다수 시스템의 지속적인 변경은 필수불가결한 요소입니다. 그러나 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따릅니다. 때문에 **클래스를 분할하여 체계적으로 정리하면 단위테스트가 쉬워지고 그에 따라 변경에 수반하는 위험을 낮출 수 있습니다.**



### 예시 코드

만약 아래와 같은 Sql 클래스가 있다고 가정하겠습니다.

```java
public class Sql {
    public Sql(String table, Column[] columns)
        public String create()
        public String insert(Object[] fields)
        public String selectAll()
        public String findByKey(String keyColumn, String keyValue)
        public String select(Column column, String pattern)
        public String select(Criteria criteria)
        public String preparedInsert()
        private String columnList(Column[] columns)
        private String valuesList(Object[] fields, final Column[] columns)
        private String selectWithCriteria(String criteria)
        private String placeholderList(Column[] columns)
}
```

만약 위 클래스가 미완성이라 update같은 SQL 기능을 만든다고 했을 때, **위 클래스를 고치게 되면 다른 코드를 망가뜨릴 위험이 존재하여, 테스트도 완전히 다시** 해야합니다.

또한 **select문을 수정할 때나, insert문을 수정할 때도 동일하게 위 클래스를 수정해야하기 때문에 단일 책임 원칙도 위반**하게 됩니다.

때문에 아래와 같은 방법으로 리팩토링할 수 있습니다.

```java
abstract public class Sql {
    public Sql(String table, Column[] columns)
    abstract public String generate();
}

public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns)
        
    @Override
    public String generate();
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns)
    @Override
    public String generate()
}

public class InsertSql extends Sql {
    public InsertSql(String table, Column[] columns, Object[] fields)
    @Override
    public String generate()
    private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql {
    public SelectWithCriteriaSql(String table, Column[] columns, Criteria criteria)
    @Override
    public String generate()
}

public class SelectWithMatchSql extends Sql {
    public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern)
    @Override
    public String generate()
}

public class FindByKeySql extends Sql
    public FindByKeySql(String table, Column[] columns, String keyColumn, String keyValue)
    @Override
    public String generate()
}

public class PreparedInsertSql extends Sql {
    public PreparedInsertSql(String table, Column[] columns)
    @Override public String generate() {
    private String placeholderList(Column[] columns)
}

    
// 유틸 클래스
public class Where {
    public Where(String criteria)
    public String generate()
}

// 유틸 클래스
public class ColumnList {
    public ColumnList(Column[] columns)
    public String generate()
}
```

위 코드는 아래의 방법으로 분리한 것입니다.

- 각 SQL문에 해당하는 함수를 추상 클래스 Sql을 상속하는 클래스들로 분리
- 해당 함수에서 사용된 비공개 함수들은 해당 클래스의 함수로 분리
- 공통으로 사용하는 Where, ColumnList는 유틸리티 클래스로 분리

이렇게 분리하게 되면 아래와 같은 장점이 있습니다.

1. 코드의 이해가 쉬워짐
2. 각 기능의 함수들을 각각의 클래스로 분리하면서 다른 로직을 망가뜨릴 위험성이 사라짐 :arrow_right: `SRP(Single Responsibility Principle)` : 단일 책임 원칙 준수
3. 테스트 코드를 작성하기 쉬워짐
4. 새로운 기능을 추가해도 기존의 코드를 변경하지 않아도 됨 :arrow_right: `OCP(Open-Closed Principle)` : 개방-폐쇄 원칙 준수





## 10-4. 변경으로부터 격리

상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 불안정하고 결합도가 높아 코드 테스트가 어렵습니다. 그래서 아래의 예시와 같이 인터페이스와 추상 클래스를 적절히 사용하는 것이 좋습니다.



### 예시 코드

```java
// 증권 거래소 인터페이스
public interface StockExchange {
    Money currentPrice(String symbol);
}

// 포트폴리오 클래스
public Portfolio {
    private StockExchange exchange;
    public Portfolio(StockExchange exchange) {
        this.exchange = exchange;
    }
    ...
}

// 포트 폴리오 테스트
public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;
    
    @Before
    protected void setUp() throws Exception {
        exchange = new FixedStockExchangeStub();
        exchange.fix("MSFT", 100);
        portfolio = new Portfolio(exchange);
    }
    

    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
        portfolio.add(5, "MSFT");
        Assert.assertEquals(500, portfolio.value());
    }
}
```

위의 코드처럼 거래소를 인터페이스로 정의하면 해당 인터페이스를 구현한 다양한 거래소가 올 수 있으므로 **유연성과 재사용성이 높아집니다.** 때문에 **결합도가 낮아 각 시스템 요소가 다른 요소로부터 격리되어 있어 이해하기가 더 쉽습니다.**

이렇게 결합도를 최소로 줄이면 자연스럽게 `DIP(Dependency Inversion Principle)` : 의존 역전 원칙도 준수하게 됩니다.



---

참고 : Clean Code - 로버트 C. 마틴

