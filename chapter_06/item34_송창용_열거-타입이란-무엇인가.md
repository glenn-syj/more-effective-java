## 열거 타입이란 무엇인가

### 정리

**내용 요약**

`열거 타입`이란 일정 개수의 상수 값을 정의한 다음, '그 외의 값'은 허용하지 않는 타입이다.

사계절, 태양계의 행성, 카드게임의 카드 종류처럼 요소의 갯수가 정해져 있는 것이 좋은 예다.

<br>

java에 열거 타입이 존재하지 않았을 때는, '정수 상수를 하나씩 선언해서 사용'하는 '정수 열거 패턴'을 사용했다.

'정수 열거 패턴'은 열거 타입에 비해 여러 단점이 있다.

1. 타입 안전을 보장할 수 없다.
2. 표현력이 좋지 않다.
3. 컴파일하면 상수 값이 클라이언트 파일에 그대로 새겨지기 때문에 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 정상 작동한다.
4. 정수 상수 값을 문자열로 출력하기가 쉽지 않다.

'정수' 대신 '문자열' 상수를 사용하는 '문자열 열거 패턴' 또한 존재하지만 해당 패턴은 정수 열거 패턴보다 더욱 좋지 않다.

+ 경험이 부족한 프로그래머는 문자열 상수의 값을 하드코딩하게 될 것이며, 문자에 오타를 내도 곧바로 에러를 확인할 수 없기에 예기치 않은 순간에 에러를 맞닦뜨리게 된다.
+ 또한 문자열을 비교함으로써 숫자를 비교할 때보다 성능이 크게 저하된다.

`열거 패턴`의 여러 단점을 해결해주는 `열거 타입`이 이 떄 등장하게 된다.

```java
// 열거 타입의 기본적인 형태

public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Song { NAVEL, TEMPLE, BLOOD }
```

자바의 열거 타입은 특히 __완전한 형태의 클래스__ 라서 다른 언어의(EX. C언어) 열거 타입보다 훨씬 **강력하다**

자바의 열거 타입의 특성은 다음과 같다.

1. 열거 타입 자체가 클래스이다.
2. 상수는 각자 자신의 인스턴스를 *하나씩* 만 만들어 public static final 필드로 공개한다. (인스턴스 통제)
    + 따라서, `싱글턴 패턴`은 원소가 하나뿐인 열거타입이라 할 수 있고, `열거 타입`은 싱글턴을 일반화한 형태라고 할 수 있다.
3. 열거 타입은 타입 안정성을 제공한다.
4. 열거 타입에는 이름이 같은 상수를 추가로 삽입할 수 있다.
5. 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 된다.
6. 열거 타입의 toString 메서드는 적절한 문자열 형태로 요소들을 출력해준다.
7. 열거타입에는 임의의 메서드 혹은 필드, 그리고 인터페이스를 추가하거나 구현하도록 만들 수 있다.

---

이 때, 7번에 해당하는 특성에 의문을 가질 수 있다.

*'열거 타입에 메서드와 필드를 추가하는 행위는 왜 필요한 것일까?'*

예를들어, Apple이라는 열거 타입이 존재할 때, 추가로 과일의 색이나 과일의 이미지를 보여주고 싶은 경우 7번에 해당한다.

즉, 상수와 연관된 다른 데이터를 상수 자체에 내재시키고 싶을 때 사용하는 것이다.

해당 방식은 생성자를 통해 다음과 같이 구현할 수 있다.

```java
Planet (double mass, double radius){

    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / ( radius * radius );

}
```

---

열거 타입은 많은 장점을 가지고 있지만, *대부분의 경우* 열거 타입의 성능은 정수 상수와 별반 다르지 않다.

하지만 필요한 원소를 컴파일 타임에 전부 알 수 있다면 열거 타입을 사용하여야, 강력하게 타입 안정성을 확보할 수 있다.


### 심화 탐구


**출발점**

열거 타입에 대한 이론은 이해하였지만, 자주 사용하던 개념이 아니다보니 생소한 느낌에서 벗어나기 쉽지 않다.

따라서, 보통 열거타입은 어디서 사용되는지 찾아보고 다양한 활용방법을 정리하여 이후 다른 작업에서 작성해볼 수 있도록 해보겠다.

**설명**

#### 우아한테크코스 미션 로또

우아한테크코스에서 로또 기능을 구현할 것을 미션으로 제시하였고, 조건으로 `enum`을 사용할 것을 덧붙였습니다.

해당 미션을 통해 enum을 활용할 수 있는 방식을 정리해보겠습니다.

