## 왜 싱글턴은 모킹이 어려운가?

### 정리

**내용 요약**

싱글턴은 함수와 같은 무상태(stateless) 객체나 설계 상 유일성이 보장되어야 할 경우에 이용되고는 한다. 싱글턴 생성에는 보통 `public static final` 혹은 정적 팩터리 메서드가 쓰인다. 여기에서 전자와 후자는 각각의 장단을 가진다.

`public static final`을 이용한 싱글턴 생성은 인스턴스가 시스템에서 하나가 됨을 보장한다. 그런데 이는 클래스 정보를 읽어오는 리플렉션 API를 이용한 private 생성자 호출에 주의해야 한다. 또한, 직렬화-역직렬화에서 `Serializable` 인터페이스와 `transient` 키워드를 이용해 싱글턴을 보장해야 할 필요가 있다. 그렇지 않으면 `getInstance()` 를 통해 반환되지 않았으므로, 매 번 새로운 인스턴스가 생성되는 까닭이다.

정적 팩터리 방식 역시도 리플렉션을 통한 생성자 호출과 직렬화-역직렬화에는 주의해야 한다는 단점은 있다. 하지만 손쉽게 싱글턴 방식에서 여러 인스턴스를 생성하는 방식으로 바꿀 수 있다는 점에서는 유리하다. 게다가 코드를 더욱 유연한 제네릭 팩토리 방식으로 확장할 수도 있다. (메서드 참조를 supplier로 사용한다는 설명은 이해가 확실치 않아, 다른 글에서 후술하겠다.)

저자는 추천하는 싱글턴 생성 방식으로 열거 타입을 꼽는다. `enum` 클래스의 생성자는 `private`이기에 직접적으로 호출될 수 없고, 클래스가 접근될 때 `public static final`로  생성되는 까닭이다. 즉, 예시 코드에서, `enum` 클래스는 `Elvis.INSTANCE` 에 처음으로 접근될 때에 초기화되고, `Elvis.INSTANCE`는 싱글턴으로 유지된다. 물론 싱글턴이 Enum 외의 클래스를 상속해야 할 때에는 `final` 제한자로 인해 이용할 수 없다. (인터페이스 구현은 가능하다.) 

### 심화 탐구

**출발점**

세부적인 내용 이해도 쉽지만은 않았다.  글의 시작점에서, 저자는 싱글턴은 모킹(mocking)이 어렵기 때문에 테스트에의 어려움이 있을 수는 있다고 말한다. 하지만 싱글턴이 모킹이 어려운 이유에 대해서 저자는 충분히 설명하지 않는다. 부족한 이해를 보충하기 위해, 테스트에 있어서 싱글턴과 모킹의 관계를 우선적으로 살펴볼 필요가 있겠다.

**설명**

1. 모킹이란 무엇인가

우선 여기에서 말하는 테스트란 유닛 테스트(unit test)를 일컫는 것으로 보인다. 유닛 테스트는 가장 작은 기능적 단위인 유닛(unit) 테스트하는 프로세스를 의미한다. 이 프로세스는 코드 변경시마다 자동으로 실행되며, 분리된 코드 영역은 디버깅에의 효율성으로 향한다. 아래는 유닛 테스트에서 유용하게 쓰이는 Junit 라이브러리를 사용하지 않은, 유닛 테스트의 예시적 코드다.

```java
// https://stackoverflow.com/questions/4325014/how-to-do-unit-testing-without-the-use-of-a-library

class Foo {
    public int bar(int input);
}

class TestFoo {

    public void testBarPositive() {
        Foo foo = new Foo();
        System.out.println(foo.bar(5) == 7);
    }

    public void testBarNegative() {
        Foo foo = new Foo();
        System.out.println(foo.bar(-5) == -7);
    }

    public static void main(String[] args) {
        TestFoo t = new TestFoo();
        t.testBarPositive();
        t.testBarNegative();
    }
}

```

유닛 테스트에서 어떠한 객체 혹은 유닛은 다른 객체들 또는 유닛들로부터 의존성을 가진다. 만약 위 코드에서 `Foo` 클래스가 `Bar` 클래스에 의존성을 가진다면, 이를 더 이상 유닛 테스트라고 부르기엔 무리일지도 모른다. 따라서 의존성에서 격리시키기 위해, 다른 객체 혹은 유닛들의 행동(behaviour)을 구현하는 모의 객체인 목(mock)이 의존성을 줄이기 위해 쓰인다. 그렇다면 싱글턴과 모킹은 객체의 생성에서 연관되어 있는 개념이라고도 할 수 있다.

2. 싱글턴과 모킹

예시적인 코드를 살펴보자. (이후의 설명은 코드의 출처인 stackoverflow 글에기반한다.)

