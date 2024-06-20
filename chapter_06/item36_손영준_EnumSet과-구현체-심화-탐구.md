## EnumSet과 구현체 심화 탐구

### 정리

비트필드는 각 비트를 이용해 여러 상수를 하나의 집합처럼 이용해, 집합 연산을 쉽게 이용할 수 있다. 하지만 비트 필드는 가독성이 좋지 않으며, 사전적으로 적절한 크기를 가지는 타입을 선택해 확장성이 좋지 않다.

EnumSet 추상 클래스는 Set 인터페이스를 구현하며 비트필드를 대체할 수 있다. EnumSet의 내부는 비트 벡터로 구현된 반면, 대규모 작업은 산술 연산을 이용해 구현되어 효율적이다. 즉, 비트필드와 같은 레거시 방식은 자바의 설계에서는 크게 사용할 필요가 없다. 

### 심화 탐구

**출발점**

EnumSet 클래스가 추상클래스라면, 그 구현체는   어떻게 구현되어 있을까?  이러한 세부 구현도 구현이지만, 먼저 간단히 EnumSet에 대해서 알아볼 필요가 있다. 특히, 해당 절에서 설명되지 않은 EnumSet의 추가적인 특성들을 살펴보겠다.

**설명**

1. 추가적인 특성

EnumSet은 한 단일 enum 타입에 해당하는 값들만 가질 수 있고, null 값은 허용하지 않는다. 또한, EnumSet은 멀티스레드로 동기화되지 않기에, 외부에서 동기화되어야 한다. 

앞서 EnumSet의 비트 벡터가 long을 이용한다는 설명을 들은 바 있다. 비트 벡터는 비트의 위치를 이용해 포함 관계를 표현한다. 즉, i번째 enum은 i번째 비트에 저장된다. 

2. 구현체

EnumSet의 구현체로는 RegularEnumSet과 JumboEnumSet이 있다.

RegularEnumSet은 long 배열을 그대로 가지며, enum의 요소가 64개 이하라면 단일 long 값을 이용해서 비트로 포함 관계를 표현할 수 있다.

```java

/**
 * Bit vector representation of this set.  The 2^k bit indicates the
 * presence of universe[k] in this set.
 */
private long elements = 0L;

```

RegularEnumSet에서 포함관계를 나타내는 elements는 64개 이하이기에, 하나의 long 타입으로 표현될 수 있다.

반면 JumboEnumSet은 여러 개의 long 배열을 이용해, 64개 이상의 enum 요소에 대한 포함관계를 비트로 표현할 수 있다. 

```java
/**
 * Bit vector representation of this set.  The ith bit of the jth
 * element of this array represents the  presence of universe[64*j +i]
 * in this set.
 */
private long elements[];

```

즉, long 배열을 이용해 enum 요소들을 나타낼 수 있다. 만약 67개의 enum 요소가 있다면, 64개까지는 elements[0]에 저장되고, 65번~67번은 elements[1]에 저장될 것이다.

### 나가며

추후 탐구할 주제로, RegularEnumSet과 JumboEnumSet의 메소드가 동작하는 방식을 구체적으로 살펴보아도 좋겠다.

### References

https://docs.oracle.com/javase%2F8%2Fdocs%2Fapi%2F%2F/java/util/EnumSet.html

https://www.baeldung.com/java-enumset