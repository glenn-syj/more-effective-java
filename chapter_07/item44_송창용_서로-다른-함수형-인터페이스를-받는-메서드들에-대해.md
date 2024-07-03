## 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들에 대해

### 정리

**내용 요약**

자바 표준 라이브러리에 이미 존재하는 함수형 인터페이스는 직접 구현할 필요가 없다.

java.util.function 패키지에는 총 43개의 인터페이스가 존재하는데, 대표적인 6가지의 인터페이스가 존재한다.

+ 인수가 하나인 UnaryOperator 인터페이스
+ 인수가 두 개인 BinaryOperator 인터페이스
+ 인수 하나를 받아 boolean을 반환하는 Predicate 인터페이스
+ 인수와 반환 타입이 다른 Function 인터페이스
+ 인수를 받지 않고 값을 반환하는 Supplier 인터페이스
+ 인수를 받지만 반환값은 없는 Consumer 인터페이스

위의 여섯 가지 인터페이스들은 해당 기본 타입에 따라서 변형된 형태가 존재한다.

예를 들어, int 타입을 받는 Predicate는 IntPredicate로 변형할 수 있다.

하지만 Function 인터페이스의 경우, 입력과 반환 타입이 다르기 때문에 다른 인터페이스와는 다른 형태로 변형된다.

예를 들면, long 타입의 데이터를 받아 int 타입의 데이터를 반환한다면 SourtToReturn 형식을 이용하여 인터페이스 앞에 붙여서 LongToIntFunction이 된다.

만일, 입력이 객체참조이고 결과가 기본 타입이라면 ToResult 형식을 적용하여 `ToLongFunction<int[]>`처럼 사용할 수 있다.


---


그러나 원하는 표준 인터페이스가 존재하지 않는다면, 직접 작성할 수밖에 없다. 그런데 원하는 인터페이스와 구조적으로 똑같은 표준 함수형 인터페이스가 존재하더라도 직접 작성해야만 할 때가 있다.

Comparator<T> 인터페이스는 구조적으로 ToIntBiFunction<T, U>와 동일하다. 그러나 ToIntBiFunction<T, U>가 존재하더라도 Comparator<T> 를 사용해야할 이유가 있다.

첫 째, Comparator의 이름이 용도를 더 잘 설명하기 때문이다.
둘 째, 반드시 지켜야할 구약이 구현할 때 포함되어있기 때문이다.
셋 째, 유용한 디폴드 메서드들을 가지고 있기 때문이다.

따라서 위와 같은 세 가지 이유를 참고하여 하나 이상이 해당될 때에는 직접 함수형 인터페이스를 작성해야만 한다.

또한, 직접 함수형 인터페이스를 작성할 때에는 해당 인터페이스에 @FunctionalInterface 애너테이션을 적용해야한다.

@FunctionalInterface 애너테이션은 @Override 처럼 프로그래머의 의도를 명시해준다.
@FunctionalInterface는 적용된 인터페이스가 람다용으로 설계되었음을 알려준다.
또한, @FunctionalInterface가 적용된 인터페이스는 추상 메서드를 오직 하나만 가지고 있어야 컴파일이 이루어지며, 누군가가 실수로 메서드를 추가하지 못하게 막아준다.


하지만, 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들에 대해, 해당 메서드들을 다중 정의해서는 안 된다.

클라이언트에게 혼동을 일으켜 문제를 야기할 가능성이 있기 때문이다.

예를 들어, ExcecutorService의 Submit 메서드는 Callable<T>를 받는 것과 Runnable을 받는 것을 다중정의했다. 그래서 올바른 메서드를 알려주기 위해 형변환해야 할 때가 자주 발생한다. (Item 52 참고.)



### 심화 탐구


**출발점**


해당 아이템 마지막 부분의 '서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들에 대해, 해당 메서드들을 다중 정의해서는 안 된다.' 부분을 이해하지 못하였다.

따라서, 예시로 나온 ExcecutorService의 Submit 메서드가 Callable<T>를 받는 것과 Runnable을 받는 것에 대해 다중정의 한 것을 분석하여 해당 문구가 어떤 의미인지 정리해보겠다.


**설명**

#### ExcecutorService에 대해

ExcecutorService란 비동기 작업의 진행을 추적할 수 있는 Future 객체를 반환해주는 인터페이스이다.

해당 인터페이스의 submit() 메서드는 Future 객체를 반환하여 작업의 실행 취소 및 완료 대기를 할 수 있다.

해당 submit 메서드는 두 가지 타입의 인수를 받을 수 있는데 하나는 **Callable<T>** 이고 다른 하나는 **Runnable**이다.

위에서 '서로 다른 함수형 인터페이스'에 해당하는 것이 이 'Callable<T>'와 'Runnable'을 일컫는 것이다.

실제로 에러가 발생할 코드 예시는 다음과 같다.


```java
package org.example;


import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        // 1. Thread 생성자 호출
        new Thread(System.out::println).start();

        // 2. ExecutorService의 submit 메서드 호출
        ExecutorService exec = Executors.newSingleThreadExecutor();
        exec.submit(System.out::println); // compile error
    }
}
```

위 코드에서의 compile error는 submit 메서드의 다중정의로 인하여, 컴파일러가 어떤 메서드를 호출해야할지 판단하지 못함으로써 발생하는 것이다.


따라서, 컴파일러가 어떤 메서드를 호출해야할지 혼동하지 않도록 아래와 같이 명시적으로 형변환을 해주면 정상적으로 작동한다.

[chatGPT 코트를 참고하였다.]
```java
package org.example;


import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        // 1. Thread 생성자 호출
        new Thread(System.out::println).start();

        // 2. ExecutorService의 submit 메서드 호출
        ExecutorService exec = Executors.newSingleThreadExecutor();
        // 명시적으로 Callable로 형변환
        exec.submit((Callable<Void>) () -> {
            System.out.println("Callable task executed");
            return null;
        });
    }
}
```


### 어려운 점

ExecutorService의 submit 메서드에 대해서 컴파일러가 혼동하지 않도록 명시적인 형변환을 해주는 예시를 찾거나 직접 만들어 보고 싶었으나,

쉽지 않았기에 chatGPT를 통하여 명시적인 형변환을 한 예시를 찾을 수 있었다.

---


Reference:

https://www.baeldung.com/java-execute-vs-submit-executor-service

https://docs.oracle.com/javase%2F8%2Fdocs%2Fapi%2F%2F/java/util/concurrent/ExecutorService.html

https://jaeseo0519.tistory.com/233