```java

public enum Rank {
    FIRST(6, false, 2_000_000_000, (count, bonus) -> count == 6),
    SECOND(5, true, 30_000_000, (count, bonus) -> count == 5 && bonus),
    THIRD(5, false, 1_500_000, (count, bonus) -> count == 5 && !bonus),
    FOURTH(4, false, 50_000, (count, bonus) -> count == 4),
    FIFTH(3, false, 5_000, (count, bonus) -> count == 3),
    NONE(0, false, 0, (count, bonus) -> count <= 2);

    private final int count;
    private final boolean bonus;
    private final long prize;
    private final BiFunction<Integer, Boolean, Boolean> expression;

    Rank(int count, boolean bonus, long prize, BiFunction<Integer, Boolean, Boolean> expression) {
        this.count = count;
        this.bonus = bonus;
        this.prize = prize;
        this.expression = expression;
    }

    ...

    public boolean win(int count, boolean bonus) {
        return expression.apply(count, bonus);
    }

    ...
}

```

해당 코드를 이해하기 위해서는 우선 BiFunction이라는 인터페이스에 대해 알아야합니다.

BiFunction은 enum 내에서 자주 사용되는 인터페이스므로 알아둔다면 enum을 활용할 때 도움이 될 것입니다.

BiFunction interface는 두 개 이상의 매개변수를 전달받아 작업을 수행한 이후 새로운 값을 반환해줍니다.

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);

    default <V> BiFunction<T, U, V> andThen(Function<? super R, extends V> after) {
    Objects.requireNonNull(after);
    return (T t, U u) -> after.apply(apply(t, u));
  }
}

```
BiFunction<T, U, R>에서의 T는 첫 번째 매개변수 타입을, U는 두 번째 매개변수 타입을 가리키며 R은 반환타입을 가리킵니다.

apply 메서드는 T와 U에 해당하는 두 개의 매개변수를 전달받아 특정 작업을 수행한 이후 값을 반환하는 역할을 합니다.

다음 코드를 보시면 BiFunction에 대해 이해가 수월할 것입니다.

```java
public static void main(String args[]) {
  BiFunction<Integer, Integer, String> biFunctionAdd =
          (num1, num2) ->  Integer.toString(num1 + num2);

  BiFunction<Integer, Integer, String> biFunctionMinus =
          (num1, num2) -> Integer.toString(num1 - num2);

  BiFunction<Integer, Integer, String> biFunctionMultiple =
          (num1, num2) -> Integer.toString(num1 * num2);

  System.out.println("100 + 50 = " + biFunctionAdd.apply(100, 50));
  System.out.println("100 - 50 = " + biFunctionMinus.apply(100, 50));
  System.out.println("100 * 50 = " + biFunctionMultiple.apply(100, 50));
}
```

apply 메서드를 통해 계산할 두 개의 Integer 타입의 매개변수를 전달하여 BiFunction에 정의된 작업을 수행한 이후 String 타입의 결과물을 반환합니다.


다시 우아한테크코스의 미션으로 돌아가보겠습니다.

```java

public enum Rank {
    FIRST(6, false, 2_000_000_000, (count, bonus) -> count == 6),
    SECOND(5, true, 30_000_000, (count, bonus) -> count == 5 && bonus),
    THIRD(5, false, 1_500_000, (count, bonus) -> count == 5 && !bonus),
    FOURTH(4, false, 50_000, (count, bonus) -> count == 4),
    FIFTH(3, false, 5_000, (count, bonus) -> count == 3),
    NONE(0, false, 0, (count, bonus) -> count <= 2);

    private final int count;
    private final boolean bonus;
    private final long prize;
    private final BiFunction<Integer, Boolean, Boolean> expression;

    Rank(int count, boolean bonus, long prize, BiFunction<Integer, Boolean, Boolean> expression) {
        this.count = count;
        this.bonus = bonus;
        this.prize = prize;
        this.expression = expression;
    }

    ...

    public boolean win(int count, boolean bonus) {
        return expression.apply(count, bonus);
    }

    ...
}

```

enum 내부에 선언된 FIRST ~ NONE은 모두 '일차하는 번호의 개수', 보너스 번호 필요 여부, 상금, 당첨 조건을 제시하고 있습니다.

'당첨 조건'은 SECOND를 살펴보면,

`(count, bonus) -> count == 5 && bonus` 로 작성이 되어있는데, 이는 5개의 번호가 일치해야 하며, 보너스 번호가 필요함을 명시하고 있습니다.

BiFunction인터페이스를 Integer와 Boolean 타입의 매개변수를 받으면 Boolean 타입의 데이터를 반환하는 expression 으로 선언해주고 있습니다.

이후, win 메서드를 통해 맞춘 갯수와 보너스 번호의 여부를 매개변수로 받아와 expression의 apply 메서드로 넘겨주면 조건에 일치하는 등급을 반환하게 됩니다.

---



### 어려운 점

enum을 활용하는 다른 예시도 찾았으나, 분량이 많고 지금 이해하기에는 어려운 내용이 많아 분석 후 정리하지 못하였다.

우아한 형제들 기술블로그에 작성된 내용으로 enum에 대해 더 파악하고 활용해보고 싶은 분이 있다면 방문하여 글을 읽어보는 것을 추천한다.

https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html

---


Reference:

https://developer-talk.tistory.com/716

https://woo-chang.tistory.com/51




