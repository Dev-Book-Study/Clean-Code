# 📖 부록A. 동시성 II

## 🔖 목차

1. [클라이언트/서버 예제](#01-클라이언트서버-예제)
2. [가능한 실행 경로](#02-가능한-실행-경로)
3. [라이브러리를 이해하라](#03-라이브러리를-이해하라)
4. [메서드 사이에 존재하는 의존성을 조심하라](#04-메서드-사이에-존재하는-의존성을-조심하라)
5. [작업 처리량 높이기](#05-작업-처리량-높이기)
6. [데드락](#06-데드락)
7. [다중 스레드 코드 테스트](#07-다중-스레드-코드-테스트)
8. [스레드 코드 테스트를 도와주는 도구](#08-스레드-코드-테스트를-도와주는-도구)

## 들어가기 전에

이 장에서는 동시성 프로그래밍이라는 크고 험난한 영역을 간략하게 소개한다.

- 다중 스레드 코드를 깨끗하게 유지하는 방법
- 동시 갱신과 동시 갱신을 방지하는 깨끗한 동기화/잠금 기법
- 스레드가 I/O 위주 시스템의 처리율을 높여주는 이유와 실제로 처리율을 높이는 방법
- 데드락과 깔끔하게 데드락을 방지하는 기법
- 보조 코드를 추가해 동시성 문제를 사전에 노출하는 전략

추천 도서
- Concurrent Programming in Java : Design Principles and Patterns, Doug Lea

## 01. 클라이언트/서버 예제

서버는 소캣을 열어놓고 클라이언트가 연결하기를 기다린다. 클라이언트는 소캣에 연결해 요청을 보낸다.

### (1) 서버

서버

```java
ServerSocket serverSocker = new ServerSocket(8009);

while (keepProcessing) {
    try {
        Socket socket = serverSocket.accept();
        process(socket);
    } catch (Exception e) {
        handle(e);
    }
}
```

클라이언트

```java
private void connectSendReceive(int i) {
    try {
        Sockey socket = new Socket("localhost", PORT);
        MessageUtils.sendMessage(socket, Integer.toString());
        MessageUtils.getMessage(socket);
        socket.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

테스트

```java
// 10,000밀리초(10초) 내에 끝나는지를 검사한다.
@Test(timeout = 10000)
public void shouldRunInUnder10Seconds() throws Exception {
    Thread[] threads = createThreads();
    startAllTreads(threads);
    waitForAllThreadsToFinish(threads);
}
```

위의 테스트 케이스는 시스템 작업 처리량을 검증하는 전형적인 예이다.

만약 테스트가 실패한다면? 먼저 애플리케이션이 어디서 시간을 보내는지 알아야 한다. 가능성은 두 가지이다. 대개 시스템은 둘 다 하느라 시간을 보내지만, 특정 연산을 살펴보면 대개 하나가 지배적이다.

- I/O
  - 소켓 사용, 데이터베이스 연결, 가상 메모리 스와핑 기다리기 등에 시간을 보낸다.
  - 동시성이 성능을 높여주기도 한다. 시스템 한쪽이 I/O를 기다리는 동안에 다른 쪽이 뭔가를 처리해 노는 CPU를 효과적으로 활용할 수 있다.
- 프로세서
  - 수치 계산, 정규표현식 처리, 가비지 컬렉션 등에 시간을 보낸다.
  - 새로운 하드웨어를 추가해 성능을 높일 수 있다. 프로세서 연산에 시간을 보내는 프로그램은 스레드를 늘인다고 빨라지지 않는다. CPU 사이클은 한계가 있기 때문이다.

### (2) 스레드 추가하기

서버의 `process` 함수가 주로 I/O 연산에 시간을 보낸다면 아래와 같이 스레드를 추가하여 자료 처리량을 높일 수 있다.

```java
void process(final Socket socket) {
    if (socket == null) {
        return;
    }

    Runnable clientHandler = new Runnable() {
        public void run() {
            try {
                String message = MessageUtils.getMessage(socket);
                MessageUtils.sendMessage(socket, "Processed: " + message);
                closeIngoringException(socket);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    };
    Thread clientConnection = new Thread(clientHandler);
    clientConnection.start();
}
```

### (3) 서버 살펴보기

위의 코드에서 서버 코드가 지는 책임이 많다.

- 소캣 연결 관리
- 클라이언트 처리
- 스레드 정책
- 서버 종료 정책

**스레드를 관리하는 코드는 스레드만 관리해야 한다.**

- 비동시성 문제까지 뒤섞이지 않더라도 동시성 문제는 그 자체만으로도 추적하기 어려운 탓이다.

**스레드 관리 책임을 클래스로 분리해야 한다.**

- <u>스레드 관리 전략이 변할 때 전체 코드에 미치는 영향이 작아지며 다른 책임을 간섭하지 않는다.</u>
- 또한, 스레드 걱정하지 않고 <u>다른 책임을 테스트하기가 훨씬 더 쉬워진다.</u>

서버 (각 책임을 클래스로 분할한 버전)

```java
public void run() {
    while (keepProcessing) {
        try {
            ClientConnection clientConnection = connectionManager.awaitClient();
            ClientRequestProcessor requestProcessor = new ClientRequestProcessor(clientConnection);
            clientScheduler.schedule(requestProcessor);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    connectionManager.shutdown();
}
```

```java
// 동시성 문제가 발생할 수 있는 곳
public interface ClientScheduler {

    void schedule(ClientRequestProcessor requestProcessor);

}

// 동시성 정책을 구현한다.
public class ThreadPerRequestScheduler implements ClientScheduler {

    public void schedule(final ClientRequestProcessor requestProcessor) {
        Runnable runnable = new Runnable() {
            public void run() {
                requestProcessor.process();
            }
        }
        Thread thread = new Thread(runnable);
        thread.start();
    }

}
```

### (4) 결론

동시성은 그 자체가 복잡한 문제이기 때문에 **다중 스레드 프로그램에서는 단일 책임 원칙이 특히 중요하다.**

## 02. 가능한 실행 경로

```java
public class IdGenerator {

    int lastIdUsed;

    public int incrementValue() {
        return ++lastIdUsed;
    }

}
```

만약 `IdGenerator` 인스턴스는 그대로지만 스레드가 2개일 때, 각 스레드가 `incrementValue()` 메서드를 한 번씩 호출한다면 가능한 결과는 아래와 같다.

- 스레드1이 94를 얻고, 스레드2가 95를 얻고, lastIdUsed가 95가 된다.
- 스레드1이 94를 얻고, 스레드2가 94를 얻고, lastIdUsed가 95가 된다.
- **스레드1이 94를 얻고, 스레드2가 94를 얻고, lastIdUsed가 94가 된다.**

### (1) 경로 수

`return ++lastIdUsed;` 라는 자바 코드 한 줄은 바이트 코드 명령 8개에 해당한다. 두 스레드가 명령8개를 뒤섞어 실행할 가능성이 충분하다.

루프나 분기가 없는 명령 N개를 스레드 T개가 차례로 실행한다면 가능한 경우의 수 = $\frac{(NT)!}{N!^T}$

따라서, 위의 자바 코드 한 줄이 실행 가능한 경로 수는 12,870개다. ($T=2$, $N=8$)

만약 아래와 같이 `synchronized` 키워드를 사용한다면 가능한 경로 수는 2개로 줄어든다. (스레드가 N개일 때, 가능한 경로 수 = $N!$)

```java
public synchronized void incrementValue() {
    ++lastIdUsed;
}
```

### (2) 심층 분석

**원자적 연산** (atmoic operation) : 중단이 불가능한 연산

JVM 명세에 따르면 64비트 값을 할당하는 연산은 32비트 값을 할당하는 연산 두 개로 나눠진다. 즉, 첫 32비트 값을 할당한 직후에 그리고 둘째 32비트 값을 할당하기 직전에 다른 스레드가 끼어들어 두 32비트 값 중 하나를 변경할 수 있다는 의미다.

전처리 증가 연산자 `++` 는 중단이 가능하다. 따라서 원자적 연산이 아니다.

### (3) 결론

**어떤 연산이 안전하고 안전하지 못한지 파악할 만큼 메모리 모델을 이해하고 있어야 한다.**

- 공유 객체/값이 있는 곳
- 동시 읽기/수정 문제를 일으킬 소지가 있는 코드
- 동시성 문제를 방지하는 방법

## 03. 라이브러리를 이해하라

### (1) Executor 프레임워크

- 스레드 풀링으로 정교한 실행을 지원한다.
- `Executor`는 `java.util.concurrent` 패키지에 속하는 클래스
- 애플리케이션에서 스레드는 생성하나 스레드 풀을 사용하지 않는다면 혹은 직접 생성한 스레드 풀을 사용한다면 `Executor` 클래스를 사용하면, 코드가 깔끔해지고, 이해하기 쉬워지고, 크기가 작아진다.

#### 기능

- 스레드 풀을 관리하고, 풀 크기를 자동으로 조정하며, 필요하다면 스레드를 재사용한다.
- 다중 스레드 프로그래밍에서 많이 사용하는 `Future`도 지원한다. `Future`는 독립적인 연산 여럿을 실행한 후 모두가 끝나기를 기다릴 때 유용하다.
- `Runnable` 인터페이스를 구현한 클래스는 물론 `Callable` 인터페이스를 구현한 클래스도 지원한다. `Callable` 인터페이스는 `Runnable` 인터페이스와 유사하지만 결과 값을 반환한다.

### (2) 스레드를 차단하지 않는 방법 (Non Blocking)

`AtomicBoolean`, `AtomicInteger`, `AtomicReference` 등

- 기본 유형이 아니라 객체를 사용하고, `++` 연산자가 아니라 `incrementAndGet()` 메서드를 사용한다.
- `synchronized` 를 사용하는 것보다 거의 항상 더 빠르다.

성능이 좋은 이유

- `synchronized` 키워드는 언제나 락을 건다. 그러나 락을 거는 대가는 비싸다.
- 여러 스레드가 같은 값을 수정해 문제를 일으키는 상황은 그리 잦지 않다.
- 그래서 위의 문제가 발생했을 때 효율적으로 감지해 갱신이 성공할 때까지 재차 시도한다. 이 방법이 거의 항상 더 효율적이다.

동작 방식

1. 메서드가 공유변수를 갱신하려 할 때, CAS 연산은 현재 변수 값이 최종으로 알려진 값인지 확인한다. (CAS 연산은 원자적이다.)
2. 그렇다면 변수 값을 갱신한다.
3. 아니라면 다른 스레드가 끼어들었다는 뜻이므로 변수 값을 갱신하지 않는다.
4. CAS 연산으로 값을 변경하려던 메서드는 값이 변경되지 않았다는 사실을 확인하고 다시 시도한다.

<details>
<summary>CAS 란?</summary>

- CAS(Compare And Swap)은 동시성 처리를 위한 방법 중 하나로서 말 그대로 '비교한 후 바꿔주는 것'이다.
- 멀티 스레드, 멀티 코어 환경에서 각 변수는 스레드 내의 스택(캐시)에 저장된다. 이 변수에 대해서 스레드에 저장된 값과 메인 메모리에 저장된 값을 비교하여 값이 같다면 새로운 값으로 치환해준다.
- 값이 다르다면 계속 재시도 한다.
- 참고 : <https://chickenpaella.tistory.com/97>

</details>
### (3) 다중 스레드 환경에서 안전하지 않은 클래스

본질적으로 다중 스레드 환경에서 안전하지 않은 클래스

- `SimpleDateFormat`
- 데이터베이스 연결
- java.util 컨테이너 클래스
- 서블릿

각각의 메서드는 스레드에 안전할지라도 사이에 다른 스레드가 끼어들어 문제가 발생할 수 있다.

#### 예시

```java
if(!hashTable.containsKey(someKey)) {
    hasTable.put(someKey, new SomeValue());
}
```

위 코드의 동시성 문제

- 위의 코드에서 `containsKey()`와 `put()`는 스레드에 안전하다. 하지만 사이에 다른 스레드가 끼어들어 HashTable에 값을 추가할 수도 있다.

해결방안

- HashTable을 잠근다. HashTable을 사용하는 클라이언트 모두가 클라이언트-기반 잠금 메커니즘을 구현한다.

  ```java
  synchronized(map) {
      if (!map.containsKey(key)) {
          map.put(key, value);
      }
  }
  ```

- HashTable을 객체로 감싼 후 다른 API를 사용한다. Adapter 패턴을 사용해 서버-기반 잠금 메커니즘을 구현한다.

  ```java
  public class WrappedHashTable<K, V> {

      private Map<K, V> map = new HashTable<K, V>();

      public synchronized void putIfAbsent(K key, V value) {
          if (map.containsKey(key)) {
              map.put(key, value);
          }
      }
  }
  ```

- 스레드에 안전한 집합 클래스를 사용한다.
  ```java
  ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<Integer, String>();
  map.putIfAbsent(key, value); // 스레드에 안전하다.
  ```

## 04. 메서드 사이에 존재하는 의존성을 조심하라
서버
```java
public class IntegerIterator Implements Iterator<Integer> {

    private Integer nextValue = 0;

    public synchronized boolean hasNext() {
        return nextValue < 100000;
    }

    public synchronized Integer next() {
        if (nextValue = 100000) {
            throw new IteratorPastEndException();
        }
        return nextValue++;
    }

    public synchronized Integer getNextValue() {
        return nextValue;
    }
}
```

클라이언트
```java
IntegerIterator iterator = new IntegerIterator();
while (iterator.hasNext()) { // (1)
    int nextValue = iterator.next(); // (2)
    // ...
}
```

위 코드의 동시성 문제

- (1)과 (2)에서 두 스레드가 `IntegerIterator` 인스턴스 하나를 공유한다면, 맨 끝에 두 스레드가 서로를 간섭해 한 스레드가 끝을 지나치는 바람에 예외가 발생할 가능성이 작게나마 존재한다.
- 즉, 개별 메서드는 동기화되었지만 클라이언트는 메서드 두 개를 사용한다. 👉 문제상황
- 정수 목록 끝에서 스레드가 서로 간섭할 때만 문제가 발생한다.

해결방안

- 실패를 용인한다.
- 클라이언트-기반 잠금 메커니즘을 구현한다. (클라이언트를 수정한다.)
- 서버-기반 잠금 메커니즘을 구현한다. (서버를 수정한다.)

### (1) 실패를 용인한다.

때로는 실패해도 괜찮도록 프로그램을 조정할 수 있다. <br> 예) 클라이언트가 예외를 받아 처리한다.

### (2) 클라이언트-기반 잠금

```java
IntegerIterator iterator = new IntegerIterator();

while (true) {
    int nextValue;
    synchronized (iterator) {
        if (!iterator.hasNext()) {
            break;
        }
        nextValue = iterator.next();
    }
    doSomethingWith(nextValue);
}
```

각 클라이언트는 `synchronizd` 키워드를 이용해 `IntegerIterator` 객체에 락을 건다.

- 비록 DRY 원칙을 위반하지만, 다중 스레드 환경에 안전하지 못한 외부 도구를 사용하는 경우라면 필요할지도 모른다.
- ⚠️ 서버를 사용하는 모든 프로그래머가 락을 기억해 객체에 걸었다 풀어야 하므로 다소 위험한 전략이다.

### (3) 서버-기반 잠금
서버
```java
public class IntegerIteratorServerLocked {

    private Integer nextValue = 0;

    public synchronized Integer getNextOrNull() {
        if (nextValue < 100000) {
            return nextValue++;
        } else {
            return null;
        }
    }

}
```

클라이언트
```java
while (true) {
    Integer nextValue = iterator.getNextOrNull();
    if (next == null) {
        break;
    }
    // ...
}
```

👍 일반적으로 서버-기반 잠금이 더 바람직하다.

- 코드 중복이 줄어든다. (클라이언트)
- 성능이 좋아진다.
  > 단일 스레드 환경으로 교체하기 쉽다.
- 오류가 발생할 가능성이 줄어든다.
  > 클라이언트에서 잠금을 잊어버리는 바람에 오류가 발생할 위험이 서버 프로그래머 한 명으로 제한된다.
- 스레드 정책이 하나다. (서버)
- 공유 변수 범위가 줄어든다. 문제가 생길 경우 사펴볼 곳이 적다.

만약 서버 코드에 손대지 못한다면?

- Adapter 패턴을 사용해 API를 변경한 후 잠금을 추가한다.

  ```java
  // 클라이언트
  public class ThreadSafeIntegerIterator {

      private IntegerIterator iterator = new IntegerIterator();

      public synchronized Integer getNextOrNull() {
          if (iterator.hasNext()) {
              return iterator.next();
          }
          return null;
      }

  }
  ```

- 스레드에 안전하며 인터페이스가 확장된 집합 클래스를 사용한다.

## 05. 작업 처리량 높이기

**동기화 영역은 언제나 작을수록 좋다.**

처리 순서에 무관하게 페이지를 읽어와 독립적으로 처리해도 괜찮다면 다중 스레드가 처리율을 높여줄지도 모른다.

## 06. 데드락

### (1) 데드락이란?

**데드락(교착상태)** 이란? 두 개 이상의 작업이 서로 상대방의 작업이 끝나기 만을 기다리고 있기 때문에 결과적으로 아무것도 완료되지 못하는 상태

데드락은 다음 4가지 조건을 모두 만족하면 데드락이 발생한다.

1. 상호 배제
   - 여러 스레드가 한 자원을 공유하나 그 자원은 (1) 여러 스레드가 동시에 사용자히 못하며 (2) 개수가 제한적이라면 상호 배제 조건을 만족한다.
   - 예) 데이터베이스 연결, 쓰기용 파일 열기, 레코드 락, 세마포어 등과 같은 자원
2. 잠금 & 대기
   - 일단 스레드가 자원을 점유하면 필요한 나머지 자원까지 모두 점유해 작업을 마칠 때까지 이미 점유한 자원을 내놓지 않는다.
3. 선점 불가
   - 한 스레드가 다른 스레드로부터 자원을 빼앗지 못한다.
   - 자원을 점유한 스레드가 스스로 내놓지 않는 이상 다른 스레드는 그 자원을 점유하지 못한다.
4. 순환 대기
   - 각 프로세스는 순환적으로 다음 프로세스가 요구하는 자원을 가지고 있다.

### (2) 해결방안

위의 4가지 조건 중 하나라도 깨버리면 데드락은 발생하지 않는다.

1. 상호 배제 조건 깨기
   - 동시에 사용해도 괜찮은 자원을 사용한다. 예) `AtomicInteger`
   - 스레드 수 이상으로 자원 수를 늘린다.
   - 자원을 점유하기 전에 필요한 자원이 모두 있는지 확인한다.
   - 한계
     - 대다수 자원은 그 수가 제한적인 데다 동시에 사용하기도 어렵다.
     - 게다가 첫 번째 자원을 사용하고 나서야 두 번째로 필요한 자원이 밝혀지는 경우도 없지 않다.
2. 잠금 & 대기 조건 깨기
   - 각 자원을 점유하기 전에 확인한다. 만약 어느 하나라도 점유하지 못한다면 지금까지 점유한 자원을 몽땅 내놓고 처음부터 다시 시작한다.
   - 문제점 : 작업 처리량을 크게 떨어뜨린다.
     - 기아 → CPU 효율을 저하시킨다.
       - 한 스레드가 계속해서 필요한 자원을 점유하지 못한다.
       - 점유하려는 자원이 한꺼번에 확보하기 어려운 조합일 수도 있다.
     - 라이브락 → 쓸데 없이 CPU만 많이 사용한다.
       - 여러 스레드가 한꺼번에 잠금 단계로 진입하는 바람에 계속해서 자원을 점유했다 내놨다를 반복한다.
       - 단순한 CPU 스케줄링 알고리즘에서 특히 쉽게 발생한다.
3. 선점 불가 조건 깨기
   - 다른 스레드로부터 자원을 뺏어오는 방법
   - 일반적으로 간단한 요청 메커니즘으로 처리한다.
     1. 필요한 자원이 잠겼다면 자원을 소유한 스레드에게 풀어달라 요청한다.
     2. 소유 스레드가 다른 자원을 기다리던 중이었다면 자신이 소유한 자원을 모두 풀어주고 처음부터 다시 시작한다.
   - 장점
     - 스레드가 자원을 기다려도 괜찮다. → 처음부터 다시 시작하는 횟수가 줄어든다.
   - 단점
     - 모든 요청을 관리하기가 간단하지 않다.
4. 순환 대기 조건 깨기
   - 데드락을 방지하는 가장 흔한 전략
   - 자원을 똑같은 순서로 할당하게 만들면 순환 대기는 불가능해진다.
   - 모든 스레드가 일정 순서에 동의하고 그 순서로만 자원을 할당한다면 데드락은 불가능하다.
   - 문제점
     - 자원을 할당하는 순서와 자원을 사용하는 순서가 다를지도 모른다. → 자원을 꼭 필요한 이상으로 오랫동안 점유한다.
     - 때로는 순서에 따라 자원을 할당하기 어렵다. 첫 자원을 사용한 후에야 둘째 자원 ID를 얻는다면 순서대로 할당하기란 불가능하다.

**프로그램에서 스레드 관련 코드를 분리하면 조율과 실험이 가능하므로 통찰력이 높아져 최적의 전략을 찾기 쉬워진다.**

## 07. 다중 스레드 코드 테스트

다중 스레드 코드를 테스트하기 쉽지 않다.

- 테스트를 작성하더라도 문제가 <u>너무 드물게 발생하는 바람에 대개는 테스트로 발견하지 못한다.</u> 책의 저자는 간단한 동시성 문제를 발견하기 위해 일억 번을 테스트를 돌려야 안정적인 실패를 확인했다.
- 테스트를 조율해 한 장비에서 안정적인 실패를 얻었다 치더라도 <u>다른 장비, 다른 운영체제, 다른 JVM으로 옮기면 십중팔구 값을 다시 조율해야 한다.</u>

**테스트 코드를 작성 방법**

- 몬테 카를로 테스트
  1. 조율이 가능하게 유연한 테스트를 만든다.
  2. 임의로 값을 조율하면서 반복해 돌린다.
  3. 테스트가 실패하면 버그가 있다는 증거다. 테스트가 실패한 조건은 신중하게 기록한다.
  4. 테스트는 일찌감치 작성하기 시작해 통합 서버에서 계속 돌린다.
- 시스템을 배치할 플랫폼 전부에서 테스트를 돌린다. 반복해서 돌린다. 계속해서 돌린다. 실패 없이 오래 돌아갈수록 (1) 실제 코드가 올바르거나, (2) 테스트가 부족해 문제를 드러내지 못한 것을 의미한다.
- 부하가 변하는 장비에서 테스트를 돌린다. 실제 환경과 비슷하게 부하를 걸어줄 수 있다면 그렇게 한다.

하지만 위와 같은 조치를 모두 취한다고 하더라도 코드에서 스레드 문제를 찾아낼 가능성은 매우 낮다.

## 08. 스레드 코드 테스트를 도와주는 도구

IBM의 ConTest 도구

- 스레드에 안전하지 않는 코드에 보조 코드를 더해 실패할 가능성을 높여주는 도구
- 사용 방법
  1. 실제 코드와 테스트 코드를 작성한다. 다양한 부하 상황에서 여러 사용자를 시뮬레이션하는 테스트도 빼놓지 않는다.
  2. ConTest로 실제 코드와 테스트코드에 보조 코드를 추가한다.
  3. 테스트를 실행한다.
- 보조 코드를 추가한 클래스는 훨씬 더 빠르고 확실히 실패했다. 책의 저자는 기존의 천만 번에 한 번 정도 실패하던 코드가 서른 번에 한 번 정도 실패했다.

## References

- <https://ko.wikipedia.org/wiki/%EA%B5%90%EC%B0%A9_%EC%83%81%ED%83%9C>