```java
// https://stackoverflow.com/questions/2302179/mocking-a-singleton-class
class Singleton {
    private static final myInstance = new Singleton();
    public static Singleton getInstance () { return myInstance; }
    private Singleton() { ... }
    // public methods
}

class Client {
    public doSomething() {
        Singleton singleton = Singleton.getInstance();
        // use the singleton
    }
}
```

문제는 두 가지다. 우선적으로, 생성자가 `private`하므로 테스트 내에서 상속을 통한 하위 클래스 인스턴스의 생성을 이용할 수도 통제할 수도 없다. 이는 곧 다형성을 고려할 수 없다는 뜻이기도 하다. 나아가 만약 `Client`의 `doSomething`에 대한 테스트 코드가 `Singleton`을 이용해야 한다면, 어떻게 목(mock)이 싱글턴의 행동을 구현하고 대체할 수 있는가?

싱글턴이 유닛 테스트에서 가지는 문제점을 현대 프레임워크의 이용 없이 어떻게 개선할 수 있을까? 이는 곧 싱글톤 구현을 테스트 가능한 것으로 만드는 일에 대한 질문이기도 하다. 즉, (1) 하위 클래스에 대한 확장성과 (2) `Singleton`에 대한 대체를 만족하는 방식이 필요하다. 

위 글에서는 진짜(real) 객체는 물론 모킹 하위 클래스에 구현되는 인터페이스를 제공하는 방법과 유닛 테스트 내 인스턴스의 교체를 허락하는 `setInstance` 메서드를 추가하는 방식을 소개한다.

```java
// https://stackoverflow.com/questions/2302179/mocking-a-singleton-class
interface Singleton {

    // 아래는 원래 코드이나, interface에서 private field는 허용되지 않음
    // private static final myInstance;

    
    public static Singleton getInstance() { return myInstance; }
    public static void setInstance(Singleton newInstance) { myInstance = newInstance; }
    // public method declarations
}

// Used in production
class RealSingleton implements Singleton {
    // public methods
}

// Used in unit tests
class FakeSingleton implements Singleton {
    // public methods
}

class ClientTest {
    private Singleton testSingleton = new FakeSingleton();
    @Test
    public void test() {
        Singleton.setSingleton(testSingleton);
        client.doSomething();
        // ...
    }
}
```

`Singleton` 인터페이스는 `setInstance()`를 통해서 `myInstance`의 참조값을 인자로 받은 싱글턴 인스턴스로 교체할 수 있도록 만든다. 하지만 원본 코드에서는 interface 내에 private field를 이용했으나, 이러한 방법은 컴파일 시점에서 오류를 발생시키기에 수정이 필요하다. (해당 코드는 보충이 필요하며, 인터페이스를 이용하는 다른 방식에 대한 탐구할 필요가 있다.)

여기에서 인터페이스를 잘 이용하고 있다고 가정하면, `RealSingleton`과 유닛 테스트를 위해 모킹된 `FakeSingleton`은 모두 `Singleton` 인터페이스에 대한 구현 클래스가 된다. 이제 `ClientTest`에서도 보이듯, 유닛 테스트 코드에서도 기존 `setInstance()` 메서드를 이용해 모킹 클래스를 싱글턴에서 이용할 수 있기에, 테스트 용이성을 증가시킬 수 있다.

무엇보다도 중요한 것은 상황에 싱글턴 패턴이 적절한가이겠지만, 싱글턴으로 된 코드를 테스트하고 수정해야 할 날이 올 지도 모른다. 이번 글은 JUnit과 Mockito를 이용하지 않고 유닛 테스트를 수행하는 것보다도, 기초적인 유닛 테스트 개념과 싱글턴 패턴의 단점을 우회하는 법을 살펴보았다.

### References

https://docs.oracle.com/javase/8/docs/technotes/guides/language/enums.html

https://stackoverflow.com/questions/26285520/implementing-singleton-with-an-enum-in-java

https://stackoverflow.com/questions/16771373/singleton-via-enum-way-is-lazy-initialized

https://stackoverflow.com/questions/19440511/explanation-needed-why-adding-implements-serializable-to-singleton-class-is-ins?rq=1

https://www.baeldung.com/java-factory-pattern-generics

https://stackoverflow.com/questions/2302179/mocking-a-singleton-class

https://stackoverflow.com/questions/2665812/what-is-mocking

https://en.wikipedia.org/wiki/Unit_testing

https://aws.amazon.com/ko/what-is/unit-testing/

https://www.baeldung.com/java-final

**Furthermore**

https://stackoverflow.com/questions/40244571/when-we-should-use-supplier-in-java-8

https://dzone.com/articles/how-to-use-enums-in-java

https://stackoverflow.com/questions/3622455/what-is-the-purpose-of-mock-objects