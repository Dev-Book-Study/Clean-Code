# 📖 11장. 시스템

## 🔎 요약

- 시스템 수준에서도 깨끗함을 유지하는 방법을 다루고 있다.
- 시스템을 준비하는 과정(객체 생성, 의존성 연결 등)과 준비 과정 이후에 이어지는 런타임 로직(도메인)을 분리해야 한다.
  - 방법
    - 생성과 관련된 코드는 main이나 main이 호출하는 모듈로 옮긴다.
    - 런타임 로직 도중에 객체를 생성해야 하는 경우 ABSTRACT FACTORY 패턴을 사용한다.
    - 의존성 주입(DI)
- 관심사를 분리하라.
  - 자바에서 사용하는 관점 혹은 관점과 유사한 메커니즘
    - 자바 프록시
    - 순수 자바 AOP 프레임워크
    - AspectJ

## 🔖 목차

- [01. 도시를 세운다면?](#01-도시를-세운다면)
- [02. 시스템 제작과 시스템 사용을 분리하라](#02-시스템-제작과-시스템-사용을-분리하라)
- [03. 확장](#03-확장)
- [04. 자바 프록시](#04-자바-프록시)
- [05. 순수 자바 AOP 프레임워크](#05-순수-자바-aop-프레임워크)
- [06. AspectJ 관점](#06-aspectj-관점)
- [07. 테스트 주도 시스템 아키텍처 구축](#07-테스트-주도-시스템-아키텍처-구축)
- [08. 의사 결정을 최적화하라](#08-의사-결정을-최적화하라)
- [09. 명백한 가치가 있을 때 표준을 현명하게 사용하라](#09-명백한-가치가-있을-때-표준을-현명하게-사용하라)
- [10. 시스템은 도메인 특화 언어가 필요하다](#10-시스템은-도메인-특화-언어가-필요하다)
- [11. 결론](#11-결론)

## 01. 도시를 세운다면?

- 도시가 잘 돌아가는 이유는
    1. 도시에는 큰 그림을 그리는 사람들도 있으며 작은 사항에 집중하는 사람들도 있다.
    2. 적절한 추상화와 모듈화 때문이다. 그래서 큰 그림을 이해하지 못할지라도 개인과 개인이 관리하는 '구성요소'는 효율적으로 돌아간다.
- 소프트웨어 팀도 도시처럼 구상한다. :white_check_mark: 깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다.
- 이 장에서는 높은 추상화 수준, 즉 시스템 수준에서도 깨끗함을 유지하는 방법을 살펴본다. :point_right: 서론

## 02. 시스템 제작과 시스템 사용을 분리하라

- 제작(construction)은 사용(use)와 아주 다르다.
- :white_check_mark: 소프트웨어 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 '연결'하는) 준비 과정과 (준비 과정 이후에 이어지는) 런타임 로직을 분리해야 한다.
- 시작 단계는 모든 애플리케이션이 풀어야 할 관심사(concern)다. **관심사 분리**는 우리 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나다.

#### 나쁜 예

```java
// 초기화 지연 혹은 계산 지연 기법
public Service getService() {
    if (service == null) {
        service = new MyServiceImpl(...);
    }
    return service;
}
```

초기화 지연(Lazy Initialization) 혹은 계산 지연(Lazy Evaluation) 기법

- 장점
  - 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않는다. 따라서 애플리케이션이 시작하는 시간이 그만큼 빨라진다.
  - 어떤 경우에도 `null`을 반환하지 않는다.
- 문제점

  - `getService()` 메서드가 `MyServiceImpl`과 생성자 인수에 명시적으로 의존한다. 런타임 로직에서 해당 객체를 전혀 사용하지 않더라도 의존성을 해결하지 않으면 컴파일이 안 된다.
  - `MyServiceImpl`이 무거운 객체라면 적절한 테스트 전용 객체(TEST DOUBLE 이나 MOCK OBJECT)를 service 필드에 할당해야 한다. 또한, 일반 런타임 로직에다 객체 생성 로직을 섞어 놓은 탓에 service가 `null`인 경로와 `null`이 아닌 경로 등 모든 실행 경로도 테스트해야 한다.
  - 무엇보다 `MyServiceImpl`이 모든 상황에 적합한 객체인지 모른다.

    > 모든 상황에 적합한 객체라면 해당 메서드의 클래스가 전체 문맥을 알아야 한다는 것을 의미한다. 이는 불가능하다. 또한 모든 문맥에 적합할 가능성도 적다.

### (1) Main

- :white_check_mark: 생성과 관련된 코드는 모두 main이나 main이 호출하는 모듈로 옮기고, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.
- main 함수에서 시스템에 필요한 객체를 생성한 후 이를 애플리케이션에 넘긴다. 애플리케이션은 그저 객체를 사용할 뿐이다.
- 애플리케이션은 main이나 객체가 생성되는 과정을 전혀 모른다. 단지 모든 객체가 적절히 생성되었다고 가정한다.

### (2) 팩토리

- :white_check_mark: 물론 때로는 객체가 생성되는 시점을 애플리케이션이 결정할 필요도 생긴다. 이때는 ABSTRACT FACTORY 패턴을 사용한다.

### (3) 의존성 주입

- 사용과 제작을 분리하는 강력한 메커니즘 하나가 **의존성 주입(DI)** 이다. 의존성 주입은 제어 역전(IoC) 기법을 의존성 관리에 적용한 메커니즘이다.
- 제어 역전에서는 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다. 새로운 객체는 넘겨받은 책임만 맡으므로 SRP를 지키게 된다.
- 의존성 관리 맥락에서 객체는 의존성 자체를 인스턴스로 만드는 책임을 지지 않는다. 대신에 이런 책임을 다른 '전담' 메커니즘에 넘겨야 한다. 그렇게 함으로써 제어를 역전한다. 대개 <u>'책임질' 매커니즘으로 'main' 루틴이나 특수 컨테이너를 사용한다.</u>

#### 예시

JNDI 검색

```java
MyService myService = (MyService)(jndiContext.lookup("NameOfMyService"));
```

- 호출하는 객체는 실제로 반환되는 객체의 유형을 제어하지 않는다. 대신 호출하는 객체는 의존성을 능동적으로 해결한다.

> - "_호출하는 객체_ " -> MyService 라는 인터페이스 객체를 사용하는 객체
> - "_객체의 유형을 제어하지 않는다._ " -> 객체의 유형은 객체를 생성한 곳에서 설정한대로 사용하다.
> - "_의존성을 능동적으로 해결한다._ " -> 이름을 통해서 생성한 객체를 주입받아 해결한다.

진정한 의존성 주입이란?

- 클래스가 의존성을 해결하려 시도하지 않는다. 클래스는 완전히 수동적이다. 대신에 의존성을 주입하는 방법으로 setter 메서드나 생성자 인수를 제공한다.
- DI 컨테이너는 (대개 요청이 들어올 때마다) 필요한 객체의 인스턴스를 만든 후 생성자 인수나 setter 메서드를 사용해 의존성을 설정한다. 실제로 생성되는 객체 유형은 설정 파일에서 지정하거나 특수 생성 모듈에서 코드를 명시한다. 예) 스프링 프레임워크

초기화 지연

- 대다수 DI 컨테이너는 필요할 때까지는 객체를 생성하지 않고, 대부분은 계산 지연이나 비슷한 최적화에 쓸 수 있도록 팩토리를 호출하거나 프록시를 생성하는 방법을 제공한다.
- 즉, 계산 지연 기법이나 이와 유사한 최적화 기법에서 이런 메커니즘을 사용할 수 있다.

## 03. 확장

- 우리는 오늘 주어진 사용자 스토리에 맞춰 시스템을 구현해야 한다. 내일은 새로운 스토리에 맞춰 시스템을 조정하고 확장하면 된다. (애자일 방식)
- TDD, 리팩터링, 그리고 TDD와 리팩터링으로 얻어지는 깨끗한 코드는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다. :point_right: 깨끗한 코드 & 아키텍처의 중요성
- 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다. :point_right: 관심사를 분리하는 것이 핵심이다!

### 횡단(cross-cutting) 관심사

- 현실적으로는 영속성 방식을 구현한 코드는 온갖 객체로 흩어진다. :point_right: 영속성 프레임워크 영역과 도메인 영역이 세밀한 단위로 겹친다.
- **AOP(관점지향 프로그래밍)** 는 횡단 관심사에 대처해 모듈성을 확보하는 일반적인 방법론이다.
- AOP에서 관점이라는 모듈 구성 개념은 "특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야 한다."라고 명시한다. 명시는 간결한 선언이나 프로그램밍 메커니즘으로 수행한다.

## 04. 자바 프록시

예시

```java
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

public class BankImpl implements Bank {
    private List<Account> accounts;

    Collection<Account> getAccounts() {
        return accounts;
    }

    void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}

public class BankProxyHandler implements InvocationHandler {
    @Overrice
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 데이터베이스에서 계좌 정보를 읽어와서 Bank 메서드를 구현한다.
    }
}
```

```java
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl())
);
```

단점

- 코드 '양'과 크기는 프록시의 두 가지 단점이다. 즉, 프록시를 사용하면 깨끗한 코드를 작성하기 어렵다.
- 또한, 시스템 단위로 실행 '지점'을 명시하는 메커니즘도 제공하지 않는다.

## 05. 순수 자바 AOP 프레임워크

- 순수 자바 관점을 구현하는 스프링 AOP, JBoss AOP 등과 같은 여러 자바 프레임워크는 내부적으로 프록시를 사용한다.
- 스프링은 비즈니스 논리를 POJO로 구현한다.
  - POJO는 순수하게 도메인에 초점을 맞춘다. POJO는 엔터프라이즈 프레임워크에 (그리고 다른 도메인에도) 의존하지 않는다. 따라서 테스트가 개념적으로 더 쉽고 간단하다. (<-> EJB)
  - 상대적으로 단순하기 때문에 사용자 스토리를 올바로 구현하기 쉬우며 미래 스토리에 맞춰 코드를 보수하고 개선하기 편하다.
- 프로그래머는 설정 파일이나 API를 사용해 필수적인 애플리케이션 기반 구조를 구현한다. 영속성, 트랜잭션, 보안, 캐시, 장애조치 등과 같은 횡단 관심사도 포함된다.
- 실제로 스프링이나 JBoss 라이브러리의 관점을 명시한다. 이때 프레임워크는 사용자가 모르게 프록시나 바이트코드 라이브러리를 사용해 이를 구현한다. 이런 선언들이 요청에 따라 주요 객체를 생성하고 서로 연결하는 등 DI 컨테이너의 구체적인 동작을 제어한다.
- 위의 예시에서 클라이언트는 `Bank` 객체에서 `getAccounts()`를 호출한다고 믿지만 실제로는 `Bank` POJO의 기본 동작을 확장한 중첩 DECORATOR 객체 집합의 가장 외곽과 통신한다.

```xml
<!-- app.xml -->
<beans>
    <!-- ... -->
    <bean
        id="appDataSource"
        class="org.apache.commons.dbcp.BasicDataSource"
        destory-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="me"
    />
    <bean
        id="bankDataAccessObject"
        class="com.example.banking.persistence.BankDataAccessObject"
        p:dataSource-ref="appDataSource"
     />
    <bean
        id="bank"
        class="com.example.banking.model.Bank"
        p:dataAccessObject-ref="bankDataAccessObject"
    />
</beans>
```

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

- 스프링 관련 자바 코드가 거의 필요 없으므로 애플리케이션은 사실상 스프링과 독립적이다. :point_right: 코드를 테스트하고 개선하고 보수하기 용이해진다.

## 06. AspectJ 관점

- 관심사를 관점으로 분리하는 가장 강력한 도구는 AspectJ 언어이다. AspectJ는 언어 차원에서 관점을 모듈화 구성으로 지원하는 자바 언어 확장이다.
- AspectJ는 관점을 분리하는 강력하고 풍부한 도구 집합을 제공하지만, 새 도구를 사용하고 새 언어 문법과 사용법을 익혀야 한다는 단점이 있다.
- AspectJ '애너테이션 폼'은 새로운 도구와 새로운 언어라는 부담을 어느 정도 완화한다. '애너테이션 폼'은 순수한 자바 코드에 자바 5 애너테이션을 사용해 관점을 정의한다. 또한, 스프링 프레임워크는 애너테이션 기반 관점을 쉽게 사용하도록 다양한 기능을 제공한다.

## 07. 테스트 주도 시스템 아키텍처 구축

- 애플리케이션 도메인 논리를 POJO로 작성할 수 있다면, 즉 코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능해진다. 그때그때 새로운 기술을 채택해 단순한 아키텍처를 복잡한 아키텍처로 키워갈 수도 있다.
- BDUF(Big Design Up Front)를 추구할 필요가 없다.
  - BDUF는 구현을 시작하기 전에 앞으로 벌어질 모든 사항을 설계하는 기법
  - 처음에 쏟아 부은 노력을 버리지 않으려는 심리적 저항으로 인해, 그리고 처음 선택한 아키텍처가 향후 사고 방식에 미치는 영향으로 인해, 변경을 쉽사리 수용하지 못하기 때문에 해로울 수 있다.
  - 소프트웨어 구조가 관점을 효과적으로 분리한다면, 극적인 변화가 경제적으로 가능하다.
- '아주 단순하면서도' 멋지게 분리된 아키텍처로 결과물을 빠르게 출시한 후, 기반 구조를 추가하며 조금씩 확장해 나가도 괜찮다. 세계 최대 웹 사이트들은 고도의 자료 캐싱, 보안, 가상화 등을 이용해 아주 높은 가용성과 성능을 효율적이고도 유연하게 달성했다. 설계가 최대한 분리되어 각 추상화 수준과 범위에서 코드가 적당히 단순하기 때문이다.
- :white_check_mark: 프로젝트를 시작할 때는 일반적인 범위, 목표, 일정은 물론이고 결과로 내놓을 시스템의 일반적인 구조도 생각해야 한다. 하지만 변하는 환경에 대처해 진로를 변경할 능력도 반드시 유지해야 한다.
- 좋은 API는 걸리적거리지 않아야 한다. 그래야 팀이 창의적인 노력을 사용자 스토리에 집중한다. 그리하지 않으면 아키텍처에 발이 묶여 고객에게 최적의 가치를 효율적으로 제공하지 못한다.

### 요약

- 최선의 시스템 구조는 각기 POJO (또는 다른) 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다.
- 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다.
- 테스트 주도 기법을 적용할 수 있다.

## 08. 의사 결정을 최적화하라

- 모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.
- 가장적합한 사람에게 책임을 맡기면 가장 좋다. 최대한 정보를 모아 최선의 결정을 내리기 위해서다.

### 요약

- 관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다.
- 이런 기민함 덕택에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다.
- 또한 결정의 복잡성도 줄어든다.

## 09. 명백한 가치가 있을 때 표준을 현명하게 사용하라

### 요약

- 표준을 사용했을 때의 장점
  - 아이디어와 컴포넌트를 재사용하기 쉽고,
  - 적절한 경험을 가진 사람을 구하기 쉬우며,
  - 좋은 아이디어를 캡슐화하기 쉽고,
  - 컴포넌트를 엮기 쉽다.
- 표준을 사용했을 때의 단점
  - 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다.

## 10. 시스템은 도메인 특화 언어(DSL)가 필요하다

- DSL(Domain-Specific Language)은 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다.
- DSL로 짠 코드는 도메인 전문가가 작성한 구조적인 산문처럼 읽힌다.
- 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 의사소통 간극을 줄여준다. 도메인 전문가가 사용하는 언어로 도메인 논리를 구현하면 도메인을 잘못 구현할 가능성이 줄어든다.
- 효과적으로 사용한다면 추상화 수준을 코드 관용구나 디자인 패턴 이상으로 끌어올린다.

### 요약

- DSL을 사용하면 고차원 정책에서 저차원 세부사항에 이르기까지 모든 추상화 수준과 모든 도메인을 POJO로 표현할 수 있다.

## 11. 결론

- 깨끗하지 못한 아키텍처는
  - 도메인 논리를 흐리며 기민성을 떨어뜨린다. 도메인 논리가 흐려지면 제품 품질이 떨어진다.
  - 버그가 숨어들기 쉬워지고, 스토리를 구현하기 어려워진다.
  - 기민성이 떨어지면 생산성이 낮아져 TDD가 제공하는 장점이 사라진다.
- 모든 추상화 단계에서 의도는 명확히 표현해야 한다. 그러려면 POJO를 작성하고 관점 혹은 관점과 유사한 메커니즘을 사용해 각 구현 관심사를 분리해야 한다.
- 시스템을 설계하든 개별 모듈을 설계하든, 실제로 돌아가는 가장 단순한 수단을 사용해야 한다.
