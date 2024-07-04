# 메서드 참조 방식의 특징 및 장단점에 대한 고찰



## item43 - 람다보다는 메서드 참조를 사용하라

### 🔍 내용 요약

람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 **간결함**이다. 그런데 자바에는 함수 객체를 심지어 람다보다도 더 간결하게 만드는 방법이 있으니, 바로 **메서드 참조**다. 

다음 코드는 임의의 키와 Integer 값의 매핑을 관리하는 프로그램의 일부다. 
merge 메서드는 키, 값, 함수를 인수로 받으며, 주어진 키가 맵 안에 아직 없다면 주어진 {키, 값} 쌍을 그대로 저장하고, 키가 이미 있다면 세 번째 인수로 받은 함수를 현재 값과 주어진 값에 적용한 다음, 그 결과로 현재 값을 덮어쓴다. 

```java
map.merge(key, 1, (count, incr) -> count + incr);
```

다음 코드는 위의 코드에서 크게 하는 일이 없이 공간을 차지하는 count와 incr를 메서드의 참조를 전달하여 똑같은 결과를 더 보기 좋게 얻는 코드다. 

```java
map.merge(key, 1, Integer::sum);
```

메서드 참조는 람다의 간단명료한 대안이 될 수 있지만, 항상 그런 것은 아닐 수 있으므로 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하자. 

메서드 참조의 유형은 총 다섯 가지가 있다. 
1. 정적 메서드를 가리키는 메서드 참조
2. 인스턴스 메서드를 참조하는 유형
    - 한정적(bound) 인스턴스 메서드 참조
    - 비한정적(unbound) 인스턴스 메서드 참조
3. 생성자를 가리키는 메서드 참조
    - 클래스 생성자를 가리키는 메서드 참조
    - 배열 생성자를 가리키는 메서드 참조

추가로 람다로 할 수 없는 일이라면 보통 메서드 참조로도 할 수 없는데, 제네릭 함수 타입을 구현할 때는 메서드 참조만이 가능하다. 

<br>

--------------------------------------------------

### 🧐 심화 탐구

이전 아이템의 **익명 클래스보다는 람다를 사용하라**와 다음 아이템의 **표준 함수형 인터페이스를 사용하라**와 연결된 아이템이라는 생각이 들었다. 이후 스트림에 대해 학습하면 더욱 자주 사용하게될 메서드 참조에 대해 특징과 장단점에 대해 알아보는 것을 이번 심화 탐구의 주제로 삼았다. 

이번 아이템에서 메서드 참조의 예시로 든 merge의 사용 예시의 경우 일단 메서드 참조라는 것이 너무 생소해서 이해하기가 어려웠다. 

```java
default V merge(K key, V value,
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
                remappingFunction.apply(oldValue, value);
    if (newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

다음은 merge 메서드의 내부 구조이다. 

여기서 메서드 참조를 넣을 수 있는 파라미터의 타입이 `BiFunction<? super V, ? super V, ? extends V>` 으로 선언된 것을 볼 수 있었다. 
BiFunction은 기본 함수형 인터페이스(p.265)중 하나로 타입 매개변수의 종류가 3가지로 보이는데 마지막 타입 매개변수(R)의 경우 apply 메서드의 반환형을 정의하기 위해서만 존재해서 교재에서는 인수를 2개씩 받는다고 써있는 것으로 보인다. 

파라미터로 함수형 인터페이스를 받고 있으므로 이는 교재와 동일한 기능을 할 것으로 예상 되는 코드를 작성해보았다.(기존의 Comparator 사용법에서 영감을 받아 작성했다)

```java
package item43;

import java.util.HashMap;
import java.util.Map;
import java.util.function.*;

public class Main {
    
    static class MyBiFunction implements BiFunction<Integer, Integer, Integer>{

        @Override
        public Integer apply(Integer i1, Integer i2) {
            return i1 + i2;
        }
    }

    public static void main(String[] args) {

        Map<Integer, Integer> map = new HashMap<>();

        int key = 1;

        // case1 - 함수 객체 방식
        MyBiFunction myBiFunction = new MyBiFunction();
        map.merge(key, 1, myBiFunction);
        System.out.println(map.get(key));    // 1

        // case2 - 익명 클래스(since JDK1.1)
        map.merge(key, 1, new BiFunction<Integer, Integer, Integer>() {
            @Override
            public Integer apply(Integer i1, Integer i2) {
                return i1 + i2;
            }
        });
        System.out.println(map.get(key));    // 2

        // case3 - 람다식(since java 8)
        map.merge(key, 1, (count, incr) -> count + incr);
        System.out.println(map.get(key));    // 3

        // case4 - 메서드 참조(since java 8)
        map.merge(key, 1, Integer::sum);
        System.out.println(map.get(key));    // 4
    }
}

