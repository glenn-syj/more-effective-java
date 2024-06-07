## 제네릭과 가변 인자에 대한 심화 탐구

### 정리

가변 인수와 제네릭을 함께 쓴다면, 타입 안전성이 침해될 수 있다. 저자는 이를 근거로 제네릭 varargs 배열 매개변수에 값을 저장하는 것이 안전하지 않다고 말한다. 그러나 타입 안전성과 유용함의 상충(trade-off)을 고려하자면, 유용함이 더욱 앞선다. 자바 7부터는 `@SafeVarargs` 애너테이션을 통해 작성자 측에서의 안전성 보장을 가능하게 하였다. 이 애너테이션을 쓰기 위한 조건으로, 타입 안전성은 (1) varargs 배열에 조작을 가하지 않고, (2) 외부에서 배열을 참조할 수 없다면 보장됨에 유의해야 한다.

또한, 제네릭 varargs 배열을 이용하기 위해 `toArray` 메서드를 이용하는 대신 List를 이요함으로써 타입 안전성을 고려해 메서드를 설계할 수도 있다. 메서드 호출 시에는 `List.of` 메서드를 함께 이용하면 된다. 여기에서는 컴파일러 측에서의 타입 안전성이라는 장점과 코드 가독성과 퍼포먼스 저하라는 단점이 상충한다.


### 심화 탐구

**출발점**

해당 절은 제네릭 가변 인자 배열(`T... args`)을 매개 변수로 이용할 때의 문제점을 중심으로 작성되었다. 제네릭은 익숙했으나, 가변 인자는 사실 주로 사용해보지 않아 낯설었다. 비록 `main` 메서드를 자동 생성했을 때에도 `public static void main(String[] args)`와 같은 방식으로 이용되었음에도 말이다. 그렇다면 가변 인자 배열은 간략히 무엇이고, 어떻게 작동하는 걸까?

**설명**

1. Varargs란 무엇인가

Varargs는 자바 5에 도입되었다. 즉, 자바 5 이전에는 여러 매개 변수가 들어오는 경우에 그 개수에 따라 다른 메서드를 작성해야 했다. 하지만 자바 5에서부터는 배열 형식으로 해당 데이터를 받아오게 되었다. 이번 절에서의 문제 상황 역시, 배열의 실체화 불가능성으로부터 온다.

Varargs는 오직 하나만 들어올 수 있고, 가장 마지막 파라미터로 들어와야 한다. 아래 코드들을 살펴보자.

```java
// https://www.baeldung.com/java-varargs
public String formatWithVarArgs(String... values) {
    // ...
}
```

위 코드는 당연하게도 규칙을 잘 지키고 있다. 인자가 아무 것도 들어오지 않은 상태도 오류를 일으키지 않는다.

```java
public String customWithVarArgs(long customerId, String... values) {
    // ...
}
```

위 메서드는 매개 변수 `customerId`에 해당하는 인자를 요구하고 있다. 만약 해당 인자가 들어오지 않는다면, 오류를 일으킨다.

```java
public String customWithVarArgsWrong(String... values, long customerId) {
    // ...
}
```

그러나 위 메서드는 컴파일 타임에 오류를 일으키므로, 실행되지 않음에 유의해야 한다.

2. 예시 코드를 통한 힙 오염 확인

```java
// based on: https://www.baeldung.com/java-varargs
public class Main {
	public static void main(String[] args) {
		String[] array = returnAsIs("One", "Two");
	}
	 
	static <T> T[] toArray(T... arguments) {
        System.out.println(arguments.getClass());
		return arguments;
	}
	
	static <T> T[] returnAsIs(T a, T b) {
        System.out.println(a.getClass());
		System.out.println(b.getClass());
		System.out.println(toArray(a, b));
	    return toArray(a, b);
	}
}
```

해당 절의 내용과 동일하게, 위 코드는 `ClassCastException` 오류를 부른다. 이는 다음 두 과정의 충돌에서 발생하는 오류다. 

- `toArray` 메소드에 a와 b가 넘겨지면서 `Object[]` 타입의 array를 생성한다.
- `String[]` 배열은 `Object[]` 배열의 하위 클래스가 아니므로, 형변환이 이루어지지 않는다. 

`returnAsIs(T a, T b)` 내 `a`는 String이고, `b`는 String이다. 하지만 `toArray` 내에서 받아온 `arguments`의 클래스는 `Object[]`다.

힙 오염도 이러한 특성에서 발생한다.

```java
public class Main {
	public static void main(String[] args) {
		Object[] array = returnAsIs("One", "Two");
	}
	 
	static <T> T[] toArray(T... arguments) {
		return arguments;
	}
	
	static <T> T[] returnAsIs(T a, T b) {
	    return toArray(a, b);
	}
}
```

그에 비해 array를 `Object[]`로 받는 경우에는 문제가 발생하지 않는다. `System.out.println()`을 통해 `arguments.getClass()`의 클래스 정보를 출력해봐도 `class [Ljava.lang.Object;`라는 결과값을 확인할 수 있다.

### 더 나아가기

```java
public class Main {
	public static void main(String[] args) {
		toArray("A", "B", "C");
        toArray(1, 2, 3);
        toArray(1, 2, "A");
        toArray("A", 1, new int[] {1, 2});
        toArray("A", 1, List.of(42));
	}
	 
	static <T> T[] toArray(T a, T b, T c) {
		System.out.println("not in proxy: " + toArrayProxy(a, b, c).getClass());
		return toArrayProxy(a, b);
	}
	
	static <T> T[] toArrayProxy(T... arguments) {
		System.out.println("in proxy: " + arguments.getClass());
		return arguments;
	}
}
```

위 코드의 결과는 당연하게도, `not in proxy: `에서든 `in proxy: `에서든 모두 `Object[]`에 해당하는 결과를 불렀다. 이는 당연히도, 앞서 확인한 결과다. 그러나 실험 중에, 제네릭 및 Varargs와 관련된 흥미로운 사실을 발견했다.

아래 코드들의 예시에서, varargs로 서로 다른 타입을 인자로 받는 경우에 제네릭은 어떻게 적용될까?

```java
public class Main {
	public static void main(String[] args) {
		toArray("A", "B", "C");
        toArray(1, 2, 3);
        toArray("A", 1, 2);
        toArray("A", 1, new int[] {1, 2});
        toArray("A", 1, List.of(42));
	}
	 
	static <T> T[] toArray(T... arguments) {
		System.out.println(arguments.getClass());
		return arguments;
	}
	
}
```

```
출력값

class [Ljava.lang.String;
class [Ljava.lang.Integer;
class [Ljava.io.Serializable;
class [Ljava.io.Serializable;
class [Ljava.lang.Object;
```

각각의 경우에, `getClass()`를 이용해 받아낸 Varargs 배열의 타입은 흥미롭다. 특히, `toArray("A", 1, 2)`와 `toArray("A", 1, new int[] {1, 2})`가 `Serializable[]`에 해당하는 클래스를 가지는 점이 그렇다. 이는 제네릭의 결정 과정에 대한 추가적인 조사가 필요하므로, 추후 질문 시 이슈를 통해 서술하겠습니다.

### References

https://www.baeldung.com/java-varargs
https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html