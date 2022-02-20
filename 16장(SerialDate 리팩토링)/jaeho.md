# 16장 SerialDate 리팩토링

---

> [JCommon 라이브러리](http://www.jfree.org/jcommon)

> __SerialDate 란?__
>
> 날짜를 표현하는 자바 클래스이며, java.util.Date의 불편함을 해소하고자 만든 라이브러리.
>
> 하루 중 시각, 시간대에 무관하게 특정 날짜를 표기하기 위해, 즉 시간 기반 날짜 클래스가 아닌 순수 날짜 클래스를 만들기 위해 만듦

### 1. 첫째, 돌려보자

- `SerialDateTests` 라는 클래스는 다누이 테스트 케이스 몇 개를 포함하고 있으며, 돌려보면 실패하는 테스트 케이스는 없다.
- 하지만, 테스트 케이스를 보면 모든 경우에 대해 점검하지 않음을 알 수 있다.
- 클로버를 이용하여 단위테스트를 실행한 결과, 실행 가능한 문장 185개중 단위테스트가 실행하는 문장은 91개에 불과했다. (대략, 50%)

> __클로버__
>
> 코드 커버리지 분석 도구

> __경계 조건__
>
> 에러냐 아니냐의 경계가 되는 조건을 경계조건이라고 한다.
>
> CORRECT: 경계조건을 생각할 때 고려해야 할 7가지
>
> - Conformance 준수: 데이터가 특정 양식을 따르는지 검증해야함.
> - Ordering 순서: 데이터 순서 혹은 컬렉션에서 특정 데이터의 위치등이 사용되는 코드는 쉽게 잘못 될 수 있어 특히 주의해야함.
> - Range 범위: 프로그램이 허용할 수 있는 데이터 범위를 넘어지 검증해야함.
> - Reference 참조: 범위를 넘어서는 것을 참조하는가? 외부 의존성이 있는가? 특정 상태에 있는 객체를 의존하고 있는가? 반드시 존재해야하는 조건들이 있는가? 등의 사전조건들을 검증해야함.
> - Existence 존재: 스스로에게 '주어진 값이 존재하는가?'를 물어봄으로써 많은 잠재적 결함을 발견할 수 있음.
> - Cardinality 기수: 어떤 컬렉션을 다룰때는 갯수의 개념에 대한 테스트를 고려해야함.
> - Time 시간: 시간에 관해 고려해야할 테스트케이스를 확인해야함. (상대적시간, 절대적시간, 동시성)

### 2. 둘째, 고쳐보자

- 코드를 고칠 때마다 JCommon 단위테스트와 자신이 짠 테스트를 실행한다. ==> 즉, 변경한 코드는 JCommon 프레임워크에서 사용해도 문제 없다.
- __p.448 line.1__
  - 라이선스 정보, 저작권, 작성자, 변경 이력등의 법적인 정보는 필요하므로 라이선스 정보와 저작권은 보존한다.
  - 그러나 변경이력은 없애도 된다. ==> 현재는 소스 코드 제어 도구(Git 등)를 사용하므로
- __p.449 line.61__
  - import문은 `java.text.*`와 `java.util.*` 로 줄여도 된다.
- __p.449 line.67__
  - javadoc 주석은 HTML 태그를 사용한다. 한 소스코드에 여러 언어를 사용하는 것은 탐탁지 못하니 주석 전부를 `<pre>`로 감싸자 ==> 이렇게 되는 소스코드에 보이는 형식이 javadoc에 그대로 유지된다.
- __p.450 line.86__
  - 클래스 선언부분. 
  - `SerialDate` 클래스에 serial이라는 단어가 들어가는 이유는, '일련번호'를 사용해 클래스를 구현했기 때문.
  - 글쓴이는 두 가지 이유로 꺼림칙하다고 한다.
    1. '일련번호'라는 용어는 정확하지 못하다. ==> 일련번호보다는 '상대 오프셋'이라는 단어나 '서수'가 더 서술적으로 느껴진다고 함(?)
    2. SerialDate라는 이름은 구현을 암시하지만 실상은 추상 클래스이다 ==> 이름의 추상화 수준이 올바르지 못하다고 생각한다고함
  - 그래서 `SerialDate` 보다는 `Date` 가 낫다고 생각하지만, 라이브러리에 이미 Date라는 클래스가 너무나 많다. ==> 결국엔 `DayDate`라는 이름을 쓰기로 함.
- __p.481 목록 B-3__
  - `MonthConstants` 클래스는 달을 정의하는 static final 상수모음에 불과하다.
  - 상수는 상속하여 사용하기 보다, enum 형식으로 정의하는 것이 마땅하다.
  - int로 받던 메서드는 이제 Month로 받을 수 있으며 메서드를 줄이고 오류 검사 코드도 필요하지 않게 되었다. ==> 그만큼 시간 소모는 필요
- __p.450 line.91__
  - `serialVersionUID` 변수는 직렬화를 제어한다. ==> 이 값을 변경하면 이전 소프트웨어 버전에서 직렬화한 DayDate를 인식하지 못한다. (InvalidClassException 발생)
  - `serialVersionUID` 변수를 선언하지 않으면 컴파일러가 자동으로 생성한다. ==> 모듈을 변경할 때마다 변수 값이 달라진다.
  - 문서에서는 직접 선언을 권하지만, 저자는 깜빡하고 변경하지 못해 디버깅을 해야됨을 감안하여 자동 제어가 안전하게 느낀다고 함.

> __Serializer 직렬화__
>
> - 자바 직렬화란 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술과 바이트로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)을 아울러서 이야기합니다.
> - 시스템적으로 이야기하자면 JVM(Java Virtual Machine 이하 JVM)의 메모리에 상주(힙 또는 스택)되어 있는 객체 데이터를 바이트 형태로 변환하는 기술과 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태를 같이 이야기합니다.
>
> 참고: https://devlog-wjdrbs96.tistory.com/268

