## 다중 상속 관계에서의 다이아몬드 문제

`Effective Java` 4장 클래스와 인터페이스 - '아이템 20 - 추상 클래스보다는 인터페이스를 우선하라' 를 읽고

### 정리

**내용 요약**

1. 다중 구현 매커니즘

자바가 제공하는 다중 구현 메커니즘은 **추상 클래스(Abstract Class)**와 **인터페이스(Interface)**가 있다. 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.

- 다중 구현 매커니즘: 다중 구현 메커니즘은 한 클래스가 여러 인터페이스를 구현하거나 다중 상속의 특성을 모방하는 방식이다. 자바에서는 클래스가 다른 클래스를 단 하나만 상속할 수 있지만, 인터페이스를 통해서는 여러 인터페이스를 구현할 수 있다. 이를 통해 다중 상속의 장점을 활용 가능하다.

**추상 클래스**

추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 **하위 클래스**가 되어야 하는 제약이 있다.

- 추상 클래스를 확장한 클래스는 다른 클래스를 상속받을 수 없게 된다.
- 기존 클래스 위에 새로운 추상 클래스를 끼워넣기 어렵다.
    - 두 클래스가 같은 추상 클래스를 확장하고자 하는 경우, 해당 추상 클래스는 두 클래스의 공통 조상이어야 한다. (그렇지 않으면 계층 구조에 혼란을 일으킨다.)

**인터페이스**

인터페이스가 선언한 메서드를 모두 정의한 클래스는 상속 여부와 상관없이 같은 타입이 된다. 기존 클래스들도 새로운 인터페이스를 쉽게 구현할 수 있다. 또한 인터페이스는 믹스인 정의에 유용하다.

- 믹스인(Mix-In)이란 클래스가 구현할 수 있는 타입이다.
- 대상 타입의 주된 기능에 선택적 기능을 혼합하여 제공한다고 선언하는 효과를 준다.
    - Comparable은 자신을 구현한 클래스의 인스턴스들은 순서를 정할 수 있음을 선언한다.
- 추상 클래스는 믹스인을 정의할 수 없다.

인터페이스는 자신을 구현하는 모든 클래스가 같은 기능을 구현하도록 강제하는 것에 목적이 있다.

자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 말해주는 타입 정의 용도로 사용하며, 추상 클래스와는 다르게 **기능에 초점**을 맞춘다.

인터페이스는 다중 상속을 지원하기 때문에 새로운 타입을 정의하는 데에 제약이 없으며, 구조적 표현이 중요하지 않으므로 전혀 다른 클래스도 같은 인터페이스를 가질 수 있다.

2. 템플릿 메서드 패턴

인터페이스와 추상 골격 구현(Skeletal Implementation) 클래스를 함께 제공하는 방법이다.

- 인터페이스로 타입을 정하고 골격 구현 클래스가 나머지 메서드들까지 구현한다.
- 단순히 골격 구현 클래스를 확장하는 것만으로 인터페이스를 구현하는 일이 대부분 완료된다.
- 인터페이스와 추상 클래스의 장점을 모두 갖는다.
    - 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때의 제약에서 자유롭다.
- 관례상 인터페이스 이름이 `XXX`라면 추상 골격 구현 클래스는 `AbstractXXX` 이다.

**골격 구현 작성**

- 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정한다.
    - 이 기반 메서드들은 골격 구현에서는 추상 메서드가 된다.
- 기반 메서드들을 통해 직접 구현할 수 있는 메서드는 디폴트 메서드로 제공한다.
- 기반 메서드나 디폴트 메서드로 만들지 못한 메서드들은 인터페이스를 구현하는 골격 구현 클래스를 만들어 작성해 넣는다.
    - 골격 구현 클래스는 필요하다면 `public` 필드와 메서드를 추가해도 된다.

3. 결론

일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 

복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자. 골격 구현은 '가능한 '한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. '가능한 한' 이라고 한 이유는, 인터페이스에 걸려 있는 구현상의 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.

### 심화 탐구

**출발점**

자바에서는 왜 다중상속을 허용하지 않는걸까? 그리고 어떻게 인터페이스를 통해서는 다중상속이 가능한건지에 대한 의문을 해소하기 위해 심화 탐구를 시작했다.

**설명**

다중 상속은 한 클래스가 두 개 이상의 상위 클래스로부터 상속을 받을 수 있게 하는 기능이다. 다른 객체지향언어인 C++에서는 여러 조상 클래스로부터 상속받는 것이 가능한 **'다중상속(multiple inheritance)'** 을 허용하지만 자바에서는 오직 단일 상속만을 허용한다.

