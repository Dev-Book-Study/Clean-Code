# 6장. 객체와 자료구조

6장에서는 객체와 자료구조에 제대로 구분하고 활용하는 방법에 대해 나와있습니다.



## 6-1. 자료 추상화

자료를 세세하게 공개하기 보다는 추상적인 개념으로 표현하는 것이 좋습니다.

```java
// 6-1 코드 구체적인 Point Class
public class Point {
	public double x;
	public double y;
}

// 6-2 추상적인 Point Class
public interface Point {
    double getX();
    double getY();
    void setCartesian(double x, double y);
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}
```

`6-1 코드`는 포인트 클래스를 구체적으로 정의하였지만, `6-2 코드`는 포인트를 인터페이스로 선언함으로써, 추상적으로 정의하였습니다. 또한, **6-2에서 값을 설정할 때는 set을 이용해 무조건 2개의 인자를 받아야 하기 때문에 값 설정에 실수를 줄일 수 있습니다.**

다만 **get, set 함수 내부에 다양한 조건을 추가할 수 있지만 각 값마다 get,set 함수를 제공한다면 구현을 외부로 노출시키는 것이나 마찬가지**입니다.

```java
// 6-3 코드 interface로 정의했지만 각 메서드가 어떤 일을 하는지 명확하게 나타냄
public interface Vehicle {
    double getFuelTankCapacityInGallons();
    double getGallonsOfGasoline();
}

// 6-4 추상적으로 메서드 이름을 설정함으로써 다양하게 사용가능
public interface Vehicle {
    double getPercentFuelRemaining();
}
```

**6-3 코드와 같이 인터페이스로 정의하더라도 메서드를 구체적으로 정의한다면 자료를 세세하게 공개하는 것이나 마찬가지**이기 때문에 **6-4 코드처럼 추상적인 개념으로 표현**하는 편이 좋습니다.





## 6-2. 자료/객체 비대칭

- `객체` : 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 제공 :arrow_right: ​`객체 지향 4대 원칙(캡슐화, 상속, 추상화, 다형성)`
- `자료구조` : 자료를 그대로 공개하며 별다른 함수는 제공하지 않음 :arrow_right: 말 그대로 자료만 관리



### 자료구조 형식(절차 지향)으로 만든 코드

```java
public class Square {
    public Point topLeft;
    public double side;
}
public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}
public class Circle {
    public Point center;
    public double radius;
}

// 도형이 작동하는 방식을 구현
public class Geometry {
    public final double PI = 3.141592653589793;
    
    public double area(Object shape) throws NoSuchShapeException
    {
        if (shape instanceof Square) {
            Square s = (Square)shape;
            return s.side * s.side;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle)shape;
            return r.height * r.width;
        } else if (shape instanceof Circle) {
            Circle c = (Circle)shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}
```

객체 지향 프로그래밍 관점에서 본다면 위와 같은 코드는 좋지 않을 수 있습니다.

하지만 **도형 클래스를 자료구조, Geometry 클래스를  자료구조 관리 클래스로 본다면 Geometry 클래스에 둘레 길이를 구하는 perimeter() 함수를 추가해도 도형 클래스에 아무런 영향을 끼치지 않습니다.**

그러나 **새 도형을 추가해야 한다면, Geometry 클래스에 속한 함수를 모두 수정**해야합니다.



### 객체 형식(객체 지향)으로 만든 코드

```java
public interface Shape {
    public double area();
}

public class Square implements Shape {
    private Point topLeft;
    private double side;
    
    @Override
    public double area() {
        return side*side;
    }
}

public class Rectangle implements Shape {
    private Point topLeft;
    private double height;
    private double width;
    
    @Override
    public double area() {
        return height * width;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;
    
    @Override
    public double area() {
        return PI * radius * radius;
    }
}
```

객체 형식으로 만든 코드는 자료구조 형식과는 다르게 **새 도형을 추가하면 기존 함수에 영향을 끼치지 않습니다.** :arrow_right: 각 객체마다 area를 재정의

하지만 **새 함수를 추가해야 한다면 도형 클래스 전부를 수정**해야 합니다.



### 정리

객체와 자료구조는 아래와 같은 특징을 가지고 있기 때문에 상황에 따른 센스있는 활용이 중요할 것 같습니다.

- `객체` : 객체 Class 추가가 쉬우나, 함수를 추가한다면 기존에 존재하는 객체도 전부 수정해야 함
- `자료구조` : 자료 구조 Class를 추가할 때마다 자료 구조 관리 Class도 수정해야하나, 함수를 추가하는 경우는 자료 구조 관리 Class만 수정해주면 됨



### 나의 생각

객체와 자료구조의 정의만 놓고본다면 그럴 수도 있다고 생각하지만, 요즘은 `절차 지향 프로그래밍` 보다는 대부분 `객체 지향 프로그래밍`, `함수형 프로그래밍`을 선호하기 때문에 위의 예시가 현재 시점에서는 적절하다고 생각되지 않습니다. :arrow_right: 자바의 경우 자료구조에서 타입을 `Object`라는 광범위한 인터페이스로 받기 때문에 대부분의 객체를 수용 가능함



## 6-3. 디미터 법칙

디미터 법칙은 잘 알려진 휴리스틱 입니다. 디미터의 법칙은 `Object-Oriented Programming: An Objective Sense of Style` 에서 처음으로 소개되었습니다.

