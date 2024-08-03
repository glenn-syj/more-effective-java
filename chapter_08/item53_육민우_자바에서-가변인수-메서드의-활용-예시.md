# 자바에서 가변인수 메서드의 활용 예시

## item53 - 가변인수는 신중히 사용하라

### 🔍 내용 요약

**인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수(varargs)가 반드시 필요하다.** 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 전달하는 과정 생기는데 이때 성능이 느려질 수 있으므로 가변인수를 사용할 때는 성능 문제까지 고려하자.

<br>

---

### 🧐 심화 탐구

이번 아이템의 내용 자체는 굉장히 간단했다.

메서드의 인수가 몇 개가 들어올지 모르는 상황에서 활용할 수 있는 가변인수의 개념을 소개했고, 평범한 매개변수와 가변인수로 나누어서 매개변수를 받는 활용을 통해 예외처리를 줄이고 코드를 간단하게 작성할 수 있는 테크닉을 소개했다. 이후 가변인수는 같은 크기의 배열을 만들고 초기화하는 방식으로 동작해서 성능이 느려질 수 있음을 언급했고 가변인수에 들어오는 인수의 개수 대부분이 몇 개 안될 것으로 예상되면 다중정의를 활용하는 패턴을 통해 성능 문제에 도움을 줄 수 있음을 알 수 있었다.

<br>

#### 1. printf

```java
public PrintStream printf(String format, Object ... args) {
    return format(format, args);
}
```

위 코드는 가변인수를 활용하는 대표적인 메서드인 `printf` 메서드로 메서드의 인수를 보면 `String format`에 포맷팅할 문자열을, `Object ...args`에 가변인수를 받아 이를 실제로 포맷팅할 `format` 메서드에 전달하고 있음을 짐작할 수 있다. `format` 메서드의 내부는 훨씬 더 복잡해서 가변인수를 사용한다는 점에 초점을 맞춰 간단하게 알아보았다.

<br>

#### 2. EnumSet

```java
public static <E extends Enum<E>> EnumSet<E> of(E e) {
    EnumSet<E> result = noneOf(e.getDeclaringClass());
    result.add(e);
    return result;
}


public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    return result;
}


public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    result.add(e3);
    return result;
}


public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    result.add(e3);
    result.add(e4);
    return result;
}


public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4,
                                                E e5)
{
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    result.add(e3);
    result.add(e4);
    result.add(e5);
    return result;
}


@SafeVarargs
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) {
    EnumSet<E> result = noneOf(first.getDeclaringClass());
    result.add(first);
    for (E e : rest)
        result.add(e);
    return result;
}
```

가변인수의 성능 문제와 관련해 소개한 패턴의 실제 예시로 소개한 EnumSet의 정적 팩터리의 경우 `of` 메서드를 말하는 것으로 보이는데 다음과 같이 인수 5개까지는 다중정의를 활용해서 처리하고 그 이상일 경우 가변인수를 통해 처리하는 것을 볼 수 있다. 이와 유사하게 `List.of` 메서드의 경우 인수 10개까지는 다중정의를 통해 처리하는 것을 볼 수 있었다. `@SafeVarargs` 어노테이션의 경우 이전의 제네릭 배열에서 형변환 경고(공변, 불공변)에 대한 어노테이션을 붙였던 것과 유사한 어노테이션으로 보이는데(가변인수는 배열로 처리되어서) 프로그래머가 형변환에 안전하게 설계함을 보장할 수 있을 경우 붙이면 좋을 것으로 보인다.

<br>

#### 3. main

```java
package item53;

public class Test {
    public static void main(String[] args) {
        for(String arg : args){
            System.out.println(arg);
        }
    }
}

// Hello
// world!
```

```java
package item53;

public class Test {
    public static void main(String... args) {
        for(String arg : args){
            System.out.println(arg);
        }
    }
}

// Hello
// world!
```

마지막은 main 메서드에 대한 예시로 다음과 같이 main 메서드의 String[] args를 String... args 처럼 가변인수로 받아도 문제없이 실행되며 다음과 같은 결과가 나왔다.
`args`에 인수를 넣는 방법으로 window 인텔리제이에서 상단바 -> Run -> Edit Configurations -> Add New Configuration -> Build and run에서 클래스(item53.Test)와 인수(Hello world!) 입력을 통해 설정했다.

main 메서드의 `String[] args`는 이처럼 가변인수로 보이며 가변인수처럼 동작하는 것으로 보였는데 배열로 되어있는 점이 의아해 ChatGPT와 스택오버플로우를 통해 알아보았는데 실제 동작은 `String... args`처럼 하지만 자바 컴파일러가 `String[] args`로 변환한다 정도의 답변을 얻을 수 있었다.

<br>

ref : https://docs.oracle.com/javase/8/docs/api/java/lang/SafeVarargs.html

ref : https://stackoverflow.com/questions/30949614/is-there-any-difference-between-string-args-and-string-args-in-java

ref : https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.1.4

---

### 🧠 고민해볼 점

가변인수 메서드의 경우 이를 사용하는 프로그래머가 해당 메서드가 가변인수 메서드임을 알아야 의미가 있을 것으로 보였다. 이런 점에서 정적 팩터리 메서드나 `printf`처럼 특수한 메서드들이 아니면 성능면에서 좋지 않은 가변인수 메서드를 굳이 활용할 필요가 없을 것 같다는 생각이 들었다.
