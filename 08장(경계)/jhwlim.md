# 📖 8장. 경계

## 🔎 요약
- 외부 코드를 깔끔하게 처리하는 방법 (경계를 관리하는 방법)
    1. 새로운 클래스로 경계를 감싸자. 👉 일급 컬렉션을 사용하자.
    2. Adapter 패턴을 통해 우리가 원하는 인터페이스로 변환하자.
- 학습 테스트를 하자.

## 🔖 목차

1. [외부 코드 사용하기](#01-외부-코드-사용하기)
2. [경계 살피고 익히기](#02-경계-살피고-익히기)
3. [log4j 익히기](#03-log4j-익히기)
4. [학습 테스트는 공짜 이상이다](#04-학습-테스트는-공짜-이상이다)
5. [아직 존재하지 않는 코드를 사용하기](#05-아직-존재하지-않는-코드를-사용하기)
6. [깨끗한 경계](#06-깨끗한-경계)

## 들어가기 전에

- 시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다.
- 구입한 패키지, 오픈소스 또는 사내의 다른 팀에서 제공하는 컴포넌트와 같은 외부 코드를 우리 코드에 깔끔하게 통합해야만 한다.

## 01. 외부 코드 사용하기

- 패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애쓴다. 반면, 사용자는 자신의 요구에 집중하는 인터페이스를 바란다. 이런 긴장으로 인해 시스템 경계에서 문제가 생길 소지가 많다.

### 예시

```java
// 인터페이스
Map sensors = new HashMap();

// 클라이언트
Sensor s = (Sensor) sensors.get(sensorId);
```

위 코드의 문제점

- Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있다.
- 의도가 분명하게 드러나지 않는다.

### Generics 사용하여 개선하기

```java
// 인터페이스
Map<String, Sensor> sensors = new HashMap<>();

// 클라이언트
Sensor s = sensors.get(sensorId);
```

위 코드의 문제점

- `Map<String, Sensor>`가 사용자에게 필요하지 않은 기능까지 제공한다. <br> 👉 Map을 조작할 위험이 있다. 예) `clear()` 메서드 호출
- `Map<String, Sensor>` 인스턴스를 여기저기로 넘긴다면, 인터페이스가 변할 경우 수정할 코드가 상당히 많아진다.

### 캡슐화를 이용하여 개선하기

경계 인터페이스인 Map을 `Sensors` 안으로 숨긴다.

```java
public class Sensors {
    private Map sensors = new HashMap();

    public Sensor getById(String id) {
        return (Sensor) sensors.get(id);
    }

    // ...
}
```

위 코드의 장점

- Map 인터페이스가 변하더라도 나머지 프로그램에는 영향을 미치지 않는다. Sensors 클래스 안에서 객체 유형을 관리하고 변환하기 때문이다.
- 프로그램에 필요한 인터페이스만 제공한다. 그래서 코드는 이해하기 쉽지만 오용하기는 어렵다.

Map 인터페이스를 사용할 때 주의사항

- ⚠️ Map을 여기저기 넘기지 말라.
- ⚠️ Map과 같은 경계 인터페이스를 이용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의한다.
- ⚠️ Map 인스턴스를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않는다.

👉 List, Map 와 같은 Collection을 그대로 사용하지 말고, Wrapping하여 일급 컬렉션을 사용하자. (참고 사이트 : [일급 컬렉션의 소개와 써야할 이유](https://jojoldu.tistory.com/412))

## 02. 경계 살피고 익히기

- 외부 패키지 테스트는 우리 책임은 아니지만 우리 자신을 위해 우리가 사용할 코드를 테스트하는 편이 바람직하다.
- ✅ 학습 테스트를 하자.
  - ℹ️ 학습 테스트란? 곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성해 외부 코드를 작성하는 것
  - 프로그램에서 사용하려는 방식대로 외부 API를 호출하여, 통제된 환경에서 API를 제대로 이해하는지를 확인한다.

## 03. log4j 익히기

1. 학습 테스트를 통해 사용법을 익힌다.
2. 여기서 얻은 모든 지식을 독자적인 로거 클래스로 캡슐화한다. <br> → 나머지 프로그램은 log4j 경계 인터페이스를 몰라도 된다.

## 04. 학습 테스트는 공짜 이상이다

✅ 학습 테스트의 장점

1. 학습 테스트는 필요한 지식만 확보하는 손쉬운 방법이다.
2. 학습 테스트는 패키지가 예상대로 도는지 검증한다. (우리 코드와 호환이 되는지)
3. 실제 코드와 동일한 방식으로 인터페이스를 사용하는 테스트 케이스는 새 버전으로 이전하기 쉬워진게 한다.

## 05. 아직 존재하지 않는 코드를 사용하기

1. 협업을 하는 팀이 아직 API를 설계하지 않았을 경우, 자체적으로 인터페이스를 정의한다.
   - 우리가 바라는 인터페이스로 구현하기 때문에 우리가 인터페이스를 전적으로 통제할 수 있다.
   - 또한, 코드 가독성도 높아지고 코드 의도도 분명해진다.
2. Adapter 패턴으로 API 사용을 캡슐화해 API가 바뀔 때 수정할 코드를 한 곳으로 모은다.

<details>
<summary>ℹ️ Adapter 패턴이란?</summary>
<div markdown="1">

- 클래스의 인터페이스를 사용자가 기대하는 다른 인터페이스로 변환하는 패턴
- 호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들이 함께 작동하도록 해준다.
- 종류 
    1. Object Adapter 
        - 구성(Composition)을 이용하여 사용자가 기대하는 인터페이스를 구현한다. 
        - 즉, 어댑터 내부에 사용하려는 private API 인스턴스가 존재하고, 인스턴스의 메서드를 호출하여 내부 로직을 구현한다. 
    2. Class Adapter 
        - 다중 상속을 이용하여 사용자가 기대하는 인터페이스를 구현한다. 
        - API 인터페이스(구현체)를 상속받아 내부에서 API 호출이 가능하고, 이를 이용하여 내부 로직을 구현한다.
</div>
</details>

## 06. 깨끗한 경계

- ✅ 경계에 위치하는 코드는 깔끔히 분리한다.
- ✅ 기대치를 정의하는 테스트 케이스도 작성한다.
- ✅ 통제가 불가능한 외부 패키지에 의존하는 대신 통제가 가능한 우리 코드에 의존하도록 하자.
- ✅ 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하자.
  - 방법 (1) : 새로운 클래스로 경계를 감싸기
  - 방법 (2) : Adapter 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환하기
  - 장점
    1. 코드 가독성 향상
    2. 경계 인터페이스를 사용하는 일관성 향상
    3. 외부 패키지가 변했을 때 변경할 코드 감소
