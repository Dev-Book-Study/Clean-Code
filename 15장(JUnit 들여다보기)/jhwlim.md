# 📖 15장. Junit 들여다보기

## 🔖 목차

1. [Junit 프레임워크](#01-junit-프레임워크)
2. [결론](#02-결론)

## 들어가기 전에

- 이 장에서는 JUnit 프레임워크에서 가져온 코드를 평가하고, 개선해본다.

## 01. Junit 프레임워크

- 문자열 비교 오류를 파악할 때 유용한 모듈(코드)인 `ComparisonCompactor`를 살펴본다.
- `ComparisonComparctor`는 두 문자열을 받아 차이를 반환한다.

### 테스트코드 살펴보기

```java
// ComparisonCompactorTest (p324~327)
public void testMessage() { ... }
public void testStartSame() { ... }
public void testEndSame() { ... }
public void testSame() { ... }
public void testNoContextStartAndEndSame() { ... }
public void testStartAndEndContext() { ... }
public void testStartAndEndContextWithEllipses() { ... }
public void testComparisonErrorStartSameComplete() { ... }
public void testComparisonErrorEndSameComplete() { ... }
public void testComparisonErrorEndSameCompleteContext() { ... }
public void testComparisonErrorOverlapingMatches() { ... }
public void testComparisonErrorOverlapingMatchesContext() { ... }
public void testComparisonErrorOverlapingMatches2() { ... }
public void testComparisonErrorOverlapingMatches2Context() { ... }
public void testComparisonErrorWithActualNull() { ... }
public void testComparisonErrorWithActualNullContext() { ... }
public void testComparisonErrorWithExpectedNull() { ... }
public void testComparisonErrorWithExpectedNullContext() { ... }
public void testBug609972() { ... }
```

- 위 테스트 케이스로 ComparisonCompactor 모듈에 대한 코드 커버리지 분석시 100% 가 나온다. <br> 이는 테스트 케이스가 모든 행, 모든 if문, 모든 for문을 실행한다는 의미이다.

> 다음에 설명하는 `ComparisonCompactor` 원본을 살펴보면, public 메서드는 `compact()` 하나 인데, 이를 테스트하기 위해서 위와 같이 많은 테스트 케이스를 작성했다. 테스트를 작성하는 것의 중요성은 책을 읽으면서 계속 느끼고 있지만, 테스트를 작성하는 것은 역시 어려운 일인 것 같다.

### `ComparisonCompactor` (원본)

```java
// p327~329
public class ComparisonCompactor {

    private static final String ELLIPSIS = "...";
    private String final String DELTA_END = "]";
    private String final STring DELTA_START = "[";

    private int fContextLength;
    private String fExpected;
    private String fActual;
    private int fPrefix;
    private int fSuffix;

    public ComparisonCompactor(int contextLength, String expected, String actual) {
        fContextLength = contextLength;
        fExpected = expected;
        fActual = actual;
    }

    public String compact(String message) { ... }

    private String compatString(String source) { ... }
    private void findCommonPrefix() { ... }
    private void findCommonSuffix() { ... }
    private String computeCommonPrefix() { ... }
    private String computeCommonSUffix() { ... }
    private boolean areStringEqual() { ... }
}
```

### 원래 코드를 어떻게 개선하면 좋을까?

1. 접두어 `f` <br> → 오늘날 사용하는 개발 환경에서는 변수 이름에 범위를 명시할 필요가 없다. 접두어 `f`는 중복되는 정보다.

   ```java
   // before
   private int fContextLength;
   private String fExpected;
   private String fActual;
   private int fPrefix;
   private int fSuffix;

   // after
   private int contextLength;
   private String expected;
   private String actual;
   private int prefix;
   private int suffix;
   ```

2. compact 함수 시작부에 캡슐화되지 않은 조건문 <br> → 의도를 명확히 표현하기 위해 조건문을 캡슐화한다. 즉, 조건문을 메서드로 뽑아내 적절한 이름을 붙인다.

   ```java
   // before
   if (expected == null || actual == null || areStringEqual()) { ... }

   // after
   if (shouldNotCompact()) { ... } // 메서드로 분리한다.
   ```

3. 함수에서 멤버 변수와 이름이 똑같은 변수를 사용한다. <br> → 서로 다른 의미이기 때문에 이름은 명확하게 붙인다.

   ```java
   // before
   String expected = compactString(this.expected);
   String actual = compactString(this.actual);

   // after
   String compactedExpected = compactString(expected);
   String compactActual = compactString(actual);
   ```

   > 멤버 변수와 이름이 똑같은 변수를 사용해서 문제가 있다기보다는, `compactString()` 메서드를 통해 한 번 가공되는 값인데 똑같은 이름을 사용하기 때문에 적절하지 않아보인다.

4. 부정문은 긍정문보다 이해하기 약간 더 어렵다. <br> → 긍정으로 만들고 조건문을 반전한다.

   ```java
   // before
   if (shouldNotCompact()) {
       // 압축을 진행하지 않는다.
   }
   // 압축을 진행한다.

   // after
   if (canBeCompact()) { // `shouldNotCompact()`의 조건을 반전하고, 메서드 이름을 `canBeCompact()`로 수정한다.
       // 압축을 진행한다.
   } else {
       // 압축을 진행하지 않는다.
   }
   ```

   > else 문은 삭제하는 것이 더 좋을 것 같다.
   > 또한, 위에서 제시한 코드보다는 아래와 같이 `shouldNotCompact()` 부분만 `!canBeCompact()`로 바꾸는 것이 더 좋지 않을까?

   ```java
   if (!canBeCompact()) {
       // 압축을 진행하지 않는다.
   }
   // 압축을 진행한다.
   ```

   > 그리고 압축을 진행하지 않는다는 것을 Exception을 이용해서 처리하는 것도 코드가 좀 더 깔끔해질 수 있는 방법이 아닐까 생각된다.

5. `compact()` 함수 이름이 적절하지 않다. 문자열을 압축하는 함수이지만 실제로는 `canBeCompacted()`가 `false`이면 압축하지 않는다. 함수에 compact라는 이름을 붙이면 오류 점검이라는 부가 단계가 숨겨진다. 또한, 함수는 단순히 압축된 문자열이 아니라 형식이 갖춰진 문자열을 반환한다. <br> → `formatCompactedComparison` 이라는 이름이 적합하다.

   ```java
   // before
   public String compact() { ... }

   // after
   public String fomatCompactedComparison(String message) { ... }
   ```

   > 하지만 추상화 관점에서 public 메서드의 이름이 너무 구체적이지 않은가?

6. 예상 문자열과 실제 문자열을 진짜로 압축하는 부분은 `compactExpectedAndActual()` 메서드로 분리한다. 하지만 형식을 맞추는 작업은 formatCompactedComparison에게 전적으로 맡긴다. `compactExpectedAndActual`은 압축만 수행한다.

   ```java
   // before
   public fomatCompactedComparison() {
       if (canBeCompacted()) {
           findCommonPrefix();
           findCommonSuffix();
           String compactExpected = compactString(expected);
           String compactActual = compactString(actual);
           return Assert.format(message, compactExpected, compactActual);
       } else {
           // 압축하지 않는다.
       }
   }

   // after
   private String compactExpected;
   private String compactActual;

   public fomatCompactedComparison() {
       if (canBeCompacted()) {
           compactExpectedAndActual();
           return Assert.format(message, compactExpected, compactActual);
       } else {
           // 압축하지 않는다.
       }
   }

   private void compactExpectedAndActual() { ... }
   ```

7. `compactExpectedAndActual()` 내부에서 함수의 사용방식이 일관적이지 못하다. 또한, 멤버 변수 이름도 좀 더 정확하게 바꾼다.

   ```java
   // before
   private int prefix;
   private int suffix;

   private void compactExpectedAndActual() {
       findCommonPrefix();
       findCommonSuffix();
       compactExpected = compactString(expected);
       compactActual = compactString(actual);
   }

   private void findCommonPrefix() { ... }
   private void findCommonSuffix() { ... }

   // after
   private int prefixLength;
   private int suffixLength;

   private void compactExpectedAndActual() {
       findCommonPrefixAndSuffix();
       compactExpected = compactString(expected);
       compactActual = compactString(actual);
   }

   private void findCommonPrefixAndSuffix() { ... }
   private int findCommonPrefix() { ... }
   private char charFromEnd(String s, int i) { ... }
   private boolean suffixOverlapsPrefix(int suffixLength) { ... }
   ```

   > 하지만 결국 함수의 사용방식이 일관적이지 못한다는 점은 개선하지 못한 것으로 보이며, 함수를 일관적으로 사용해야 하는 것이 반드시 옳은 것인지 모르겠다. 상황에 따라 값을 반환할 수도 있고, 값을 반환할 필요가 없을 수도 있는데 인위적으로 함수의 사용방식을 일치시키는 것이 오히려 인위적인 것으로 보여진다.

8. prefixLength와 suffixLength는 언제나 1 이상이기 때문에 `compactString()` 내부에서 if문이 필요없다.

   ```java
   // before
   private String compactString(String source) {
       String result = DELTA_START
           + source.substring(prefixLength, source.length() - suffixLength)
           + DELTA_END;
       if (prefixLength > 0) {
           result = computedCommonPrefix() + result;
       }
       if (suffixLength > 0) {
           result = result + computedCommonSuffix();
       }
       return result;
   }

   // after
   private String compactString(String source) {
       return computeCommonPrefix()
           + DELTA_START
           + source.substring(prefixLength, source.length() - suffixLength)
           + DELTA_END
           + computeCommonSuffix();
   }
   ```

### `ComparisonCompactor` (최종 버전)

```java
// p338~341
public class ComparisonCompactor {
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String ELETA_START = "[";

    private int contextLength;
    private String expected;
    private String actual;
    private int prefixLength;
    private int suffixLength;

    public ComparisonCompactor(int contextLength, String expected, String actual) { ... }

    public String formatCompactedComparison(String message) { ... }
    private boolean shouldBeCompacted() { ... }
    private boolean shouldNotBeCompacted() { ... }
    private void findCommonPrefixAndSuffix() { ... }
    private char charFromEnd(String s, int i) { ... }
    private boolean suffixOverlapsPrefix() { ... }
    private void findCommonPrefix() { ... }
    private String compact(String s) { ... }
    private String startingEllipsis() { ... }
    private String startingContext() { ... }
    private String delta(String s) { ... }
    private String endingContext() { ... }
    private String endingEllipsis() {... }
}
```

- 모듈은 일련의 분석 함수와 일련의 조합 함수로 나뉜다. 전체 함수는 위상적으로 정렬했으므로 각 함수가 사용된 직후에 정의된다. 분석 함수가 먼저 나오고 조합 함수가 그 뒤를 이어서 나온다.
- 코드를 리팩터링하는 과정에서 원래 했던 변경을 되돌리는 경우가 흔하다. 리팩터링은 코드가 어느 수준에 이를 때까지 수많은 시행착오를 반복하는 작업이기 때문이다. :point_right: 실제로 위에서 정리했던 [원래 코드를 어떻게 개선하면 좋을까?](#원래-코드를-어떻게-개선하면-좋을까) 에서 개선했던 코드가 최종 버전에 반영되지 않은 내용들도 존재한다.

## 02. 결론

- 세상에 개선이 불필요한 모듈은 없다. 코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있다. :point_right: 아무리 잘 짜여진 모듈(코드)이라고 하더라도 개선할 부분이 있다면 개선하라!