- __p.450 line.93__

  - 불필요한 주석 삭제

- __p.450 line.97, 100__

  - 97, 100 라인의 주석은 앞서 설명한 일련번호를 언급한다.

  - 각 라인의 변수는 `DayDate` 클래스가 표현할 수 있는 최초 날짜와 최후 날짜를 의미하며, 좀 더 깔끔하게 표현하면 다음과 같다.

  - ```java
    public static final int EARLIEST_DATE_ORDINAL = 2;	// 1/1/1900
    public static final int LATEST_DATE_ORDINAL = 2958465;	// 12/31/9999
    ```

  - EARLIEST_DATE_ORDINAL의 값이 2인 이유는 엑셀에서 날짜를 표현하는 방식과 관련이 있다고 한다.

  - 그런데 이 부분이 SpreadsheetDate의 구현과 관련되지, DayDate와는 아무 상관이 없다. ==> 따라서, SpreadsheetDate로 옮겨져야 한다.

- __p.450 line.104, 107__

  - `DayDate` 는 언제까지나 추상 클래스로, 구체적인 구현 정보를 포함할 필요가 없다 ==> 최소 년도(MINIMUM_YEAR_SUPPORTED)와 최대 년도(MAXIMUM_YEAR_SUPPORTED)를 지정할 이유도 없음
  - 그래서 이 두 변수를 SpreadsheetDate 클래스로 옮길 예정이었으나, `RelativeDayOfWeekRule` 클래스에서도 이 두 변수를 이용하고 있었음.
  - DayDate 자체를 훼손하지 않으면서 구현 정보를 전달할 방법이 필요 ==> 결론적으로, ABSTRACT FACTORY 패턴을 적용해 DayDate 인스턴스를 생성하는 DayDateFactory를 만들어 해결.

