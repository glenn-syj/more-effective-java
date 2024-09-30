## BigDecimal의 계산 성능

---

### 정리

**내용 요약**

#### float와 double 타입의 부동소수점 오차.

 float와 double 타입은 과학과 공학계산용으로 설계된 타입이다. 그렇기에 정확한 값보다는 근사치를 빠르게 계산하기 위한 목적으로 설계되었다.

 즉, float와 double 타입은 정확한 값을 얻기 위한 계산에는 적합하지 않으며, 특히 정확한 값을 요구하는 금융 분야에서는 사용해서는 안되는 타입이다.

 따라서, 금융 분야 또는 정확한 값을 도출해야하는 상황에서는 float와 double 대신 BigDecimal, int, long 타입을 사용해야만 한다.

 아래의 간단한 예시를 통해 float와 double을 사용하면 부정확한 값이 도출될 수 있음을 알 수 있다.

 ```java
 System.out.println(1.03 - 0.42);

 -> 0.6100000000000001
 ```

 일반적으로 0.61 수치가 나와야 하지만 float 또는 double 타입의 연산을 사용하면 이러한 오차가 발생하고 만다. 따라서 정확한 값을 도출하기 위해서는 float와 doubel 타입의 사용을 지양해야한다.

 그러나 BigDecimal과 int, long 타입 또한 사용하는 것에 있어서 몇 가지 단점을 감수해야만 한다.

 BigDecimal은 빠르게 계산을 도출하기 위한 float와 double에 비해 속도가 느리다.

 반대로 int와 long 타입은 BigDecimal 보다 우수한 속도를 보이지만 상대적으로 값의 범위가 제한되고 소수점을 직접 관리해야한다는 불편함이 존재한다.

---

### 심화 탐구

float와 double 타입을 사용하면 부동소수점 오차가 발생한다는 것은 알고 있었지만, 두 타입에 대한 대안책인 BigDecimal에 대해서는 자주 사용해보지 않았기 때문에 이 기회에 BigDecimal 타입에 대해 알아보려 한다.

따라서 BigDecimal의 여러 반올림 메서드와 float와 double과의 성능 차이를 조사해보겠다.

---

**출발점**

float와 double 타입을 사용하면 부동소수점 오차가 발생한다는 것은 알고 있었지만, 두 타입에 대한 대안책인 BigDecimal에 대해서는 자주 사용해보지 않았기 때문에 이 기회에 BigDecimal 타입에 대해 알아보려 한다.

따라서 BigDecimal의 여러 반올림 메서드와 float와 double과의 성능 차이를 조사해보겠다.

---

**설명**

`Immutable, arbitrary-precision signed decimal numbers`

BigDecimal이란 아무리 **큰 수라도 정확하게 표현할 수 있는** (arbitary-precision) 객체를 말한다.

BigDecimal은 8가지의 반올림 방법을 지정할 수 있는데 정확한 반올림 방식을 지정하지 않으면 MathContext 객체가 정확한 방식의 처리를 할 수 없기 때문에 예외가 발생한다.

Bigdecimal의 반올림 방식은 열거형의 값을 이용하여 표시된다.

+ ROUND_CEILING : 양수의 경우 올림하고, 음수의 경우 버림. 즉, 결과적으로 항상 더 큰 값으로 반올림된다.
+ ROUND_FLOOR : 양수의 경우 버림하고, 음수의 경우 올림. 즉, 결과적으로 항상 더 작은 값으로 반올림된다.
+ ROUND_UP : 소수점 이하를 무조건 올림 (-1.111 -> -1.12)
+ ROUND_DOWN : 소수점 이하를 무조건 버림
+ ROUND_HALF_UP : 소수점 이하 자릿수가 5이상이면 올림, 그보다 작으면 버림.
+ ROUND_HALF_DOWN : 소수점 이하 자릿수가 5초과면 올림, 5이하면 버림.
+ ROUND_HALF_EVEN : 소수점 이하 자릿수가 정확히 5일 때, 가장 가까운 짝수로 반올림 (1.15 -> 1.2, 1.25 -> 1.2)
+ ROUND_UNNECESSARY : 반올림하지 않음.

BigDeciaml은 객체 기반이며, 정확한 계산을 위해 Java 코드에서 구현된 연산 방식을 사용하기 때문에 기본형이며 하드웨어 수준에서 연산되는 float 및 double 보다 느린 성능을 보인다.

실제로 코드를 통해 BigDecimal과 float 그리고 double의 성능을 측정해보았다.

```java
import java.math.BigDecimal;

public class Main {
    public static void main(String[] args) {
        // 반복 횟수 설정
        int iterations = 1000000;

        // double 타입 연산 성능 측정
        double doubleResult = 0;
        double doubleStartTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            doubleResult += 100000000L * 2L;
        }
        long doubleEndTime = System.nanoTime();
        System.out.println("double 연산 결과: " + doubleResult);
        System.out.println("double 연산 소요 시간: " + (doubleEndTime - doubleStartTime) + " ns");

        // float 타입 연산 성능 측정
        float floatResult = 0;
        float floatStartTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            floatResult += (float) 100000000 * (float) 2;
        }
        long floatEndTime = System.nanoTime();
        System.out.println("float 연산 결과: " + floatResult);
        System.out.println("float 연산 소요 시간: " + (floatEndTime - floatStartTime) + " ns");

        // BigDecimal 타입 연산 성능 측정
        BigDecimal bigDecimalResult = BigDecimal.ZERO;
        BigDecimal bigDecimalOperand = new BigDecimal("100000000");
        long bigDecimalStartTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            bigDecimalResult = bigDecimalResult.add(bigDecimalOperand.multiply(BigDecimal.valueOf(2)));
        }
        long bigDecimalEndTime = System.nanoTime();
        System.out.println("BigDecimal 연산 결과: " + bigDecimalResult);
        System.out.println("BigDecimal 연산 소요 시간: " + (bigDecimalEndTime - bigDecimalStartTime) + " ns");
    }
}

```

```
double 연산 결과: 2.0E14
double 연산 소요 시간: 2489600.0 ns
float 연산 결과: 2.01031883E14
float 연산 소요 시간: 4194304.0 ns
BigDecimal 연산 결과: 200000000000000
BigDecimal 연산 소요 시간: 24029600 ns
```

nanoTime을 이용하여 성능을 측정한 결과 BigDecimal의 연산속도는 double의 대략 10배, float의 대략 5배로 나왔다.


---


Reference:

https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html#toString--