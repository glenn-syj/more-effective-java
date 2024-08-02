## parallel 메서드에 대해서

### 정리

**내용 요약**

자바에서 동시성 프로그래밍 기능을 점점 더 많이 지원해주고 있지만, 이를 **올바르게 사용하는 것**은 어려운 일이다.

동시성 프로그래밍 기능을 올바르게 사용하지 않으면, 스트림 라이브러리가 파이프라인을 병렬화하는 방법을 찾아내지 못하여 **성능을 떨어뜨리며**, **제대로된 결과를 출력하지 못한다.**

대체로 **데이터를 원하는 크기로 쉽게 나눌 수 있는** 자료구조인 ArrayList, HashMap, HashSet, ConcurrentHashMAp의 인스턴스이거나 배열, int 범위, long 범위가 스트림의 소스라면 병렬화의 효과가 좋다.

또한 이웃한 원소들이 메모리에 저장되어 있는 거리가 가까운 정도를 말하는 **참조 지역성이 좋다면** 병렬화의 효과가 좋다.

거기에다, 스트림 파이프라인의 종단 연산 동작 방식 또한 병렬 수행 효율에 영향을 준다.

**종단 연산**에서 수행하는 작업량이 **전체 작업에서 많은 비중을 차지하면서 순차적**이라면 병렬 수행의 효과는 제한된다.

종단 연산 중 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업인 '**축소**'가 병렬화에 가장 적합하다.

**조건에 맞으면 바로 반환하는 메서드** 또한 병렬화에 적합하다.

반면 가변 축소를 수행하는 메서드는 **컬렉션들을 합치는 부담**이 존재하기 때문에 병렬화에 적합하지 않다.

잘못 병렬화하게 되는 이유로는 파이프라인이 사용하는 mapper, filters 혹은 프로그래머가 제공하는 다른 객체가 '**규약을 지키지 않는 것**'이 있다.

규약은 연산에 건네지는 함수는 반드시 **결합 법칙을 만족하고, 간섭받지 않고, 상태를 갖지 않아야 한다는 것**을 말한다.

이러한 병렬화 효과를 얻기 위한 여러 조건에도 불구하고, 스트림을 병렬화하여 **성능을 개선할 수 있는 상황은 많지 않다.**

하지만 조건이 잘 갖춰진다면 **parallel 메서드 호출 하나로 큰 성능 향상을 불러올 수도** 있는 만큼 병렬화하는 방법을 잘 알아두어야 한다.



### 심화 탐구


**출발점**


본문에서 parallel 메서드를 통해 프로세스 코어 수에 비례하는 성능 향상을 꾀할 수도 있으며 머신러닝 및 데이터 처리 작업에서 큰 성능 향상을 불러일으킬 수 있다고 하였다.

parallel 메서드를 이용하여 앞으로 있을 프로젝트와 작업들의 성능 향상 방법을 숙지하기 위해 parallel 메서드에 대하여 정리해보겠다.



**설명**

#### BaseStream 인터페이스에 대해

이전, #184 이슈에서 parallel과 parallelSteam 메서드의 차이점을 정리하면서 언급했듯이.

parallel 메서드는 Java 8 버전에 등장한 BaseSteram 인터페이스의 메서드이다.

먼저 parallel 메서드를 갖고 있는 BaseStream 인터페이스에 대해 알아야 parallel 메서드를 파악할 수 있을 것이다.

BaseStream은 컬렉션의 요소를 하나씩 참조하여 람다식으로 처리할 수 있는 반복자 기능을 갖는 스트림 인터페이스이다.

이러한 BaseStream 또한 여러 하위 인터페이스로 나뉘어지는데, Stream<T>, IntStream, DoubleStream, LongStream이 있다.

위의 하위 네 가지 인터페이스들은 각각 객체 요소 및 int, double, long 타입의 요소를 처리할 때 사용되는 스트림 인터페이스이다.

BaseStream 인터페이스에서 정의된 메서드들은 하위 인터페이스들이 공통으로 사용하기 위한 것들이다.

해당 공통 메서드들에는 iterator, spliterator, isParallel, sequential, parallel, unordered, onClose, close가 존재한다.


이 중 parallel 메서드에 대해 알아보자.

`Returns an equivalent stream that is parallel. May return itself, either because the stream was already parallel, or because the underlying stream state was modified to be parallel. This is an intermediate operation.`

oracle에 작성된 parallel에 대한 설명이다.

이를 번역해보면, 다음과 같다.

parallel 메서드는 등가 병렬 스트림을 반환한다. 해당 메서드는 스트림이 이미 병렬 상태이거나, 또는 기본 스트림 상태가 병렬로 수정되었을 경우 자기 자신을 반환할 수 있다.
parallel 메서드는 중간 연산을 위한 역할에 쓰인다.

parallel 메서드를 사용하는 예제 코드를 살펴보자.

우선 병렬 스트림이 아닌 순차 스트림으로 요소를 처리하는 코드를 보여주겠다.

```java

List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4);

listOfNumbers.stream().forEach(number ->
    System.out.println(number + " " + Thread.currentThread().getName())
);

```

위 코드에서 stream이 생성되었지만 병렬로 변환되지 않았기 때문에 각 요소에 대한 처리를 순차적으로 수행한다.

출력 결과는 다음과 같다.

```
1 main
2 main
3 main
4 main
```

순차 스트림을 parallel 메서드를 이용하여 병렬 스트림으로 바꿔보도록 하겠다.

```java
List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4);
    listOfNumbers.stream().parallel().forEach(number ->
        System.out.println(number + " " + Thread.currentThread().getName())
);
```

병렬 스트림 코드에서는 여러 스레드가 각 요소를 병렬로 처리하게 된다.

출력 결과는 다음과 같다.

```
첫 번째 시도

1 ForkJoinPool.commonPool-worker-2
3 main
4 ForkJoinPool.commonPool-worker-3
2 ForkJoinPool.commonPool-worker-1

```

```
두 번째 시도

4 ForkJoinPool.commonPool-worker-3
2 ForkJoinPool.commonPool-worker-1
3 main
1 ForkJoinPool.commonPool-worker-2

```

```
세 번째 시도

2 ForkJoinPool.commonPool-worker-1
4 ForkJoinPool.commonPool-worker-3
1 ForkJoinPool.commonPool-worker-2
3 main

```

각 요소들에 대하여 병렬로 처리가 이루어지고 있는데, 자세히 살펴보면 특정 요소에 대해서는 같은 쓰레드가 일관적으로 처리를 수행하지만, 그 순서는 계속 달라지고 있다.

요소의 순서에 따라 배정되는 쓰레드는 일정하고 작업을 시작하는 시점도 동일하지만, 쓰레드별로 작업을 마치는 속도는 차이가 있기 때문에 코드를 실행할 때마다 출력 순서가 달라지는 것으로 추측된다.

참고로 이전 이슈에서 parallel()과 parallelStream()의 차이점에 대해 설명하였는데, 위의 parallel 메서드를 사용한 코드를 보면 앞에 stream() 메서드를 통해 먼저 스트림을 생성해주고 있다.

그러나 parallelStream() 메서드를 사용한다면, stream을 직접 생성해주지 않아도 되어 stream() 메서드를 생략할 수 있다.

```java
List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4);
    listOfNumbers.parallelStream().forEach(number ->
        System.out.println(number + " " + Thread.currentThread().getName())
);
```


---


Reference:

https://blogs.oracle.com/javamagazine/post/java-parallel-streams-performance-benchmark

https://www.baeldung.com/java-when-to-use-parallel-stream