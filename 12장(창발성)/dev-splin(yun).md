# 12장. 창발성

이 챕터는 이제까지의 깨끗한 코드에 대해서 정리하는 느낌의 챕터라고 보시면 될 것 같습니다.

창발성에서 **창발이란 하위 계층에는 없는 특성이나 행동이 상위 계층에서 자발적으로 돌연히 출현하는 현상**(영어로는 `Emergent Property`)을 말합니다.

개인적인 생각으로 이 `창발`이라는 단어의 뜻을 보았을 때, 여기서 창발성이란 단어로 말하고 싶은 것은 상위 계층 즉, 추상화를 통하여 깔끔한 코드를 구현하자라는 의미인 것 같습니다.





## 12-1. 창발적 설계로 깔끔한 코드를 구현하자

`켄트 벡`은 아래의 단순한 설계 규칙 4가지가 소프트웨어 설계 품질을 크게 높여줄 것을 믿는다고 합니다.

1. 모든 테스트를 실행한다.
2. 중복을 없앤다.
3. 프로그래머 의도를 표현한다.
4. 클래스와 메서드 수를 최소로 줄인다.

1번 부터 중요도 순이며, **위의 4가지 규칙을 따르면 코드 구조와 설계 파악이 쉬워지고 SRP나 DIP와 같은 객체 지향 원칙도 지킬 수 있다고 합니다.**





## 12-2. 단순한 설계 규칙 1 : 모든 테스트를 실행하라

설계는 의도한 대로 돌아가는 시스템을 내놓아야 합니다. 문서로는 시스템을 완벽하게 설계했지만, 시스템이 의도한 대로 돌아가는지 검증할 간단한 방법이 바로 테스트 코드를 작성하는 것이라고 합니다. 

테스트를 통과한 다는 것은 검증이 가능하다는 것이기 때문에 테스트는 중요합니다. :arrow_right: **테스트가 가능한 시스템을 만들려고 애쓰다보면 SRP, DIP 등을 준수하게 되고, 의존성 주입(DI), 인터페이스, 추상화 등과 같은 도구를 사용해 결합도를 낮춤으로써 설계 품질이 높아진다**고 합니다.

즉, **테스트 코드를 만드려는 노력 자체가 낮은 결합도와 높은 응집력이라는, 객체 지향 방법론이 지향하는 목표에 가까워 지게 된다는 것**입니다.





## 12-3. 단순한 설계 규칙 2~4 : 리팩터링

리팩토링은 테스트코드를 작성한 후 진행하는 것이 좋다고 합니다. :arrow_right: **리팩토링을 하다보면 코드를 추가하거나 변경하게 되는데, 이 때 테스트 코드를 통하여 시스템이 깨질까 걱정할 필요가 없기 때문**입니다.





## 12-4. 중복을 없애라

이제까지 계속 나왔던 말입니다. 똑같은 코드는 물론이고 비슷한 코드는 오히려 더 비슷하게 고쳐줌으로써 리팩토링이 쉬워진다고 합니다.



### 예시 코드 1

```java
int size() {}
boolean isEmpty() {}
```

위와 같이 흔히 사용하는 `size`와 `isEmpty`를 정의할 때 아래와 같이 `isEmpty`는 `size` 함수를 이용하면 더 쉽게 구현가능합니다.

```java
boolean isEmpty() {
    return 0 == size();
}
```



### 예시 코드 2

```java
public void scaleToOneDimension(
    float desiredDimension, float imageDimension) {
    if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
        return;
    float scalingFactor = desiredDimension / imageDimension;
    scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
    
    // 이 부분이 rotate의 함수와 비슷함
    RenderedOp newImage = ImageUtilities.getScaledImage(
        image, scalingFactor, scalingFactor);
    image.dispose();
    System.gc();
    image = newImage;
 }

 public synchronized void rotate(int degrees) {
     RenderedOp newImage = ImageUtilities.getRotatedImage(image, degrees);
     image.dispose();
     System.gc();
     image = newImage;
 }
```

위 코드의 주석으로 적어놨듯이, 비슷한 부분이 있는 두 코드를 아래와 같이 변경할 수 있습니다.

```java
public void scaleToOneDimension(
    float desiredDimension, float imageDimension) {
    if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
        return;
    float scalingFactor = desiredDimension / imageDimension;
    scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
    
    // 중복된 부분을 replaceImage함수로 뺌
    replaceImage(ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor));
}

 public synchronized void rotate(int degrees) {
     replaceImage(ImageUtilities.getRotatedImage(image, degrees));
}

 private void replaceImage(RenderedOp newImage) {
     image.dispose();
     System.gc();
     image = newImage;
}
```

이렇게 비슷한 부분을 함수로 뺌으로써, 코드가 깔끔해집니다. 또한, **새로 만든 replaceImage는 또 다른 책임을 가지고 있어 SRP를 위반하기 때문에 다른 클래스로 옮겨 새 메서드의 가시성이 높아집니다.**



### 예시 코드 3

템플릿 메소드(Template Method) 패턴을 이용하는 방법입니다.