- __p.450 line.109__

  - DayDate에 나타나는 요일 상수를 enum으로 변경한다.

- __p.451 line.140__

  - LAST_DAY_OF_MONTH 부터 시작하는 일련의 배열은 변수 이름만으로 의미가 확실하므로 주석을 삭제.
  - LAST_DAY_OF_MONTH 변수는 public 변수일 필요가 없다.
  - AGGREGATE_DAYS_TO_END_OF_MONTH 및 LEAP_YEAR_AGGREGATE_DAYS_TO_END_MONTH는 사용하지 않으므로, 제거
  - AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH 변수는 사용되는 위치(SpeadsheetDate)에 최대한 가깝게 옮겨라

- __p.451 line.162 ~ p.452 line.205__

  - 나오는 상수는 enum으로 변환가능

  - ```java
    public enum WeekInMonth {
        FIRST(1), SECOND(2), THRID(3), FOURTH(4), LAST(0);
        public final int index;
        
        WeekInMonth(int index) {
            this.index = index;
        }
    }
    ```

- __p.452 line.177 ~ 187__

  - 여기서 나오는 상수는 다소 모호함.
  - INCLUDE_NONE, FIRST, SECOND, BOTH 상수는 수학적으로 개구간, 반개구간, 폐구간이라는 개념이다
  - 이러한 수학적 명칭이 의도를 더 분명하게 하기 위해서, enum이름을 `DateInterval` 로 결정했으며, CLOSED, CLOSED_LEFT, CLOSED_RIGHT, OPEN로 정의 했다.

- __p.452 line.190 ~ 205__

  - 여기서의 상수는 주어진 날짜를 기준으로 특정 요일을 계산할 때 사용한다.
  - enum이름을 `WeekdayRange`로 결정하고 직전요일, 다음 요일, 가장 가까운 요일을 LAST, NEXT, NEAREST로 결정했다.

- __p.452 line.208__

  - description 변수는 더 이상 사용하지 않으므로 변수와 관련 메서드를 삭제함.

- __p.452 line.213__

  - 기본 생성자도 제거 ==> 기본 생성자는 컴파일러가 기본적으로 생성한다.

- __p.453 line.216 ~ 238__

  - isValidWeekDayCode 메서드는 건너 뛰자. ==> Day enum을 생성할 때 삭제했기 때문.

- __p.453 line.242 ~ p.454 line.270__

  - 주석 제거
  - 인수와 변수 선언에서 final 키워드 삭제

- __p.454 line.259 ~ 263__

  - for문 내의 두번의 if문을 `||` 연산자를 이용해 한번의 if문으로 변경
  - stringToWeekDayCode 메서드는 DayDate가 아닌 Day의 구문분석 메서드이므로 Day로 옮긴다.

- __p.454 line.288 ~ p.455 line.316__

  - getMonths가 두 개 나오는데, 첫 번째 getMonths는 두 번째 getMonths를 호출하며 두 번째 getMonths를 호출하는 메서드는 첫 번째 getMonths뿐이다. ==> 두 메서드를 하나로 합쳐 단순화함.

- __p.455 line.326 ~ 346__

  - `isValidMonthCode` 삭제 ==> Month enum을 만들면서 필요가 없어졌다.

- __p.456 line.356 ~ 375__

  - monthCodeToQuarter 메서드는 기능 욕심으로 보인다.

  - 대신, Month에 quarter라는 메서드를 추가했다.

  - ```java
    public int quarter() {
        return 1 + (index-1)/3;
    }
    ```

  - 이렇게 수정하게 되면 Month가 너무 커지게 되므로 Day enum과 일관성을 유지하도록 DayDate에서 분리해 독자적인 파일로 만들어 주었다.

- __p.456 line.377 ~ p.457 line.426__

  - monthCodeToString이라는 메서드는 두 개는 [위와 마찬가지로](__p.454 line.288 ~ p.455 line.316__) 처리한다.

