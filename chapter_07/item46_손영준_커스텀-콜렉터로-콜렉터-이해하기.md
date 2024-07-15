## 커스텀 콜렉터로 콜렉터 이해하기

### 정리

자바 스트림은 계산을 변환으로 재구성하면서, 이전의 결과에 대해 처리하는 순수 함수라는 점이 핵심이다. 이는 곧 입력만이 결과에 영향을 줘야하며, 스트림 연산에 건네지는 함수 객체가 부작용이 없어야 함을 의미하기도 한다.

또한, forEach 연산은 스트림과 어울리지 않기에, 계산 자체에서 쓰기에는 거북하다. 대신 수집기(collector)를 이용해 스트림 원소들을 객체 하나에 취합할 수 있다. 맵 수집기(Map Collector)는 인수가 3개인 toMap, groupingBy와 함께 toSet()과 같은 다운스트림(downstream) 수집기를 이용하기, partitioningBy와 그에 쓰이는 Predicate를 이용하기 등 수집기를 이용할 수 있는 방식은 많다.

저자는 대표적으로 toList, toSet, toMap, groupingBy, joining과 같은 수집기 팩터리의 중요성을 이야기한다.

### 심화 탐구

**출발점**

이번 장에서는 여러 Stream Collector가 소개되었음에도, Stream도 Collector도 익숙하지 않다면 단순히 어떠어떠한 기능이 있다고 이해하고 넘어가려는 관성적 독해의 위험이 있다.

나를 포함한 대부분의 학습자들은 Map Collector를 이용하거나, toSet()이나 mapFactory를 이용한다는 이야기에 익숙하지 않기 마련이다. 그렇다면 커스텀 콜렉터를 직접 만들어보면서 과정을 이해해도 좋을 것이다.

**설명**

**콜렉터 구성요소**

Stream의 collect 메서드는 스트림 요소를 수집해서 컬렉션과 같은 형태로 변환할 수 있도록 돕는다. 그렇다면 collect 메서드는 어떻게 구성되어 있는가? `toList()`를 살펴보는 대신 직접 커스텀 콜렉터를 만들어보면서 원리를 이해해보자.

우선, collect 메서드는 세 가지 주요 매개변수를 이용하는데, 이는 각각 Supplier, Acuumulator, Combiner, Finisher라고 불린다. 이름과 연관지어보면 이해가 쉽다. Supplier는 스트림 요소를 변환해 담을 컨테이너를 생성하고, Accumulator는 결과를 반복적으로 생성된 컨테이너에 누적하며, Comnbiner는 스트림 병렬 처리에서 결과를 담은 컨테이너들을 하나로 합친다. 마지막으로, Finisher는 데이터 수집을 통해 누적된 결과를 원하는 형태의 최종 결과로 변환할 때 이용되기도 한다.

**콜렉터 만들어보기**

```java
// https://www.baeldung.com/java-8-collectors
public interface Collector<T, A, R> {...}
```

우선, 콜렉터는 위와 같은 제네릭 형태를 취한다. `T`는 콜렉션에서 이용가능한 오브젝트 타입을, `A`는 변경가능한 accumulator 오브젝트의 타입을, `R`은 최종 결과의 타입을 의미한다.

직접 콜렉터 클래스를 만들어서 이용해도 되지만, 이번 글에서는 간단하게 chatGPT가 생성한 코드와 함께 간편히 `Collector.of` 메서드를 이용해 Collector를 직접 만들어 써보는 것에 의의를 둬보자. 예시 코드는 StringBuilder를 이용해 요소를 문자열로 조인하는 커스텀 콜렉터를 생성한다.