다중 상속 시 주요 문제점으로는 다이아몬드 문제가 있다.

### **다이아몬드 문제**

다이아몬드 문제는 한 클래스가 두 개의 상위 클래스로부터 상속을 받고, 이 두 상위 클래스가 동일한 조상 클래스를 가지고 있을 때 발생한다. 이러한 경우, 하위 클래스에서 조상 클래스의 메서드를 호출할 때, 어느 쪽 상위 클래스의 메서드를 호출해야 하는지 모호해지는데, 이는 클래스 상속 구조가 다이아몬드 형태를 이루기 때문에 발생한다.

**예시**

다이아몬드 문제를 알아보기 위해 아래와 같은 상속관계를 갖는 클래스들을 생성해보겠다.

Child → Mother → Person

        Father ↗

```java
public class Person {
	
	String nation = "대한민국";
	
	protected void 가문() {
		System.out.println("가문");
	}
}
```

```java
public class Father extends Person {
	@Override
	protected void 가문() {
		System.out.println("김가");
	}
}
```

```java
public class Mother extends Person {
	@Override
	protected void 가문() {
		System.out.println("이가");
	}
}
```

```java
public class Child extends Mother {
	public static void main(String[] args) {
		Child child = new Child();
		System.out.println("국적은 " + child.nation);
		child.가문();

		// 실행결과
		// 국적은 대한민국
		// 이가
	}
}
```

`Person` 클래스에 선언된 필드를 `Mother` 클래스의 상속을 거쳐 `Child` 까지 실행시켰다.

이 때, 만약 자바에서 다중상속이 허용되어서 `Child`가 `Mother`과 `Father`를 모두 상속한다고 가정해보자.

Child → Mother → Person

    ↘Father ↗

위와 같은 상속 관계에서 `Child`는 `Father`와 `Mother`의 멤버 필드와 메소드를 상속받을 것이다. 그런데 `Father`와 `Mother`에 같은 메소드가 있다면 어떨까? `Father`와 `Mother`는 `Person`을 상속받으면서 `가문()`을 서로 다르게 오버라이딩했다.

그렇다면 Father와 Mother를 상속받는 Child에서 `child.가문()`은 어떤걸 출력할까?

이 애플리케이션을 실행해야하는 JVM에서는 `Father`에도 선언되어 있고, `Mother`에도 선언되어 있는 `가문()`중 어떤걸 `Child`가 상속받아야하는지를 모르기 때문에 **컴파일 에러**가 발생한다.

그림의 형태처럼, 자바에서 다중상속 시 벌어지는 문제를 **다이아몬드 문제** 라고 한다.

### **자바에서의 해결 방법**

자바는 이러한 문제를 피하기 위해 클래스에 대한 다중 상속을 허용하지 않는 대신, 인터페이스를 통한 다중 구현을 지원하고 있다.

```java
public interface Person {
   String nation = "대한민국";
   void 가문();
}
```

```java
public interface Father extends {
   @Override
   void 가문();
}
```

```java
public interface Mother extends Person {
   @Override
   void 가문();
}
```

```java
public class Child implements Father, Mother {
   @Override
   void 가문(){
      System.out.println("이가");
   }
  
   public static void main(){
      Child child = new Child();
     
      System.out.println("국적은 "+child.nation);
      child.가문();
   }
}
```

상속받은 메소드가 구현체가 없는 추상 메소드이기 때문에 인터페이스를 통한 다중상속은 가능하다.

어차피 구현체가 없기 때문에 `Child`가 상속받은 메소드가 `Father`로부터 받은건지, `Mother`로부터 받은건지 알 필요가 없다. (어차피 상속받은 객체인 `Child`에서 구현을 할 것이기 때문이다.)



*참고 자료*

*[https://docs.oracle.com/javase/tutorial/java/IandI/multipleinheritance.html](https://docs.oracle.com/javase/tutorial/java/IandI/multipleinheritance.html)*

*[https://www.geeksforgeeks.org/java-and-multiple-inheritance/](https://www.geeksforgeeks.org/java-and-multiple-inheritance/)*

*[https://jjingho.tistory.com/158](https://jjingho.tistory.com/158)*

*[https://prodo-developer.tistory.com/85](https://prodo-developer.tistory.com/85)*

*[https://www.geeksforgeeks.org/diamond-problem-in-java/](https://www.geeksforgeeks.org/diamond-problem-in-java/)*

[https://youngjinmo.github.io/2021/03/diamond-problem/](https://youngjinmo.github.io/2021/03/diamond-problem/)*