```

4가지 방식으로 코드를 작성해보았고 실행결과를 통해 모두 동일한 기능을 한다고 생각했다. 

함수 객체 방식의 경우 기존의 고전적인 자바 문법이 익숙한 개발자에게 가장 직관적인 스타일이라는 생각이 들었다. 인터페이스의 추상 메서드를 구현하고 객체를 생성해서 전달하는 방식이라 코드의 길이가 상당히 길어지지만 어색함 없이 받아들일 수 있다는게 장점인 것 같았다. 

익명 클래스 방식의 경우 `new 인터페이스 { 추상 메서드 재정의 }` 로 사용한다는 점에서 new 뒤에는 객체 타입이 클래스라면 같은 클래스 혹은 인터페이스 타입이라면 이를 구현하는 클래스가 와야한다는 나의 인식에 바로 벗어나는 문법처럼 느껴졌다. 추상 메서드를 재정의하니 얼핏 문제가 없어보이기도 해서 단순히 어색함의 문제이지 않을까도 싶다. 

다음은 익명 클래스보다 더욱 간단하게 쓸 수 있는 람다식으로 추상 메서드가 하나만 있을 경우 특별 대우로 코드의 길이가 짧아질 뿐더러 어떤 기능을 하는지도 더 잘 표현하는 것으로 보였다. 하지만 익명 클래스 방식과 달리 반환 타입은 물론 다른 매개변수 타입까지 생략되어 있어 익숙하지 않은 개발자에게는 혼란을 가져올 수도 있을 것 같았다.(실제로 컴파일러가 알아서 처리해준다)

마지막은 메서드 참조 방식으로 람다식보다 더욱 생략을 많이 했는데 가장 큰 차이는 매개변수조차 생략해서 가독성을 올리는 것으로 보였다. map의 타입 매개변수가 <Integer, Integer>로 선언되어 있어 타입 추론이 가능하며 매개변수 2개를 받는 BiFunction 자리에 매개변수 2개를 받는 sum을 넣은 것으로 모든 것을 컴파일러가 알아서 해줄것으로 기대하는 코드로 보이는데 익숙하지 않은 개발자에겐 이게 자바 문법이 맞나 싶을 것 같았다.(이는 함수형 프로그래밍이 그저 어색해서 생기는 문제라고 생각한다.)

```java
package item43;

import java.util.HashMap;
import java.util.Map;
import java.util.function.*;

public class Main {

    static class MyBiFunction implements BiFunction<Integer, Integer, Integer>{

        @Override
        public Integer apply(Integer i1, Integer i2) {
            return i1 + i2;
        }
    }

    public static void main(String[] args) {

        Map<Integer, Integer> map = new HashMap<>();

        int key = 1;
        int len = 100_000_000;
        long starTime = 0;
        long endTime = 0;

        // case1 - 함수 객체 방식
        starTime = System.nanoTime();
        MyBiFunction myBiFunction = new MyBiFunction();
        for(int i=0 ; i<len ; i++){
            map.merge(key, 1, myBiFunction);
        }
        System.out.println(map.get(key));    // 1
        endTime = System.nanoTime();
        System.out.println((endTime - starTime) / 1000000 + "ms");

        // case2 - 익명 클래스
        starTime = System.nanoTime();
        for(int i=0 ; i<len ; i++){
            map.merge(key, 1, new BiFunction<Integer, Integer, Integer>() {
                @Override
                public Integer apply(Integer i1, Integer i2) {
                    return i1 + i2;
                }
            });
        }
        System.out.println(map.get(key));    // 2
        endTime = System.nanoTime();
        System.out.println((endTime - starTime) / 1000000 + "ms");

        // case3 - 람다식
        starTime = System.nanoTime();
        for(int i=0 ; i<len ; i++){
            map.merge(key, 1, (count, incr) -> count + incr);
        }
        System.out.println(map.get(key));    // 3
        endTime = System.nanoTime();
        System.out.println((endTime - starTime) / 1000000 + "ms");

        // case4 - 메서드 참조
        starTime = System.nanoTime();
        for(int i=0 ; i<len ; i++){
            map.merge(key, 1, Integer::sum);
        }
        System.out.println(map.get(key));    // 4
        endTime = System.nanoTime();
        System.out.println((endTime - starTime) / 1000000 + "ms");
    }
}

```

다음은 내 PC 기준으로 성능차이가 있을까 싶어서 System.nanoTime을 통해 테스트를 해본 코드이다. 백준 기준 알고리즘 문제 풀이를 제출할 경우 람다식으로 정렬한 코드보다 Comparator를 직접 만들어서 넣어줬을때 약간 더 빠르다고 생각해서 작성해본 코드였는데, 내 PC 환경과 코드를 기준으로는 실행 횟수에 따라 어느 정도 성능 차이가 유의미하게 있는 것처럼 보였다.(최대 20%) ChatGPT에게 질문했을 때는 case1에서 case4로 갈수록 최적화가 이루어져 빠를 것으로 예상한다 했는데 실제로는 익명 클래스 방식만 약간 느리고 나머지는 거의 비슷하게 나왔다. 

탐구를 진행하며 메서드 참조에 대해 알아보고자 했는데 전반적으로 진행하면서 성능면에서 최소한 별다른 손해는 없으며 가독성이 좋고 기능을 명확하게 나타낼 확률이 높으며 유지보수가 쉽게 보인다는 점에서 메서드 참조는 매력적인 문법이라는 생각이 들었다. 
메서드 참조가 어색한 개발자라면 충분한 학습을 통해 함수형 프로그래밍에 익숙해지면 좋을 것 같다. 

--------------------------------------------------

출처
https://docs.oracle.com/javase/8/docs/api/ : 메서드 참조 기능이 생긴 버전에 대한 검색
ChatGPT : 구현 방식별 성능 차이에 대한 질의응답

--------------------------------------------------

### 🧠 어려웠던 점

메서드 참조와 함수형 인터페이스에 대한 내용 및 사용 예제들을 보면서 프레임워크를 사용할때는 프레임워크에 맞게 개발을 하는 것과 비슷하다는 느낌이 들어서 자세한 원리를 이해하기 어려웠다. 개발자를 위한 편의기능의 이면에는 디테일한 원리가 녹아들어 있으므로 언어의 메커니즘에 대한 충분한 학습이 필요할 것 같다. 