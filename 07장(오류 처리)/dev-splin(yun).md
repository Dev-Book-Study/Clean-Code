# 7장. 오류 처리

언제나 사용자는 우리의 예상과 다르게 행동하기 때문에 적절한 오류 처리는 매우 중요합니다. 다만, 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워진다면 깨끗한 코드를 부르기 어렵습니다. 따라서 이 장에서는 깨끗한 코드를 위해 오류를 처리하는 기법과 고려 사항에 대해 나와있습니다.





## 7-1. 오류 코드보다 예외를 사용하라

오류 코드를 통한 오류처리보다 예외를 사용하여 오류를 처리하는 것이 좋습니다.



### 오류 코드를 통한 오류 처리

```java
public class DeviceController {
    ...
        
 public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        // 디바이스 상태를 점검
        if (handle != DeviceHandle.INVALID) {
            // 레코드 필드에 디바이스 상태를 저장
            retrieveDeviceRecord(handle);
            // 디바이스가 일시정지 상태가 아니라면 종료
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
                logger.log("Device suspended. Unable to shut down");
            }
        } else {
            logger.log("Invalid handle for: " + DEV1.toString());
        }
    }
 ...
}
```

위와 같이 오류코드를 통해 오류 처리를 할 경우 **depth가 깊어지고 프로그램 로직이 오류처리 코드와 뒤섞이기 때문에 코드가 난잡**해질 수 있습니다. 때문에 아래와 같은 방법으로 예외를 던지는 것이 좋습니다.



### 예외를 통한 오류 처리

```java
public class DeviceController {
    ...
        
    public void sendShutDown() {
        try {
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }
    
    // 프로그램 로직을 추상화
    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
    }
    
    private DeviceHandle getHandle(DeviceID id) {
        ...
        throw new DeviceShutDownError("Invalid handle for: " + id.toString());
        ...
    }
    
    ...
}
```

위와 같이 `tryToShutDown()` 함수를 만들어 프로그램 로직을 **추상화시켜줌으로써, 오류 처리와 프로그램 로직을 분류하고 가독성을 개선**하였습니다.





## 7-2. Try-Catch-Finally 문부터 작성하라

try 블록에서 무슨 일이 생기든 catch 블록은 프로그램 상태를 일관성 있게 유지해야 합니다. :arrow_right: **try-catch 문은 블록 내 코드가 일관성있게 실행되게 해주기 때문에  트랜잭션과 비슷**하다고 할 수 있습니다.



### 예시 코드

```java
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file");
}

public List<RecordedGrip> retrieveSection(String sectionName) {
    // 아직 구현하기 전의 상태이기 때문에 비어있는 List를 반환
    return new ArrayList<RecordedGrip>();
}
```

위의 코드는 파일이 없으면 예외를 던지는지 테스트하는 코드입니다. 현재는 `retrieveSection()` 함수가 예외를 던지지 않기 때문에 위 테스트는 실패합니다.

