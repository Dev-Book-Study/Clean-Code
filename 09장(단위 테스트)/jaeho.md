## 9장 단위 테스트  

---

### 1. TDD 법칙 세가지

> TDD (Test Driven Development, 테스트 주도 개발): 테스트 케이스를 작성 한 후 실제 코드를 개발하여 리팩토링하는 개발 방법론.
>
> ![스크린샷 2022-01-22 오후 9.31.49](/Users/user/Desktop/스크린샷 2022-01-22 오후 9.31.49.png)

- __첫째 법칙__ - 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
- __둘째 법칙__ - 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
- __셋째 법칙__ - 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

> 1. 실패하는 테스트를 작성하기 전에는 코드를 작성하지 않는다.
>    - You are not allowed write any prodection code unless it is to make a failing unit test pass
> 2. 실패하는 테스트 코드를 한번에 하나 이상 작성하지 않는다. (컴파일 에러도 실패다)
>    - You are not allowed to write any more of a unit test than is sufficient to fail (and compilation failures are failures.)
> 3. 현재 실패한 테스트 코드를 성공시키는데 충분한 정도로만 코드를 작성한다.
>    - You are not allowed to write any more prodection code than is sufficient to pass the one failing unit test

---

### 2. 깨끗한 테스트 코드 유지하기

- 지저분한 테스트 코드를 내놓는 것은 테스트를 안하는 것과 별반 다르지 않다.

- 테스트 코드가 지저분할 수록 변경하기 어려우며, 실제 코드를 짜는 것보다 테스트 케이스를 추가하는 시간이 걸리시 십상이다.

- _즉, 테스트 코드는 실제 코드 못지 않게 중요하다._

- __테스트는 유연성, 유지보수성, 재사용성을 제공한다.__

  - 단위 테스트(Unit Test)는 코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이다. 실제 코드를 유연하게 변경하고 만들 수 있기 때문이다.

  > 단위 테스트(Unit Test)?: 하나의 모듈을 기준으로 독립적으로 진행되는 가장 작은 단위의 테스트
  >
  > 통합 테스트(Integration Test)?: 모듈을 통합하는 과정에서 모듈 간의 호환성을 확인하기 위해 수행되는 테스트

---

### 3. 깨끗한 테스트 코드

- 깨끗한 테스트 코드를 만들 때, 가장 중요한 것은_가독성_ 이다.
- 테스트 코드는 진짜 필요한 자료 유형과 함수만 사용하자. => 세세한 코드에 집중할 필요 없이 코드가 수행하는 기능을 빠르게 이해할 수 있다.