- __p.459 line.459 ~ 517__

  - `isLeapYear` 메서드는 좀 더 서술적인 표현으로 가독성을 높였다.

  - ```java
    public static boolean isLeapYear(int year) {
        boolean fourth = year % 4 == 0;
        boolean hundredth = year % 100 == 0;
        boolean fourHundredth = year % 400 == 0;
        return fourth && (!hundreth || fourHundredth);
    }
    ```

- __p.460 line.519 ~ 536__

  - `leapYearCount` 는 사실상 DayDate에 속하지 않고 spreadsheetDate에 있는 메서드 두 개뿐이 사용하지 않으므로 이동시킨다.

- __p.460 line.538 ~ 560__

  - `lastDayOfMonth` 메서드는 LAST_DAY_OF_MONTH 배열을 사용하는데, LAST_DAY_OF_MONTH 배열은 month enum을 사용하므로 이 메서드도 여기로 옮기고 좀 더 서술적인 표현으로 변경

  - ```java
    public static int lastDayOfMonth(Month month, int year) {
        if (month == Month.FEBRUARY && isLeapYear(year))
            return month.lastDay() + 1;
        else
            return month.lastDay();
    }
    ```

- __p.461 line.562 ~ 576__

  - `addDays` 메서드는 온갖 DayDate 변수를 사용하므로 static으로 사용하지 말고 인스턴스 메서드로 변경한다.

  - 또한 toSerial 메서드를 호출하므로 toOrdinal로 변경하였다.

  - ```java
    // + 메서드 단순화
    public DayDate addDays(int days) {
        return DayDateFactory.makeDate(toOrdinal() + days);
    }
    ```

- __p.461 line.578 ~ p.462 line.601 + p.462 line.604 ~ 626__

  - [바로 위](__p.461 line.562 ~ 576__)와 마찬가지로 인스턴스 메서드로 변경

  - 알고리즘이 다소 복잡하므로 임시 변수 설명을 사용해 좀 더 읽기 쉽게 고쳤다.

  - ```java
    // p.461 line.578 ~ p.462 line.601
    public DayDate addMonths(int months) {
        int thisMonthAsOrdinal = 12 * getYear() + getMonth().index - 1;
        int resultMonthAsOrdinal = thisMonthAsOrdinal + months;
        int resultYear = resultMonthAsOrdinal / 12;
        Month resultMonth = Month.make(resultMonthAsOrdinal % 12 + 1);
    
        int lastDayOfResultMonth = lastDayOfMonth(resultMonth, resultYear);
        int resultDay = Math.min(getDayOfMonth(), lastDayOfResultMonth);
        return DayDateFactory.makeDate(reusltDay, resultMonth, resultYear);
    }
    
    // p.462 line.604 ~ 626
    public DayDate plusYears(int years) {
        int resultYear = getYear() + years;
        int lastDayOfMonthResultYear = lastDayOfMonth(getMonth(), resultYear);
        int resultDay = Math.min(getDayOfMonth(), lastDayOfMonthInResultYear);
        return DayDateFactory.makeDate(resultDay, getMonth(), resultYear);
    }
    ```

- __p.462 line.628 ~ 660__

  - `getPreviousDayOfWeek` 메서드는 올바로 동작하지만 다소 복잡하므로 단순화하고 임시 변수 설명을 사용해 좀 더 읽기 쉽게 고침.
  - 또한, 정적 메서드를 인스턴스 메서드로 고쳤으며 중복된 인스턴스 메서드를 제거함.

- __p.464 line.728 ~ p.465 line.740__

  - `getEndOfCurrentMonth` 메서드를 인스턴스 메서드로 바꾸고 몇몇 이름을 고쳤다.

  - ```java
    public DayDate getEndOfMonth() {
        Month month = getMonth();
        int year = getYear();
        int lastDay = lastDayOfMonth(month, year);
        return DayDateFactory.makeDate(lastDay, month, year);
    }
    ```