```java
public class VacationPolicy {
    public void accrueUSDivisionVacation() {
        // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
        // ...
        
        // 휴가 일수가 미국 최소 법정 일수를 만족하는지 확인하는 코드 (이 부분만 다름)
        // ...
        
        // 휴가 일수를 급여 대장에 적용하는 코드
        // ...
 }
    
 public void accrueEUDivisionVacation() {
     // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
     // ...
     
     // 휴가 일수가 유럽연합 최소 법정 일수를 만족하는지 확인하는 코드 (이 부분만 다름)
     // ...
     
     // 휴가 일수를 급여 대장에 적용하는 코드
     // ...
 }
}
```

위의 코드를 아래와 같이 **추상화를 통해 중복된 부분에서 다른 부분만 추상화 시켜 코드를 깔끔하게 만들 수 있습니다.**

```java
abstract public class VacationPolicy {
    public void accrueVacation() {
        calculateBaseVacationHours();
        alterForLegalMinimums();
        applyToPayroll();
 }
    
	private void calculateBaseVacationHours() { /* ... */ }; 
	abstract protected void alterForLegalMinimums();
	private void applyToPayroll() { /* ... */ };
}

public class USVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        // 미국 최소 법정 일수를 사용함
    }
}
public class EUVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        // 유럽연합 최소 법정 일수를 사용함
    }
}
```





## 12-5. 표현하라

자신이 코드를 작성하는 동안에는 해당 문제에 대한 이해가 빠릅니다. 하지만 나의 코드를 볼 다른 사람은 이해하는데 시간이 상대적으로 오래 걸리고 오해할 수 있기 때문에 **아래의 규칙들을 지켜 개발자의 의도를 분명히 표현**해야 합니다.

1. 좋은 이름을 선택 합니다. :arrow_right: 이름과 기능이 완전히 딴판인 클래스나 함수로 유지보수 담당자를 놀라게 해서는 안 됩니다.
2. 함수와 클래스 크기를 가능한 줄입니다. :arrow_right: 작은 클래스와 작은 함수는 이름 짓기도 쉽고, 구현하기도 쉬우며, 이해하기도 쉽습니다.
3. 표준 명칭을 사용합니다. :arrow_right: 표준 디자인 패턴(Command, Visitor 등..)을 사용할 때, 클래스 이름에 패턴 이름을 넣어주면 클래스 설계 의도를 이해하기 쉬워집니다.
4. 단위 테스트 케이스를 꼼꼼히 작성합니다. :arrow_right: 테스트 케이스는 예제라고 생각하면, **잘 만든 예제를 보면 이해하기가 쉽듯이, 잘 만든 테스트 케이스를 읽어보면 클래스 기능이 이해하기 쉬워집니다.**

**표현력을 높이는 가장 중요한 방법은 노력입니다. 코드만 작성한 후 넘어가는 것이 아니라, 나중에 읽을 사람(자신이 될수도 있음)을 고려해 조금이라도 읽기 쉽게 만드려는 충분한 고민이 필요**합니다. 



### 나의 생각

이제 까지 `디자인 패턴을 적용한다면, 주석으로 설명해야지..` 라고 생각했습니다. 그러나 위와 같이 네이밍이 헷갈리지 않는 선에서 클래스 이름에 디자인 패턴 이름을 넣는 것도 괜찮을 것 같다는 생각이 들었습니다.





## 12-6. 클래스와 메서드 수를 최소로 줄여라

SRP를 준수한다는 기본적인 개념도 극단으로 치달으면 득보다 실이 많아질 때가 있다고 합니다. 그래서 **함수와 클래스 수를 가능한 줄이라고 제안하는 것**이라고 합니다.

정책 때문에 클래스 수와 메서드가 늘어나거나, 자료 클래스와 동작 클래스를 무조건 분리해야 한다는 주장 같은 **독단적인 견해는 멀리하고 실용적인 방식을 택해야 합니다.**

**목표는 함수와 클래스 크기를 작게 유지하면서 동시에 시스템 크기도 작게 유지**하는 데 있습니다. 하지만 **4개의 규칙 중 제일 나중의 우선순위**를 가지고 있습니다. 이 의미는 클래스와 함수를 줄이는 작업도 중요하지만, **테스트 케이스를 만들고 중복을 제거하고 의도를 표현하는 작업이 더 중요**하다는 뜻입니다.





## 12-7. 12장을 마치며...

저자는 결론에서 이 책에서 소개하는 기법들은 **저자들이 수십 년 동안 쌓은 경험을 토대로 만든 것이기 때문에 이런 기법들을 알고 있다고 경험을 대신할 수는 없다**고 말합니다. 

이런 경험들의 노하우가 쉽게 이해될 수도 있지만, 반대로 한 번에 이해되지 않을 수 있다고 생각합니다. 때문에 지금 당장 이해되지 않는다고 자신을 자책할 것이 아니라, 경험들의 노하우가 길이라고 생각하고 그 길을 따라가는 것이라고 생각하면 좋을 것 같습니다. 

바닷길에서 바다가 지금 이해되지 않는 노하우라고 한다면, 바닷길에서도 바다가 보이는 위치가 있고 안보이는 위치가 있듯이, 지금 보이지 않는다고 내가 부족한 것이 아니라, 그 길을 따라 한 걸음 한 걸음 따라가다 보면, 바다가 보이는 위치에 다다를 수 있다고 저는 생각합니다. 



---

참고 : Clean Code - 로버트 C. 마틴

