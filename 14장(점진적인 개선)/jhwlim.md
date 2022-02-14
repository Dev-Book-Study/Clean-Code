# 📖 14장. 점진적인 개선

## 🔎 요약

- 이 장은 점진적인 개선을 보여주는 사례를 통해 코드가 개선되는 과정을 보여준다.
- **깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다.** 이 장에서는 점진적으로 코드를 개선하는 방법으로서 TDD 기법을 적용하였다.
- **코드는 언제나 최대한 깔끔하고 단순하게 정리하자. 절대로 썩어가게 방치하면 안 된다.**
  - 소프트웨어 설계는 **분할**만 잘해도 품질이 크게 높아진다.
  - **적절한 장소를 만들어 코드만 분리**해도 설계가 좋아진다.
  - **관심사를 분리**하면 코드를 이해하고 보수하기 훨씬 쉬워진다.

## 🔖 목차

1. [Args 구현](#01-args-구현)
2. [Args: 1차 초안](#02-args-1차-초안)
3. [String 인수](#03-string-인수)
4. [결론](#04-결론)

## 01. Args 구현

- `Args` 클래스
  - 기능 : 명령행 인수의 구문 분석을 위해 main 함수로 넘어오는 문자열 배열을 분석한다.

**Args.java**

```java
// p247~249
public class Args {

    private Map<Character, ArgumentMarshaler> marshalers;
    private Set<Character> argsFound;
    private ListIterator<String> currentArgument;

    public Args(String schema, String[] args) throws ArgsException { ... }

    private void parseSchema(String schema) throws ArgsException { ... }
    private void parseSchemaElement(String element) throws ArgsException { ... }
    private void validateSchemaElementId(char elementId) throws ArgsException { ... }
    private void parseArgumentStrings(List<String> argsList) throws ArgsException { ... }
    private void parseArgumentCharacters(String argChars) throws ArgsException { ... }
    private void parseArgumentCharacter(char argChar) throws ArgsException { ... }

    public boolean has(char arg) { ... }
    public int nextArgument() { ... }
    public boolean getBoolean(char arg) { ... }
    public String getString(char arg) { ... }
    public int getInt(char arg) { ... }
    public double getDouble(char arg) { ... }
    public String[] getStringArray(char arg) { ... }
}
```

**ArgumentMarshaler.java**

```java
// p250
public interface ArgumentMarshaler {

    void set(Iterator<String> currentArgument) throws ArgsException;
}
```

- `BooleanArguementMarshaler`, `StringArgumentMarshaler`, `IntegerArgumentMarshaler`, `DoubleArgumentMarshaler`, `StringArrayArgumentMarshaler` 는 각각 `ArgumentMarshaler` 인터페이스를 구현한다. (`getXXX()` static 메서드 구현 포함)

**ArgsException.java**

```java
// p251~253
public class ArgsException extends Exception {

    private char errorArgumentId = '\0';
    private String errorParameter = null;
    private ErrorCode errorCode = OK;

    // constructors

    // getter & setter

    public String errorMessage() { ... } // errorCode를 errorMessage로 파싱한다.

    public enum ErrorCode { ... } // errorCode를 문자열 상수로 정의
}
```

- 위의 코드에서 새로운 인수 유형(날짜, 복소수 등)을 추가하는 방법
  1. `ArgumentMarshaler` 에서 새 클래스를 파생해 `getXXX()` 함수를 추가한다.
  2. `parseSchemaElement()` 함수에 새 case 문만 추가한다.
  3. 필요하다면 새 `ArgsException.ErrorCode`를 만들고 새 오류 메시지를 추가한다.
- :star2: 위와 같은 코드를 작성하는 방법
  - **깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다.**

## 02. Args: 1차 초안

### 1차 초안

**Args.java**

```java
// p255~261
public class Args {

    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<>();
    private Map<Character, Boolean> booleanArgs = new HashMap<>();
    private Map<Character, String> stringArgs = new HashMap<>();
    private Map<Character, Integer> intArgs = new HashMap<>();
    private Set<Character> argsFound = new HashSet<>();
    private int currentArgument;
    private String errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode = ErrorCode.OK;

    private enum ErrorCode { ... }

    public Args(String schema, String[] args) throws ParseException { ... }

    private boolean parse() throws ParseException { ... }
    private boolean parseSchema() throws ParseException { ... }
    private void parseSchemaElement(String element) throws ParseException { ... }
    private void validateSchemaElementId(char elementId) throws ParseException { ... }
    private void parseBooleanSchemaElement(char elementId) { ... }
    private void parseIntegerSchemaElement(char elementId) { ... }
    private void parseStringSchemaElement(char element) { ... }
    private boolean isStringSchemaElement(String elementTail) { ... }
    private boolean isBooleanSchemaElement(String elementTail) { ... }
    private boolean isIntegerSchemaElement(String elementTail) { ... }
    private boolean parseArguments() throws ArgsException { ... }
    private void parseArgument(String arg) throws ArgsException { ... }
    private void parseElements(String arg) throws ArgsException { ... }
    private void parseElement(char argChar) throws ArgsException { ... }
    private boolean setArgument(char argChar) throws ArgsException { ... }
    private boolean isIntArg(char argChar) { ... }
    private void setIntArg(char argChar) throws ArgsException { ... }
    private void setStringArg(char argChar) throws ArgsException { ... }
    private boolean isStringArg(char argChar) { ... }
    private void setBooleanArg(char argChar, boolean value) { ... }
    private boolean isBooleanArg(char argChar) { ... }

    public int cardinality() { ... }
    public String usage() { ... }
    public String errorMessage() throws Exception { ... }
    private String unexpectedArgumentMessage() { ... }
    private boolean falseIfNull(Boolean b) { ... }
    private int zeroIfNull(Integer i) { ... }
    private String blankIfNull(String s) { ... }
    private String getString(char arg) { ... }
    private int getInt(char arg) { ... }
    private boolean getBoolean(char arg) { ... }
    private boolean has(char arg) { ... }
    private boolean isValid() { ... }

    private class ArgsException extends Exception { ... }
}
```

위 코드의 문제점

- 인스턴스 변수 개수가 많다.
- `TILT` 와 같은 희한한 문자열, HashSets와 TreeSets, try-catch-catch 블록 등 모두 지저분한 코드에 기여하는 요인이다.

### Boolean만 지원하는 버전

```java
// p261~264
public class Args {
    private String schema;
    private String[] args;
    private boolean valid;
    private Set<Character> unexceptedArguments = new TreeSet<>();
    private Map<Character, Boolean> booleanArgs = new HashMap<>();
    private int numberOfArguments = 0;

    public Args(String schema, String[] args) { ... }

    public boolean isValid() { ... }
    private boolean parse() { ... }
    private boolean parseSchema() { ... }
    private void parseSchemaElement(String element) { ... }
    private void parseBooleanSchemaElement(String element) { ... }
    private boolean parseArguments() { ... }
    private void parseArgument(String arg) { ... }
    private void parseElements(String arg) { ... }
    private void parseElement(char argChar) { ... }
    private void setBooleanArg(char argChar, boolean value) { ... }
    private boolean isBoolean(char argChar) { ... }

    public int cardinality() { ... }
    public String usage() { ... }
    public String errorMessage() { ... }
    private String unexceptedArgumentMessage() { ... }

    public boolean getBoolean(char arg) { ... }
}
```

- 나름대로 괜찮은 코드다. 간결하고 단순하며 이해하기도 쉽다.
- 하지만 이후에 String과 Integer라는 인수 유형을 두 개만 추가했을 뿐인데, 코드가 엄청나게 지저분해졌다.
- 이 책에서는 Boolean과 String을 지원하는 버전도 소개하고 있지만 인수 유형이 추가되면서 코드가 엉망이 되가는 과정을 보여준다. (p264~269)

### 그래서 멈췄다

- Boolean, String, Integer 인수 유형을 추가한 후([1차 초안](#1차-초안)) 기능을 더 이상 추가하지 않고 리팩토링을 진행함.
- 새 인수 유형을 추가하기 위해서는 주요 지점 세 곳에 코드를 추가해야 한다.
  1. 인수 유형에 해당하는 HashMap을 선택하기 위해 스키마 요소의 구문을 분석한다.
  2. 명령행 인수에서 인수 유형을 분석해 진짜 유형으로 반환한다.
  3. getXXX 메서드를 구현해 호출자에게 진짜 유형을 반환한다.
- 인수 유형은 다양하지만 모두가 유사한 메서드를 제공한다. → `ArugmentMarshaler`(인터페이스)

### 점진적으로 개선하다

- ⚠️ 프로그램을 망치는 가장 좋은 방법 중 하나는 개선이라는 이름 아래 구조를 크게 뒤집는 행위다.
- 테스트 주도 개발(TDD) 기법을 사용
  - TDD는 언제 어느 때라도 시스템이 돌아가야 한다는 원칙을 따른다.
  - TDD는 시스템을 망가뜨리는 변경을 허용하지 않는다. 변경을 가한 후에도 시스템이 변경 전과 똑같이 돌아가야 한다.
  - 이를 위해서는 언제든 실행이 가능한 자동화된 테스트 슈트 필요
- 시스템에 자잘한 변경을 가하면서 테스트를 통과하는지 확인한다. 테스트 케이스가 실패하면 다음 단계로 넘어가지 않고 테스트를 통과할 수 있도록 오류를 수정한다.

## 03. String 인수

- String 인수를 추가하는 것은 앞서 진행한 것과 마찬가지로 **한 번에 하나씩 고치면서 테스트를 계속 돌린다.**
- 테스트 케이스가 하나라도 실패하면 다음 변경으로 넘어가기 전에 오류를 수정한다.
- Integer 인수 유형도 같은 과정을 반복한다.

### 첫 번째 리팩토링을 끝낸 버전

**Args.java**

```java
// p288~295
public class Args {

    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<>();
    private Map<Character, ArgumentMarshaler> marshalers = new HashMap<>();
    private Set<Character> argsFound = new HashSet<>();
    private int currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;

    private enum ErrorCode { ... }

    public Args(String schema, String[] args) throws ParseException { ... }

    private boolean parse() throws ParseException { ... }
    private boolean parseSchema() throws ParseException { ... }
    private void parseSchemaElement(String element) throws ParseExcpetion { ... }
    private void validateSchemaElementId(char elementId) throws ParseException { ... }
    private boolean isStringSchemaElement(String elementTail) { ... }
    private boolean isBooleanSchemaElement(String elementTail) { ... }
    private boolean isIntegerSchemaElement(String elementTail) { ... }
    private boolean parseArguments() throws ArgsException { ... }
    private void parseArgument(String arg) throws ArgsException { ... }
    private void parseElements(String arg) throws ArgsException { ... }
    private void parseElement(char argChar) throws ArgsException { ... }
    private boolean setArgument(char argChar) throws ArgsException { ... }
    private void setIntArg(ArgumentMarshaler m) throws ArgsException { ... }
    private void setStringArg(ArgumentMarshaler m) throws ArgsException { ... }
    private void setBooleanArg(ArgumentMarshaler m) { ... }

    public int cardinality() { ... }
    public String usage() { ... }
    public String errorMessage() throws Exception { ... }
    private String unexpectedArgumentMessage() { ... }

    public boolean getBoolean(char arg) { ... }
    public String getString(char arg) { ... }
    public int getInt(char arg) { ... }
    public boolean has(char arg) { ... }
    public boolean isValid() { ... }

    private class ArgsException extends Exception {}

    private abstract class ArgumentMarshaler {
        public abstract void set(String s) throws ArgsException;
        public abstract Object get();
    }

    private class BooleanArgumentMarshaler extends ArgumentMarshaler { ... }
    private class StringArgumentMarshaler extends ArgumentMarshaler { ... }
    private class integerArgumentMarshaler extends ArgumentMarshaler { ... }
}
```

위 코드의 문제점

- 첫머리에 나오는 변수
- `setArgument()`는 유형을 일일이 확인한다.
- 모든 set 함수
- 오류 처리 코드

### 다음 리팩토링 수행

- 리팩토링 과정 : p295~304
- 새로운 인수 유형 추가 예시 : p304~307
- 예외/오류 처리 코드 분리 : p307~310

### 인사이트

- 소프트웨어 설계는 분할만 잘해도 품질이 크게 높아진다.
- 적절한 장소를 만들어 코드만 분리해도 설계가 좋아진다.
- 관심사를 분리하면 코드를 이해하고 보수하기 훨씬 쉬워진다.
- `ArgsException`의 `errorMessage()` 메서드
  - 명백히 SRP 위반이다. Args 클래스가 오류 메시지 형식까지 책임졌기 때문이다.
  - 하지만 `ArgsException` 클래스가 오류 메시지 형식을 처리해야 옳을까? 만약 `ArgsException`에게 맡겨서는 안된다고 생각한다면 새로운 클래스가 필요하다.
  - 하지만 미리 깔끔하게 만들어진 오류 메시지로 얻는 장점을 무시하기 어렵다.

## 04. 결론

- 그저 돌아가는 코드만으로는 부족하다.
- 나쁜 코드보다 더 오랫동안 더 심각하게 개발 프로젝트에 악영향을 미치는 요인도 없다.
- 나쁜 코드는 썩어 문드러진다.
- 물론 나쁜 코드도 꺠끗한 코드로 개선할 수 있다. 하지만 비용이 엄청나게 많이 든다.
- 반면 처음부터 코드를 깨끗하게 유지하기란 상대적으로 쉽다.
- **그러므로 코드는 언제나 최대한 깔끔하고 단순하게 정리하자. 절대로 썩어가게 방치하면 안 된다.**
