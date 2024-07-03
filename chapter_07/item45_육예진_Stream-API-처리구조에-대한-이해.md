## Stream API 처리구조에 대한 이해

### 정리

**내용 요약**

다량의 데이터 처리 작업을 돕기위해 추가된 Stream API이지만, 모든 데이터를 Stream으로 처리해야만 더 효율적인 코드는 아니다.  
Stream과 반복문을 필요에 따라 적절히 조합하여 사용해야 한다.  

[Stream이 더 적절한 데이터 연산 - 람다로 표현 가능]  
- 원소들의 시퀀스를 일관되게 변환하거나 필터링한다.  
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기, 최솟값 구하기 등)  
- 원소들의 시퀀스를 컬렉션에 모은다.  
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.  

[반복문이 더 적절한 데이터 연산 - 람다로 표현 불가]  
- 범위 안의 지역변수를 읽고 수정할 수 있어야 한다.  
- return, break, continue 문을 통해 특정 조건에서 반복문을 종료하거나 건너뛸 수 있어야 한다.  

어느쪽이 더 나은지 확신하기 힘들다면 둘 다 해보고 더 나은 쪽을 택하는 방법이 있다.  

### 심화 탐구

**출발점**
 
스트림 처리 구조는 '스트림 생성 -> 가공(중간연산) -> 소비(최종연산/종단연산)' 형태이며, 지연평가(lazy evaluation)라는 특징이 존재한다.  
또한, 최종연산을 통해 한번 close된 Stream을 가공하려면 다시 스트림을 생성해야 한다.

위와 같은 Stream API의 처리 구조에 따른 흐름을 구체적으로 쉽게 설명해보고,   
각 연산에 해당하는 메서드에는 어떠한 것들이 있는지 알아보면서 그 구조를 더 이해해보고자 한다.

**설명**

<hr>

1. 스트림 처리 구조의 흐름과 지연평가

    Stream API를 통해 데이터들을 원하는 형태로 가공하기 위해서는 데이터들이 Stream형태로 존재해야 한다. 데이터 집합을 Stream형태로 변환하는 과정을 'Stream 생성'이라고 한다. Stream API를 사용하기 위해 최초 1번 수행된다.

    데이터들이 Stream 형태로 변환되었다면, 다양한 연산(필터, 변형, 정렬 등)들을 통해 데이터를 가공할 수 있으며 이를 '중간연산'이라고 한다. 중간 연산에서는 입력값도 Stream, 결과값도 Stream이다.   
    입력값과 결과값이 같다면 떠오르는 것이 있다. 바로 '체이닝'이다. 여러개의 연산을 체이닝으로 엮어 연속 수행할 수 있는 것은 Stream API의 특징이기도 하다.

    원하는 형태로 데이터들을 가공하는 연산들을 엮었다면, 최종적으로 얻고자 하는 목적물을 만들어내기 위한 처리 과정이 필요하다. 이것을 '최종연산(또는 종단연산)'이라고 한다.   
    최종 연산은 최종 결과물을 얻기 위한 목적으로 1번만 수행할 수 있으며, 최종 연산을 통해 Stream이 닫히면 중간연산이나 최종연산을 재차 처리할 수 없다. (만일 추가 처리가 필요할 경우 다시 Stream을 생성해야 함)

    - 지연평가(Lazy Evaluation)란?

    stream 형태의 데이터들에 중간 연산들이 호출될 때, 즉시 수행되지 않고 최종연산이 호출되어야만 중간연산이 시작되는 것을 말한다.   
    Stream에서 데이터들의 처리 순서는 호출된 중간연산 하나마다 전체 데이터가 처리되고 다음 연산으로 넘어가는 것이 아니라, 마치 물이 흘러가듯 먼저 처리된 데이터들은 바로바로 다음 연산으로 넘어가는 구조이다.   
    지연평가는 도착지가 정해진 다음 흐르기 시작해야 하는 것으로 비유해볼 수 있다.