그렇다면 아래와 같이 `retrieveSection()` 함수가 예외를 던지게 구현할 수 있습니다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName)
    } catch (Exception e) { // FileNotFoundException로 예외 유형을 좁힐 수도 있습니다.
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```

위와 같이 **try 블록 내에서 발생하는 특정 예외를 catch에서 잡게 만들어야 하기 때문에 try 블록의 범위부터 구현**하게 됩니다 :arrow_right: 트랜잭션 처럼 범위 단위로 생각하면서 코드를 구현할 수 있게 됩니다.





## 7-3. 미확인 예외를 사용하라

저자는 **확인된 예외를 지양하고 미확인된 예외를 사용**하라고 말합니다. 확인된 예외는 어떤 오류가 발생했는지 정확히 알 수 있지만 애플리케이션을 안정적으로 제공하는 관점에서는 반드시 필요하지 않습니다. :arrow_right: **애플리케이션에서는 어떤 오류든 애플리케이션이 중단되는 것은 똑같기 때문이라고 생각**합니다.

또한, 확인된 예외를 사용하면 try catch 에서 해당 예외들에 관한 처리를 일일이 해주어야 하기 때문에 아래의 코드 처럼 예외 **C가 D로 바뀌었다면, try catch문도 변경**해야 합니다. 게다가 **try catch 문에서 하위 함수들의 예외를 전부 알고 있어야 합니다.**  :arrow_right: `SOLID` 법칙의 `OCP` 위반, `객체 지향 4대원칙`의 `캡슐화` 위반

```java
public int exceptionTest() {
    try {
        A();
        B();
        C(); // C함수 내에서 C예외를 D예외로 변경
    } catch (ExceiptionA e) {
        logger.log(e)
    } catch (ExceiptionB e) {
        logger.log(e)
    } catch (ExceiptionD e) { // C예외를 D로 변경
        logger.log(e)
    }
}

...
    
public void C() {
    ...
    // throw new C("C예외를 던짐");
    throw new ExceiptionC("D예외를 던짐");
    ...
}
```

때로는 확인된 예외도 유용하지만, 일반적인 애플리케이션은 의존성이라는 비용이 존재하기 때문에 의존성을 최대한 줄이기 위해 미확인 예외를 사용하는 것이 좋다고 합니다. :arrow_right: **개인정보와 같은 민감한 정보를 다루는 코드에서는 확인된 예외 처리가 필요하지 않을까 생각**됩니다.



### 나의 생각

책에서는 `확인된 예외, 미확인 예외`라고 번역했는데 저는 두 예외가 아래와 같다고 생각합니다.

- **확인된 예외** : 예외를 디테일하게 정의해준 것 즉, 추상화 수준이 낮은 것 ex) `NullPointerException`
- **미확인 예외** : 추상적으로 정의하여 예외만 보고 정확한 내용까지 추측하기 어려운 것 ex) `Exception`

**참고 링크**

- [자바의 예외의 종류 3가지](https://velog.io/@jsj3282/%EC%9E%90%EB%B0%94%EC%9D%98-%EC%98%88%EC%99%B8%EC%9D%98-%EC%A2%85%EB%A5%98-3%EA%B0%80%EC%A7%80)





## 7-4. 예외에 의미를 제공하라

자바에서 예외가 발생하면 호츨 스택을 제공하지만, 실패한 코드의 의도를 파악하려면 호출 스택만으로는 부족합니다. 때문에 오류 메세지에 정보를 담아 로깅해주는 것이 좋다고 합니다. :arrow_right: **개인적으로는 7-3에서 확인된 예외를 사용하는 대신에 오류 메세지를 적극 활용**해주는 편이 더 좋은 것 같습니다.





## 7-5. 호출자를 고려해 예외 클래스를 정의하라

오류를 분류하는 다양한 방법이 있지만, **오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 오류를 잡아내는 방법**이 되어야 합니다.



### 오류 분류의 잘못된 예

아래의 코드는 **확인된 예외를 사용함으로써 try catch에서 다양한 예외에 대한 처리를 해야하지만 각 catch문 안에서의 행동은 똑같다**고 볼 수 있습니다.

```java
ACMEPort port = new ACMEPort(12);

try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception");
} finally {
 …
}
```



### Wrapper 클래스를 만들어 예외 처리

아래의 코드는 `ACMEPort`를 감싸는 `LocalPort`를 만들어 **각 예외를 추상화시켜 try catch에서 하나의 예외로 처리**할 수 있게 하였습니다.

```java
// Wrapper 클래스
public class LocalPort {
    private ACMEPort innerPort;
    
    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }
    
    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
    
    …
}

LocalPort port = new LocalPort(12);

 try {
     port.open();
 } catch (PortDeviceFailure e) { // 하나의 예외로 처리
     reportError(e);
     logger.log(e.getMessage(), e);
 } finally {
     …
 }