> __BUILD-OPERATE-CHECK 패턴__  
>
> [참고 자료](https://medium.com/swlh/usual-production-patterns-applied-to-integration-tests-50a941f0b04a)
>
> - Build - 테스트 시나리오를 준비하는 단계이다. 이 단계에서는 일반적으로 데이터베이스에 일부 데이터를 넣는다.
> - Operate - 테스트 중인 Object/API 에서 메서드를 실행하는 단계
> - Check - 실행된 메서드가 시스템에 예상한 영향을 미쳤는지 확인하는 단계이다. 이 단계에서는 일반적으로 데이터베이스를 쿼리하고 메서드 실행 결과를 확인한다.
>
> ![image-20220123204032820](/Users/user/Library/Application Support/typora-user-images/image-20220123204032820.png)

__3-1. 도메인에 특화된 테스트 언어__

> [도메인 특화 언어 - JetBrain](https://www.jetbrains.com/ko-kr/mps/concepts/domain-specific-languages/)
>
> 도메인 특화 언어(Domain - Specific - Languages) :  특정 분야에 최적화된 프로그래밍 언어이다. 해당 분야 또는 도메인의 개념과 규칙을 사용한다. SQL이나 HTML 같은 언어가 있다. => 반대어로는 범용어인데, C / Java 와 같은 프로그래밍 언어를 말한다.
>
> 장점: 언어의 구조가 안정적이라면, 그 분야의 사람들이 코드를 이해하는 데 불편함이 없다, 또한 프로그래밍 코드의 양이 적고 가독성이 높다.
>
> 단점: 설계가 잘 이루어지지 않는다면, 읽기 어려운 코드가 될 수 있다.

- 도메인 특화 언어의 장점을 잘 이용하면 테스트 코드를 짜기도 읽기도 쉬워 진다.
- 즉, 테스트를 구현하는 당사자와 테스트를 읽어볼 독자를 도와주는 테스트 '언어'로 이용가능 하다.

__3-2. 이중표준__

- 테스트 코드에 적용하는 표준은 실제 코드에 적용하는 표준은 확실히 다르다. => 단순하고 간결하고 표현력이 풍부해야 하지만 실제 코드만큼 효율적일 필요는 없다.
- 컴퓨터의 자원과 메모리가 제한적인 가능성이 높지만, 테스트 환경은 자원이 제한적일 가능성이 낮다. 즉, 실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제 없는 방식이 있다.

---

### 4. 테스트 당 assert 하나

- `assert` 문이 단 하나인 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다.
- 때로 함수하나에 여러 `assert` 를 사용한다고 하더라도, 최대한 줄이려 하는 것이 좋은 생각이다.

> [__given-when-then 패턴__](https://brunch.co.kr/@springboot/292)
>
> [준비 - 실행 - 검증]
>
> ```java
> public void testGetPageHierarchyHasRightTags() throws Exception {
>   // given 테스트를 위해 준비하는 과정
>   givenPages("PageOne", "PageOne,ChildOne", "PageTwo");
>   
>   // when 액션을 하는 테스트를 실행하는 과정
> 	whenRequestIsIssued("root", "type:pages");
>   
>   // then 테스트를 검증하는 과정
>   thenResponseShouldContain(
>     "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
>   )
> }
> ```

> [Template Method 패턴](https://gmlwjd9405.github.io/2018/07/13/template-method-pattern.html)
>
> - 어떤 작업을 처리하는 일부분을 서브 클래스로 캡슐화해 전체 일을 수행하는 구조는 바꾸지 않으면서 특정 단계에서 수행하는 내역을 바꾸는 패턴
> - 즉, 전체적으로는 동일하면서 부분적으로는 다른 구문으로 구성된 메서드의 코드 중복을 최소하는 패턴

---

__5. F.I.R.S.T__

- 깨끗한 테스트는 다섯 가지 규칙을 따른다.
- __Fast 빠르게__ : 테스트는 빨라야 한다. 즉, 테스트는 빨리 돌아야하며 느리게 돈다면 자주 돌릴 엄두를 내지 못한다. ==> 자주 돌리지 않으면 초반에 문제를 찾아내지 못하고 고치지도 못한다.
- __Independent 독립적으로__ : 각 테스트는 서로 의존하면 안 된다. 즉, 한 테스트 때문에 다음 테스트를 돌릴 환경을 준비하면 안된다. => 테스트가 서로에게 의존하면 하나가 실패할 때, 잇달아 실패하므로 원인을 파악하기 어려워진다.
- __Repeatable 반복가능하게__ : 테스트는 어떤 환경에서도 반복 가능해야 한다.
- __Self-Validating 자가검증하는__ : 테스트는 부울(boolean)값으로 결과를 내야한다. 성공아니면 실패로 나타내어 주관적인 판단을 방지할 수 있도록 구성한다.
- __Timely 적시에__ : 테스트는 적시에 작성해야 한다. 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.

---

__6. 결론__

- 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화한다.
- 테스트 코드는 지속적으로 깨끗하게 관리하자.
- 테스트 API를 구현해 도메인 특화 언어를 만들자 => 그만큼 테스트 코드를 짜기가 쉬워진다.

