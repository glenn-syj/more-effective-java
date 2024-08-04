## 부정확한 메서드 참조와 적용 가능성 테스트에 대해

### 정리

**내용 요약**

재정의 메서드는 어떤 메서드가 실행될 것인지 확실하게 알 수 있다.

반면, 다중정의한 메서드는 어떠한 메서드가 실행될 것인지 사용자가 헷갈리기 쉬운데다, 예상치 못한 에러가 발생할 수 있다.

다중정의 메서드가 혼동을 주지 않기 위해서는 아래의 법칙을 지켜야 한다.

1. 매개변수의 갯수가 같은 다중정의 메서드는 만들지 않는다.
2. 가변인수를 사용하는 메서드는 다중정의를 해서는 안된다. (item53 참고)

이러한 법칙을 지키기 어렵다면 다중정의 메서드를 생성하지 않고 메서드의 이름을 다르게 정의하는 것도 좋은 선택이다.

ObjectOutputStream 클래스의 write 메서드는 다중정의 메서드를 생성하지 않고 readBoolean, readInt, readLong처럼 다른 이름의 메서드들을 생성하였다.

그러나 생성자의 경우는 이름을 다르게 지을 수 없기 때문에, 두 번째 생성자부터는 `무조건 다중정의가 된다`.

다행히 정적 팩터리(item1 참고)를 대안으로 사용할 수 있으며, 생성자의 경우 '재정의'가 불가능하므로 재정의와 다중정의가 혼용되지 않아 혼란을 야기할 가능성이 적어진다.

그러나 생성자들이 같은 갯수의 매개변수를 사용한다면 사용자에게 혼동을 주고 예상치 못한 에러를 초래할 수 있으므로 그에 대한 안전책을 마련해야 한다.

이러한 위험성을 방지해줄 안전책은 사용하는 매개변수 중 하나 이상을 `근본적으로 다르게` 설정하는 것이다.

`근본적으로 다른` 이라는 말은 두 변수의 타입이 서로의 타입으로 형변환이 불가능한 것을 말한다.

예를 들어, ArrayList의 int를 받는 생성자와 Collection을 받는 생성자는 int와 Collection타입이 서로의 타입으로 형변환이 불가능하므로 `근본적으로 다른` 관계가 된다.

'자바 4'까지는 모든 기본타입이 모든 참조타입과 `근본적으로 달랐다`.

그러나 '자바 5'부터 제네릭, 오토박싱이 생긴 결과 List 인터페이스가 취약해졌다.

심지어 자바 8부터 등장한 람다와 메서드 참조 역시 다중정의에 대한 혼란을 가중시켰다.

다음 코드는 이전 아이템에서도 한 번 다룬 적이 있는 ExecutorService의 submit 메서드에 관한 것이다.