- __p.465 line.765 ~ p.466 line.781__

  - `weekInMonthOtString` , `relativeToString` 메서드는 테스트 케이스 외에는 아무도 호출하지 않았다 ==> 관련한 메서드와 테스트케이스 전부 삭제

- __p.467 line.829 ~ 836__

  - p.360에서 toSerial을 toOrdinal로 변경했지만, getOrdinalDay로 변경

- __p.467 line.844 ~ 846__

  - spreadsheetDate에서 구현한 toDate를 보면 spreadsheetDate에 의존하지 않는다. 그래서 DayDate 메서드로 옮겼다.

- __목록 B-5 p.500 line.247__

  - 알고리즘이 서수 날짜 시작일의 요일에 암시적으로 의존함 ==> 그렇기에 `getDayOfWeek` 메서드는 DayDate로 옮기지 못한다.
  - _물리적 의존성_ 은 없지만 _논리적 의존성_  이 존재하기 때문이다
  - DayDate에 `getDayOfWeekForOrdinalZero` 라는 추상 메서드를 구현하고 spreadsheetDate에서 Day.SATURDAY를 반환하도록 구현 ==> 이후에는, getDayOfWeek 메서드는 DayDate로 옮긴 후 getOrdinalDay와 getDayOfWeekForOrdinalZero를 호출하게 변경함

> 물리적 의존성 : 변수 대신 함수를 이용해서 함수 간 데이터를 공유하는 것
>
> 논리적 의존성 : 다른 함수에서 사용하는 변수를 사용하여 의존성을 갖는 것

- __p.468 line.895 ~ 899__
  - 같은 주석을 반복할 필요가 없으므로 삭제함.
- __p.469 line.902 ~ 913__
  - compare 메서드 또한 추상 메서드일 필요없다.
  - spreadsheetDate -> DayDate로 옮기고, 이름도 daySince로 바꾸어 분명하게 해주었다. + 테스트 케이스 추가
- __p.469 line.915 ~ p.470 line.980__
  - 나오는 여섯 메서드는 모두 DayDate에서 구현해야 마땅한 추상 메서드이므로, SpreadsheetDate에서 DayDate로 옮겼다.

- __p.470 line.9982 ~ p.471 line.995__
  - isInRange 함수도 DayDate로 끌어올리고 if문 연쇄 대신에 enum 형식을 취해 if문 연쇄를 삭제함.

- __정리__
  1. 처음에 나오는 주석은 너무 오래되었기에 간단하게 고치고 개선했다.
  2. enum을 모두 독자적인 소스 파일로 옮겼다.
  3. 정적 변수(`dateFormatSymbols`)와 정적 메서드(`getMonthNames`, `isLeapYear`, `lastDayOfMonth`)를 `DateUtil` 이라는 새 클래스로 옮겼다.
  4. 일부 추상 메서드를 `DayDate `클래스로 끌어올렸다.
  5. `Month.make`를 `Month.frontInt`로 변경했다. 다른 enum도 똑같이 변경했다. 또한, 모든 enum에 `toInt()` 접근자를 생성하고 idnex 필드를 private로 정의했다.
  6. `plusYears` 와 `plusMonths` 에 흥미로운 중복이 있었다. `correntLastDayOfMonth`라는 새 메서드를 생성해 중복을 없앴다. 그래서 새 메서드가 모두 좀 더 명확해졌다.
  7. 팔방미인으로 사용하던 숫자 1을 없앴다. 모두 `Month.JANUARY.toInt()` 혹은 `Day.SUNDAY.toInt()`로 적절히 변경했다. `SpreadsheetDate` 코드를 살펴보고 알고리즘을 조금 손봤다. 이렇게 얻어진 결과가 _512쪽 목록 B-7 ~ 526쪽 목록 B-16_ 이다.