2. 각 처리 단계에 해당하는 메서드

    - Stream 생성

    ```java
    // List.stream()
    // List -> Stream
    List<String> list = new ArrayList<>();
    Stream<String> stream = list.stream();
    
    // Arrays.stream() or Stream.of()
    // Array -> Stream
    String[] array = {"Apple", "Banana", "Orange"};
    Stream<String> stream1 = Arrays.stream(array);
    Stream<String> stream2 = Stream.of(array);
    
    // generate : 단순히 어떤 값을 생성
    // Hello 라는 문자열을 Stream으로 생성하는 코드
    Stream<String> stream3 = Stream.generate(() -> "Hello");

    // iterate : 다음 요소를 생성하기 위해 이전 요소에 의존함.
    // 초기값 1, 다음값부터는 이전값에 2를 곱하여 요소를 생성한다.
    Stream<Integer> stream4 = Stream.iterate(1, n -> n*2).limit(5);
    ```

    - 중간 연산

    ```java
    // filter : 람다식이 true인 값만 남기고 제거
    // 1부터 100까지의 수 중 짝수만 남기는 코드
    IntStream.rangeClosed(1, 100).filter(n -> n % 2 == 0);
	
    // map : 각 원소를 특정 형태로 변환
    // 1부터 100까지의 수를 각각 제곱한 형태로 변환하는 코드
    IntStream.rangeClosed(1, 100).map(n -> n * n);
    
    // limit : 데이터 요소중 처음 몇개의 요소만 사용함
    // 1부터 100까지의 숫자 중 처음 10개의 숫자만 사용하는 코드
    IntStream.rangeClosed(1, 100).limit(10);
    ```

    이외에도 정렬연산 sorted(), 중간 내용 확인 연산 peak(람다식), 중복 제거 연산 distinct() 등의 연산들이 존재한다.

    - 최종 연산

    결국 최종 연산을 통해 원하는 데이터를 획득할 수 있다.

    >- forEach() : 각 요소를 순회하며 출력 등 처리 가능
    >- reduce() : 가장 앞에 있는 요소를 다음 원소와 특정 연산을 통해 하나의 값으로 줄여가는 메서드
    >- findFirst(), findAny() : 특정 조건에 맞는 요소를 찾기 위한 메서드
    >- anyMatch(), allMatch(), noneMatch() - 조건에 맞는지 여부 확인 (반환값 boolean)
    >- count(), max(), min() : 요소의 개수, 최대값, 최소값을 구하는 메서드
    >- sum(), average() : 요소들의 합계, 평균을 구하는 메서드
    >- collect() : stream 요소를 수집하여 원하는 형태(List, Set, Map, Array 등)로 변환

    <br>

    ```java
    // collect() 사용예시
    String[] array = {"Apple", "Banana", "Orange"};
    List<String> stream1 = Arrays.stream(array).collect(Collectors.toList());
    ```

3. (+추가) 병렬처리

    데이터 병렬 처리는 데이터 흐름을 나누어 멀티 스레드로 처리하고, 후에 합치는 과정을 통해 효과적인 대량 데이터 처리에 유리하다. Stream API는 데이터 병렬처리를 지원하며, parallel() 또는 parallelStream() 연산을 통해 병렬처리를 구현하고 처리 시간을 단축할 수 있다.

    시간을 단축할 수 있는 큰 장점에도 무조건적으로 사용하지 않는 이유는 다음과 같다.
    1. 처리 순서가 보장되지 않는다.
    2. 메모리 사용량이 증가한다. 각 요소를 여러 스레드에서 병렬로 처리하기 때문에 생성되는 객체수 증가로 메모리 사용량 또한 증가할 수 있다.
    3. 성능개선 효과가 불확실하다. 경우에 따라 병렬처리가 더 느릴 수도 있다.

<hr>
[참고자료]

- [Stream - Oracle](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [https://timotimo.tistory.com/80](https://timotimo.tistory.com/80)
- [https://www.elancer.co.kr/blog/view?seq=255](https://www.elancer.co.kr/blog/view?seq=255)
