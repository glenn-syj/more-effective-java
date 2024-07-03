## 내부 클래스와 람다의 this 키워드

### 정리

자바 8부터 람다식을 통해 인스턴스를 만듦으로써, 코드를 간결하게 작성할 수 있게 되었다. 자바의 타입 추론 규칙의 복잡성은 람다식에서 일단 타입을 생략하는 것이 우위에 서도록 만든다. 자바 타입 추론 규칙은 제네릭에서 필요한 정보를 얻기도 하기에, 제네릭의 중요성은 더욱 커진다.

하지만 람다에게도 단점은 있다. 람다식은 이름이 없고 문서화도 불가능하다. 즉, 람다식은 간결함과 가독성이 생명이다. 게다가 추상 클래스의 인스턴스를 만들 때는 람다를 아예 쓸 수 없다.

알아둬야 할 다른 주의점도 있다. 람다의 this 키워드는 자신이 아니라 바깥 인스턴스를 가리키는 반면 익명 클래스의 this는 자기 자신을 가리킨다. 자신을 참조하기 위한 this는 익명 클래스에서 이용해야 한다.

마지막으로, 람다의 직렬화에 조심해야 한다. 이는 가상머신마다 직렬화 형태가 다르기 때문이며, 직렬화해야 하는 객체가 있다면 private 정적 중첩 클래스의 인스턴스 이용이 권장된다.

### 심화 탐구

**출발점**

왜 람다의 this 키워드는 자신이 아니라 바깥을 가리키고, 익명 클래스의 this는 자기 자신을 가리키는가? 이러한 차이는 어디에서 발생하는가? 람다에서 클로저가 어떻게 작용되는지 살펴보기 전에, 현상부터 살펴볼 필요가 있다.

**설명**

Java 8에서는 람다 표현식, 클로저, 그리고 익명 함수가 도입되었다. 특히, 람다 표현식은 코드 블록을 나중에 실행하기(later execution) 위한 목적이 컸다. 이는 지연 연산(lazy evaluation)으로도 불린다. 즉, 람다 표현식은 생성된 시점이 아니라 필요할 때 실행되도록 설계된다.

특히, 렉시컬 스코프의 도입은 이전과 확실히 달랐다. 이는 내부 클래스에서와 달리 람다의 본문(body)에서 이름 구분자를 컴파일러가 다루는 방식과 관련된다. 아래의 코드는 [오라클](https://www.oracle.com/technical-resources/articles/java/architect-lambdas-part1.html)에서 가져왔다.

```java
// 코드 1-A. inner class and this
class Hello {
  public Runnable r = new Runnable() {
      public void run() {
        // Hello$1@f7ce53 출력
        System.out.println(this);
        System.out.println(toString());
      }
    };

  public String toString() {
    return "Hello's custom toString()";
  }
}

public class InnerClassExamples {
  public static void main(String... args) {
    Hello h = new Hello();
    h.r.run();
  }
}
```

코드 1-A에서는 익명 클래스를 사용해 `Runnable` 인터페이스가 구현되었다. `Runnable` 구현체에서 `this`와 `toString()`의 호출은 표현식을 만족하는 가장 내부의 스코프로 바인딩된다. 즉, `this`는 해당 익명 클래스 구현체로, 1-A에서는 Object 클래스의 `toString()`으로 바인딩된다.

```java
// 코드 1-B. inner class and
class Hello {
  public Runnable r = new Runnable() {
      public void run() {
        // Hello's custom toString() 출력
        System.out.println(Hello.this);
        System.out.println(Hello.this.toString());
      }
    };

  public String toString() {
    return "Hello's custom toString()";
  }
}
```

반면, 1-B에서는 `this`가 외부 클래스 `Hello`의 인스턴스를 참조하므로, `Hello` 클래스 내에서 정의된 `toString()` 메서드를 호출한 결과값이 출력된다. 그러나 이 코드는 반직관적이다. 특히 (1) `Hello` 클래스 내부에서 (2) `this`에 적절한 스코프를 할당하기 위해 (3) `Hello.this`를 이용해야한다는 점에서 그렇다.

```java
// 코드 2-A.
class Hello {
  public Runnable r = () -> {
      System.out.println(this);
      System.out.println(toString());
    };

  public String toString() {
    return "Hello's custom toString()";
  }
}
```

2-A에서는 이전과 달리 람다식이 쓰였다. 이 람다식에서의 `this`는 자기 자신이 아니라 외부 인스턴스를 지시한다. 클로저(closure)라고도 불리는 변수 캡쳐(variable capture)는 감싸는 스코프 외부에 있는 변수 참조를 에워싼다. 변수 캡처는 내부 클래스에서도 이루어지나, 오직 불변 (final) 변수만 참조할 수 있다. 람다는 사실 상 불변이기에(effectively final) 수정되지 않는 변수에 한해서 이루어진다. 물론, 람다 식이 실행된 이후에는 수정되어도 된다.

### 나가며

추가적으로, [자바와 코틀린에서의 클로저를 잘 설명한 글](https://jaeyeong951.medium.com/java-%ED%81%B4%EB%A1%9C%EC%A0%80-vs-kotlin-%ED%81%B4%EB%A1%9C%EC%A0%80-c6c12da97f94)이 있어 첨부한다.

### References

https://www.oracle.com/technical-resources/articles/java/architect-lambdas-part1.html

https://en.wikipedia.org/wiki/Closure_(computer_programming)