Demeter라는 프로젝트를 진행하던 개발자들은 어떤 객체가 다른 객체에 대해 지나치게 많이 알다보니, 결합도가 높아지고 좋지 못한 설계를 야기한다는 것을 발견하였습니다. 그래서 이를 개선하고자 **객체에게 자료를 숨기는 대신 함수를 공개하도록 하였는데, 이것이 바로 디미터의 법칙**입니다.

즉, 디미터의 법칙은 다른 객체가 어**떠한 자료를 갖고 있는지 속사정을 몰라야 한다는 것을 의미**하며, 이러한 이유로 `Don’t Talk to Strangers(낯선 이에게 말하지 마라)` 또는 `Principle of least knowledge(최소 지식 원칙)` 으로도 알려져 있습니다.

> `휴리스틱` :경험에 기반하여 문제를 해결하거나 학습하거나 발견해 내는 방법을 말합니다.



디미터 법칙은 클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다고 합니다. :arrow_right: 말이 좀 난해한데, **저는 이 말이 f 메서드 안에서 어떤 객체를 사용할 것인가를 말하는 것이라고 생각**합니다.

- 클래스 C
- f가 생성한 객체
- f 인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체

위와 같은 조건은 `Don’t Talk to Strangers(낯선 이에게 말하지 마라)` 조건을 생각하면 됩니다.



### 기차 충돌

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

위와 같은 코드는 **여러 객체가 한 줄로 이어진 기차처럼 보이기 때문**에 흔히 `기차 충돌`이라고 부릅니다. 위 코드는 가독성을 위해서라도 아래와 같이 나누는 편이 좋습니다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

하지만 위의 코드도 **하나의 함수가 여러 가지 객체를 탐색하게 되기 때문에 디미터 법칙을 위배**하게 됩니다.

객체라면 내부 구조를 숨겨야 하기 때문에 디미터 법칙을 위반하게 되고, 자료구조라면 아래와 같이 내부 구조를 노출시키기 때문에 디미터 법칙이 적용되지 않습니다.

```java
final String outputDir = ctxt.options.scratchDir.absolutePath;
```



#### 나의 생각

요즘 같은 많은 복잡한 정보가 연관되어 있는 어플리케이션에서 디미터 법칙을 지키는 것은 거의 불가능하다고 생각하고 마지막 자료구조 코드 처럼 프로그래밍 하는 것은 더더욱 말이 안된다고 생각합니다. 때문에 이런 게 있다 정도로만 알고가면 좋지 않을까 싶습니다.



### 잡종 구조

**중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조희/설정 함수도 있는 객체와 자료 구조를 섞어 놓은 구조**를 말합니다. 이런 잡종 구조는 객체와 자료 구조의 단점만 모아놓은 구조이기 때문에 피하는 편이 좋다고 합니다.



### 구조체 감추기

위 예시의 `ctxt, options, scratchDir`는 임시 디렉토리의 절대 경로를 얻어서 임시 파일을 생성하게 됩니다. 때문에 위 예시 처럼 공개해야하는 메서드가 많아지는 것 보다는 아래와 같이 ctxt 객체에 `createScratchFileStream` 메서드로 임시 파일을 생성하게 만드는 방법이 더 좋다고 합니다.

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```



#### 나의 생각

3장 함수에서 나왔던 추상화 수준과 연관지어서 생각할 수 있습니다. `추상화 수준이 높은 함수(createScratchFileStream)`로 `추상화 수준이 낮은 함수(ctxt.getOptions, opts.getScratchDir, getAbsolutePath)`을 감싸서 **추상화 수준을 하나로 통일해줌으로써 한 가지 일만 할 수 있게 만드는 것**입니다.





## 6-4. 자료 전달 객체

자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스를 말합니다. 이런 자료 구조체를 때로는 `자료 전달 객체 (DTO : Data Transfer Object)`라 합니다. 

DTO는 데이터베이스와 통신하거나 소켓에세 받은 메세지의 구문을 분석할 때 유용합니다. **DTO는 데이터베이스에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 형식으로 많이 사용**합니다.

DTO 같은 자료구조이지만 save나 find와 같은 탐색 함수도 제공하는 활성 레코드에 대해 나오는데, 너무 추상적이고 난해해서 아래의 링크를 참고하면 좋을 것 같습니다.

**참고 링크**

- [Spring : DTO(Data Transfer Object), DAO(Data Transfer Object), Connection Pool, Data Source](https://dev-splin.github.io/spring/Spring-DTO,DAO,Connection-Pool,Data-Source/)
- [Entity, VO, DTO의 차이](https://dev-splin.github.io/spring/Spring-Entity-DTO-VO/)





## 6-5. 6장을 마치며...

이 장에서는 객체와 자료구조의 차이에 대해 계속적으로 언급하는데, 저자가 소개한 자료구조(특히 처음 예시...)는 `객체 지향 프로그래밍`, `함수형 프로그래밍`을 기반으로 하는 언어가 많은 현 시점에서는 공감하기가 어려운 것 같습니다.

때문에 객체와 자료구조는 아래와 같이 개념적으로만 파악하면 좋을 것 같다는 게 저의 개인적인 생각입니다.

- 객체 : 동작을 공개하고 자료를 숨기며, 기존 동작을 변경하지 않으면서 새 객체를 생성하기 쉬움(확장성)
- 자료구조 : 별다른 동작 없이 자료를 노출



---

참고 : Clean Code - 로버트 C. 마틴

[https://mangkyu.tistory.com/147](https://mangkyu.tistory.com/147)
