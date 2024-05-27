## 제네릭 싱글턴 팩터리 메서드란?

### 정리

**내용 요약**

클래스와 마찬자지로 메서드도 제네릭으로 만들 수 있다.  
매개변수와 반환값의 형변환이 필요한 메서드는 제네릭을 사용하는 것이 안전하고 사용하기 쉽다.

메서드에서 취급할 타입을 타입 매개변수 목록으로서 메서드의 제한자와 반환 타입 사이에 선언하고,  
매개변수와 반환값에 해당 타입을 사용하도록 통일하면 된다.

### 심화 탐구

**출발점**

책에서는 메서드를 제네릭으로 구현한 예시 중 하나로 제네릭 싱글턴 팩터리를 언급한다.  
제네릭은 런타임에 타입정보가 소거되므로 어떤 타입으로든 매개변수화 할 수 있지만, 이를 위해서는 요청한 타입 매개변수에 맞게 객체의 타입을 바꿔주어야 하는데, 그 역할을 하는 정적 팩터리 메서드가 제네릭 싱글턴 팩터리 패턴이다.  

책에서 예시로 다룬 UnaryOperator 인터페이스는 제네릭 싱글턴 팩터리의 가장 기본적인 형태인 것으로 보인다.  
UnaryOperator 인터페이스에 대한 이해를 통해 추후 제네릭 싱글턴 팩터리 패턴 활용 가능성을 열어두고자 한다.  

**설명**

<hr>

1. UnaryOperator 인터페이스

    UnaryOperator 인터페이스는 함수인터페이스로 identity라는 단항연산자를 구현해야 한다.

    ```java
    @FunctionalInterface
    public interface UnaryOperator<T> extends Function<T, T> {

        /**
        * Returns a unary operator that always returns its input argument.
        *
        * @param <T> the type of the input and output of the operator
        * @return a unary operator that always returns its input argument
        */
        static <T> UnaryOperator<T> identity() {
            return t -> t;
        }
    }
    ```
    identity 메서드 구현부를 보면 람다식을 사용해 입력값을 같은 타입으로 반환하는 것을 알 수 있다.
    입력을 변형하거나 갱신하는 연산에 사용되며, 입력과 출력 타입이 같을 때 사용하기에 적합하다. 

    이 인터페이스를 다음과 같이 사용해볼 수 있다.

    ```java
    // 해당 인터페이스를 선언할때 같은 타입을 반환하는 단항연산을 구현한다.
    UnaryOperator<Integer> uo = t -> t*2;
	
    // 함수 객체를 사용하고자 할때 apply 메서드를 사용한다.
    System.out.println(uo.apply(2));
    
    // 출력 : 4
    ```


2. 사용이유
   
    컬렉션에 UnaryOperator를 연결하여 사용하면, 리스트의 모든 값을 특정한 흐름으로 갱신하고자 할때 유용하다.
    항상 같은 타입의 반환값을 보장하기 때문에 타입 안정성이 있고,   
    선언시 변수명에 신경을 쓴다면 단순히 컬렉션 map 메서드에 람다식을 바로 사용하는 것보다 직관적인 효과를 가질 수 있다.

    예를 들어, 다음과 같은 코드를 작성해볼 수 있다.

    ```java
    List<Integer> Nums = new ArrayList<>(List.of(2, 3, 4, 5));
		
    // map 메서드에 직접 람다식 구현부를 작성한 예시
    List<Integer> newNums1 = Nums.stream().map(t -> t*2).toList();
    
    // map 메서드 람다식 구현부에 UnaryOperator를 사용한 예시
    UnaryOperator<Integer> getDouble = t -> t*2;
    List<Integer> newNums2 = Nums.stream().map(t -> getDouble.apply(t)).toList();

    System.out.println(newNums1);  // [4, 6, 8, 10]
    System.out.println(newNums2);  // [4, 6, 8, 10]
    ```

    위 코드는 원소들의 형태를 2배 값으로 갱신한 리스트를 생성하도록 컬렉션의 map 메서드를 사용한 것이다.
    비록 단순히 값을 두배로 변경하는 내용인 탓에 newNums1을 구하는 코드가 더 간단해 보이지만,  
    복잡한 계산일 경우 람다식 구현부에서 보다 나은 가독성을 제공할 것이다.



**결론**
    
제네릭이 실체화된다면 위와 같은 역할을 해주는 항등함수는 타입별로 작성해야할 것이다.
하지만 소거 방식 덕에 제네릭 싱글턴 패턴을 사용하여 매개변수가 어떤 타입이든 해당하는 타입을 반환하는 제네릭 싱글턴 팩터리를 사용할 수 있다.

<hr>


[참고 자료]
- [Interface UnaryOperator(oracle)](https://docs.oracle.com/javase/8/docs/api/java/util/function/UnaryOperator.html)
- [ UnaryOperator 함수형 인터페이스란?](https://burningfalls.github.io/java/what-is-unaryoperator-functional-interface/)