```

이렇게 **Wrapper 클래스를 만들어 사용하면, 테스트하기도 쉬워지고 의존성도 줄어들게 됩니다.** 흔히 예외 클래스가 하나만 있어도 충분한 코드가 많기 때문에 적절한 Wrapper 클래스 사용은 고려해볼만 합니다.



### 나의 생각

`Wrapper 클래스 안에서 각 예외에 대한 분기 처리를 해주기 때문에 결국 조삼모사가 아닌가??` 라고 생각할 수 있겠지만, 사용자가 일일이 분기처리를 해주지 않아도 되기 때문에 괜찮은 방법이라고 생각합니다.





## 7-6. 정상 흐름을 정의하라

때로는 예외를 던져 중단시키는 방식이 적합하지 않는 경우도 있습니다. 



### 예시 코드

아래의 코드와 같이 **catch를 하게되면 프로그램을 중단하는 것이아니라 분기처럼 다른 로직처리를 하게 됩니다.**

```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```

그런데 위와 같은 코드는 예외가 논리를 따라가기 어렵게 만듭니다. 이런 **특수한 상황을 처리하는 특수 사례 패턴을 사용해 이 문제를 해결**할 수 있습니다.

```java
public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // 기본값으로 일일 기본 식비를 반환
    }
}
```

위와 같이 `expenseReportDAO`의 `getMeals()` 함수가 `MealExpenses`만을 반환하기 때문에 **MealExpenses를 상속하여 getTotal() 함수를 Override 함으로써 특수한 상황에 관한 처리**를 해줄 수 있습니다.

> `특수 사례 패턴` : 특정 사례에 대해 특수한 동작을 제공하는 하위 클래스를 만들어 사용하는 것.
>
> 특정 객체의 필드(변수)가 null일 가능성이 있다면 null을 처리하는 하위클래스를 만들어서 처리합니다. 예를 들어, 고객 객체가 있을 때 null에 대한 검사를 피하고 싶다면 null 고객 객체를 만드는 것입니다. 고객 객체의 모든 메서드를 가지고 특수 사례에서 일부를 재정의 합니다. 이후에 null인 고객이 있으면 null 고객 객체의 인스턴스를 대신 사용하면 됩니다.





## 7-7. null을 반환하지 마라

null을 반환하는 습관은 오류를 유발하기 좋습니다.

```java
 public void registerItem(Item item) {
     if (item != null) {
         ItemRegistry registry = peristentStore.getItemRegistry();
         if (registry != null) {
             Item existing = registry.getItem(item.getID());
             if (existing.getBillingPeriod().hasRetailOwner()) {
                 existing.register(item);
             }
         }
     }
 }
```

null 체크가 잘못된 건 아니지만, 위와 같이 null 체크 로직이 많아지게되면 실수를 유발할 수 있는 확률이 높아집니다. :arrow_right: 실제로 위 코드에서 `peristentStore`의 null 체크가 빠져있습니다.

때문에 위에서 언급했던 **특수 사례 객체를 이용하거나 예외를 던지는 것이 낫습니다.**



### 나의 생각

저도 null을 반환하는 것 보다는 차라리 예외를 던지는 것이 낫다고 생각합니다. 예외를 던지지 않고 null을 반환하는 것은 프로그램을 중단시키지 않기 때문에 프로그래머가 예상하지 못한 오류가 발생할 수 있기 때문입니다.



## 7-8. null을 전달하지 마라

함수에서 null을 반환하는 방식도 나쁘지만 함수의 인수로 null을 전달하는 방식은 더 안좋습니다. **정상적인 인수로 null을 기대하는 API가 아니라면 함수에서 null을 전달받아 동작하는 코드는 최대한 피해야합니다.**



### 예시 코드

아래의 코드는 두 지점 사이의 거리를 계산하는 메서드입니다.

```java
public class MetricsCalculator
{
    public double xProjection(Point p1, Point p2) {
        return (p2.x – p1.x) * 1.5;
    }
    
    …
}
```

만약 **인수에 null을 전달하는 방식의 함수가 있다면 위의 코드에 누군가 실수로 인수로 null을 전달할 수도 있습니다. 때문에 아래와 같이 예외 처리나 null 체크 등이 필요**하게 됩니다.

```java
// 예외 처리
public class MetricsCalculator
{
     public double xProjection(Point p1, Point p2) {
         if (p1 == null || p2 == null) {
             throw InvalidArgumentException(
                 "Invalid argument for MetricsCalculator.xProjection");
         }
         return (p2.x – p1.x) * 1.5;
 }
    
    …
}

// null 체크
public class MetricsCalculator
{
    public double xProjection(Point p1, Point p2) {
        assert p1 != null : "p1 should not be null";
        assert p2 != null : "p2 should not be null";
        return (p2.x – p1.x) * 1.5;
    }
}
```

대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다고 합니다. 때문에 애초에 null을 넘기지 못하도록 금지하는 정책이 더 합리적입니다.





## 7-9. 7장을 마치며...

대부분은 예외 처리에 대한 익숙한 방법이었습니다. 경우에 따라 다르겠지만, **너무 세세한 예외 처리보다는 추상화가 가능하다면 각 예외 처리를 추상화시켜 사용하는 방법은 적절히 사용하면 좋을 것 같다는 생각**이 들었습니다.



---

참고 : Clean Code - 로버트 C. 마틴

[https://harrislee.tistory.com/63](https://harrislee.tistory.com/63)