```java
// chatGPT 생성 코드
import java.util.stream.Collector;
import java.util.stream.Stream;

public class CustomCollectorExample {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("apple", "banana", "cherry");

        Collector<String, StringBuilder, String> joiningCollector = Collector.of(
            StringBuilder::new,
            (sb, s) -> sb.append(s).append(", "),
            (sb1, sb2) -> {
                sb1.append(sb2);
                return sb1;
            },
            sb -> sb.length() > 0 ? sb.substring(0, sb.length() - 2) : "",
            Collector.Characteristics.UNORDERED
        );

        // 스트림을 문자열로 조인
        String result = stream.collect(joiningCollector);
        System.out.println(result);  // apple, banana, cherry
    }
}
```

위 코드에서, `joiningCollector` Collector 객체는 `String` 타입을 이용가능하며, `StringBuilder` 타입을 변경가능한 accumulator의 오브젝트 타입으로 이용하고, 최종적으로도 `String`으로 결과를 취합한다.

1. `StringBuilder::new`를 Supplier로 이용하면서, 앞으로의 데이터가 들어갈 컨테이너를 생성한다.

2. `(sb, s) -> sb.append(s).append(", ")`를 Accumulator로 이용하면서, 중간 단계의 데이터를 StringBuilder 컨테이너에 추가한다.

3. `(sb1, sb2) -> { sb1.append(sb2); return sb1; }`를 Combiner로 이용하면서, 두 개의 중간 단계 StringBuilder를 병합한다.

4. `sb -> sb.length() > 0 ? sb.substring(0, sb.length() - 2) : ""`를 finisher로 이용하면서, 마지막 쉼표와 공백을 제거한다.

5. `Collector.Characteristics.UNORDERED`를 characteristic으로 이용한다면, 이는 컬렉터가 스트림 요소 순서에 의존하지 않음을 의미한다. 즉, 데이터 수집 시에 순서에 구애받지 않고 요소를 처리할 수 있다.

추가로, characteristic으로 `Collector.Characteristics.IDENTITY_FINISH`를 이용한다면 이는 수집된 결과를 변경하지 않고 그대로 반환하는 항등 함수라는 의미다.

**콜렉터 클래스**

비록 깊이 분석하지는 않겠지만, 직접 콜렉터 클래스를 정의하여 이용하여도 된다. 아래 코드는 Baeldung의 코드로, supplier/accumulator/combiner/... 가 어떻게 이용될 수 있는지 살펴볼 수 있다.

```java
public class ImmutableSetCollector<T>
  implements Collector<T, ImmutableSet.Builder<T>, ImmutableSet<T>> {

  @Override
  public Supplier<ImmutableSet.Builder<T>> supplier() {
      return ImmutableSet::builder;
  }

  @Override
  public BiConsumer<ImmutableSet.Builder<T>, T> accumulator() {
      return ImmutableSet.Builder::add;
  }

  @Override
  public BinaryOperator<ImmutableSet.Builder<T>> combiner() {
      return (left, right) -> left.addAll(right.build());
  }

  @Override
  public Function<ImmutableSet.Builder<T>, ImmutableSet<T>> finisher() {
      return ImmutableSet.Builder::build;
  }

  @Override
  public Set<Characteristics> characteristics() {
      return Sets.immutableEnumSet(Characteristics.UNORDERED);
  }

  public static <T> ImmutableSetCollector<T> toImmutableSet() {
      return new ImmutableSetCollector<>();
  }
}
```

이 클래스는 기존 콜렉터 형태를 impelement하면서, 그에 필요한 메서드를 오버라이드한다. 앞서 살펴보았던 코드에서는 supplier 및 accumulator 등이 어떠한 클래스 타입을 가지는지 살펴보지 못했으나, 콜렉터 클래스에서는 이를 확인하고 응용할 수 있겠다.

또한, 위 코드는 아래와 같이 이용할 수 있다.

```java
List<String> givenList = Arrays.asList("a", "bb", "ccc", "dddd");

ImmutableSet<String> result = givenList.stream()
  .collect(toImmutableSet());
```

### References

https://www.baeldung.com/java-8-collectors

https://stackoverflow.com/questions/31533316/about-collect-supplier-accumulator-combiner