```java
// 1번
new Thread(System.out::println).start();

// 2번
ExecutorService exe = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

위 코드에서 1번은 정상적으로 작동하지만 2번은 컴파일 오류가 발생한다.

submit 다중정의 메서드에는 Runnable을 매개변수로 받는 메서드와 Callable을 매개변수로 받는 메서드가 존재하여 혼동을 주기 때문이다.

그러나 모든 println은 void를 반환하기 때문에 반환값이 있는 Callable과 Runnable을 헷갈리는 이유는 무엇일까.

그 이유는 System.out.println은 '부정확한 메서드 참조'이기 때문이다. 또한 '암시적 타입 람다식'이나 '부정확한 메서드 참조' 같은 '인수 표현식'은 목표 타입이 선택되기 전에는 그 '의미'가 정해지지 않기 때문에 적용성 테스트 때 무시되기 때문이다.

즉, 다중정의된 메서드가 함수형 인터페이스를 인수로 받을 때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생기게 된다는 것이다.




### 심화 탐구


**출발점**

본문에서 ExecutorService의 submit 메서드가 System.out.println을 매개변수로 받을 때 발생하는 오류의 원인을 설명하는 부분을 이해할 수 없었다.

이전에도 다른 아이템에서 해당 문제를 심화 탐구의 주제로 삼았으나 끝내 완벽히 이해하지 못했으나, item52에서 구체적인 원인을 설명하고 있기 때문에 세세하게 분석하여 원인을 파악해보겠다.


**설명**

#### '부정확한 메서드 참조'에 대해

먼저 System.out::println 메서드가 부정확한 메서드 참조라는 것에 대해 이해하여보겠다.

System.out::println은 메서드 참조 구문이며, 이러한 형식을 통해 '특정 메서드를 참조' 하여 나중에 실행되도록 할 수 있다.

이러한 형식은 ReferenceType :: [TypeArguments] Identifier 로 표현할 수 있다.

System.out::println에 적용하여 파악해보면, ReferencType은 System.out을 가지고 있는 PrintStream 객체를 가리키고 있는 것이며, 

Identifier는 println이라는 메서드명을 식별자로서 가리키고 있는 것이다.

ReferenceType :: [TypeArguments] Identifier 형식으로 표현되는 메서드 참조 표현식은 다음 조건을 '모두' 만족해야 '정확'하다고 할 수 있다.

1. 메서드 참조 표현식의 RefereceType은 원시 타입을 나타내지 않는다.

-> System.out::println에서 System.out이 가리키는 PrintStream 타입은 원시 타입이 아니므로 1번을 만족하고 있다.

2. 메서드 참조 표현식이 나타나는 클래스나 인터페이스에 접근 가능한 이름 'Identifier'를 가진 `정확히 하나의 멤버 메서드`를 가지고 있어야 한다.

-> PrintStream 클래스에 println 함수명을 갖는 여러 메서드가 존재한다. println(), println(int), println(char).... 따라서 2번을 만족하지 못하고 있다.

-> 모든 조건을 만족해야 하기 때문에 2번을 만족하지 못한 시점에서 System.out.println 메서드는 '부정확한 메서드 참조'임이 확실해졌다.

3. 이 메서드는 가변 인수 메서드가 아니어야 한다.

-> 가변 인수 메서드란 인수의 개수가 고정되지 않은 메서드를 말한다.

-> ex) `printNumber(int... numbers) {}` 같은 가변 인수 메서드는 int 타입의 매개변수를 고정되지 않은 여러개 가져와 사용할 수 있다.

-> System.out::println은 가변 인수 메서드는 아니다.

4. 이 메서드가 제네릭 메서드인 경우, 메서드 참조 표현식은 TypeArgument를 제공해야 한다.

-> 이것은 System.out::println 메서드 참조 표현식은 제네릭 메서드가 아니기 때문에 해당되지 않지만,
 이전 FickleBoBo님이 발행한 이슈에 있던 예제가 제네릭 메서드이지만 TypeArgument를 제공하지 않았기에 '부정확한 메서드 참조'가 되었다.

```java
static <V> V myPrint5(){
    System.out.println("5");
    return null;

exec.submit(Main::myPrint5);                        // compile error(ambiguous)
exec.submit((Runnable) Main::myPrint5);      // 5
exec.submit((Callable<?>) Main::myPrint5);  // 5
}
```


즉, System.out::println 메서드 참조 표현식은 1, 3, 4번은 만족하고 있지만 2번은 만족하지 못하므로 '부정확한 메서드 참조' 형식이다.

Java Oracle에서 제공하는 명세서를 보면 '부정확한 메서드 참조' 형식에 대해 이런 설명을 하고 있다.

암시적으로 형식이 지정된 람다 표현식(§15.27.1) 또는 부정확한 메서드 참조(§15.13.1)를 포함하는 특정 인수 표현식은 적용 가능성 테스트에서 무시됩니다. 이는 목표 타입이 선택될 때까지 그 의미를 결정할 수 없기 때문입니다.

여기서 말하는 '적용 가능성 테스트'란 메서드 오버로드를 해결하기 위한 것으로, 주어진 인수들이 특정 메서드 시그니처(메서드명, 매개변수 집합)와 호환되는지를 판단하는 작업이다.

즉, System.out::println은 '부정확한 메서드 참조' 형식이기 때문에 메서드 오버로드 해결하는 과정에서 무시되어 ambiguous 에러를 발생시키는 것이다.

추가적으로 '목표 타입'이란 익명함수가 사용되는 곳에 따라 결정되는 타입을 '목표 타입'이라 한다.

메서드 오버로딩을 해결하는 과정에 있어서, 매개변수로 사용하는 함수형 인터페이스의 목표 타입에 대한 정보가 우선적으로 수반되어야 하지만 그렇지 않기 때문에 오류가 발생하는 것이었다.

```java
package item44;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        // 1. Thread 생성자 호출
        new Thread(System.out::println).start();

        // 2. ExecutorService의 submit 메서드 호출
        ExecutorService exec = Executors.newSingleThreadExecutor();

        exec.submit(System.out::println);                        // compile error(ambiguous)
        exec.submit((Runnable) System.out::println);
        exec.submit((Callable<?>) System.out::println);  // compile error(bad return type)
    }
}
```

따라서 이 예제의 경우, System.out::println은 '부정확한 메서드 참조' 형식으로 목표 타입에 대한 정보가 존재하지 않기 때문에 메서드 오버로딩을 해결하는 적용 가능성 테스트에서 무시되어 ambiguous 에러가 발생한다.

`exec.submit((Runnable) System.out::println)`의 경우 목표 타입을 (Runnable) 로 명시해주고 있기 때문에 적용 가능성 테스트에서 무시되지 않아 에러가 발생하지 않았던 것이다.

추가로 FickleBoBo님 께서 올린 예제를 보면

```java
package item44;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {

    static void myPrint1(){
        System.out.println("1");
    }

    static String myPrint2(){
        System.out.println("2");
        return "2";
    }

    static Object myPrint3(){
        System.out.println("3");
        return 3;
    }

    static Void myPrint4(){
        System.out.println("4");
        return null;
    }

    static <V> V myPrint5(){
        System.out.println("5");
        return null;
    }

    public static void main(String[] args) {
        // 1. Thread 생성자 호출
        new Thread(System.out::println).start();

        // 2. ExecutorService의 submit 메서드 호출
        ExecutorService exec = Executors.newSingleThreadExecutor();

        exec.submit(Main::myPrint1);                       // 1
        exec.submit((Runnable) Main::myPrint1);     // 1
        exec.submit((Callable<?>) Main::myPrint1);  // compile error(bad return type)

        exec.submit(Main::myPrint2);                        // 2
        exec.submit((Runnable) Main::myPrint2);      // 2
        exec.submit((Callable<?>) Main::myPrint2);  // 2

        exec.submit(Main::myPrint3);                        // 3
        exec.submit((Runnable) Main::myPrint3);      // 3
        exec.submit((Callable<?>) Main::myPrint3);  // 3

        exec.submit(Main::myPrint4);                        // 4
        exec.submit((Runnable) Main::myPrint4);      // 4
        exec.submit((Callable<?>) Main::myPrint4);  // 4
        
        exec.submit(Main::myPrint5);                        // compile error(ambiguous)
        exec.submit((Runnable) Main::myPrint5);      // 5
        exec.submit((Callable<?>) Main::myPrint5);  // 5
    }
}
```

각 메서드는 이름으로 하나의 메서드만을 특정 지을 수 있기 때문에 System.out:println과는 다르게 `정확한 메서드 참조` 형식 조건 중 2번을 만족시킨다.

그러나 예제의 마지막 ambiguous 에러를 일으키는 부분을 보면 해당 메서드는 V로 제네릭 타입을 사용하고 있다.

해당 메서드는 따로 제네릭을 특정 지을 수 있는 TypeArgument를 제공하지 않고 있기 때문에 `4. 이 메서드가 제네릭 메서드인 경우, 메서드 참조 표현식은 TypeArgument를 제공해야 한다.` 조건을 만족시키지 못하고 있다.

따라서 해당 메서드 또한 적용 가능성 테스트에서 무시되어 ambiguous 에러를 일으킨 것이다.


---


Reference:

https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.13.1

https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.12.2

