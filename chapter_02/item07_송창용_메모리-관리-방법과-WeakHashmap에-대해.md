## 메모리 관리 방법에 대해

### 정리

**내용 요약**

item 7에서는 자바의 가비지 컬렉터가 프로그래머를 편하게 만들어주지만 완벽하지 않기 때문에 메모리를 지속적으로 관리해야한다고 말한다.

가비지 컬렉터는 프로그래머의 입장에서 더 이상 사용하지 않는 객체의 참조를 다른 곳에서 갖고 있는 경우, 해당 객체가 더 이상 사용되지 않는 불필요한 존재라는 것을 인식하지 못하기 때문이다.

이러한 가비지 컬렉터의 단점은 프로그램에 큰 악영향을 끼칠 가능성을 가지고 있다.

가비지 컬렉터가 더 이상 사용하지 않는 하나의 객체에 대한 참조를 놓치고 남겨두기만 해도, 의도치않게 존재하게 된 객체가 다른 불필요한 객체들을 참조할 수 있다. 

결국 가비지 컬렉터가 놓친 하나의 객체가 참조하고 있는 또 다른 불필요한 객체들도 연쇄적으로 처리되지 않고 남아있게 된다.

이러한 연쇄가 몇 단계만 이어져도 수많은 불필요한 객체가 메모리를 잡아먹게 되어 성능을 낮추고, 심각할 때는 OutOfMemoryError를 일으킨다.

아래는 가비지 컬렉터가 인식하지 못하는 불필요한 객체를 생성하는 상황의 예시이다.

```java
public class Stack{
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e){
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop(){
        if(size == 0){
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity(){
        if(elements.length == size){
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

}
```

위 코드에서 pop 메소드를 실행하면, Stack 배열의 원소로 존재하던 객체를 반환한다.

이렇게 반환된 객체는 스택에서 계속해서 참조를 하고 있기 때문에 더 이상 사용하지 않더라도 가비지 컬렉터에서 처리하지 않게 된다.

가비지 컬렉터가 놓치는 객체들을 처리하기 위해서는 프로그래머가 직접 참조를 null로 만들어서 처리하거나, 해당 참조를 담은 변수를 유효 범위 밖으로 밀어내야 한다. 

만약 변수의 범위를 최소가 되게 정의했다면 참조를 담은 변수는 자연스럽게 유효 범위 밖에 위치함으로써 메모리 처리가 이루어질 것이다.

위의 예시 코드로 보여준 Stack 클래스처럼 자기 메모리를 직접 관리하는 클래스 외에도 메모리 누수를 일으키는 주범으로 캐쉬와 리스너가 존재한다.

Stack 클래스처럼 자기 메모리를 직접 관리하는 클래스는 더 이상 사용하지 않는 객체를 대상으로 null로 참조를 처리하는 방식으로 메모리를 관리할 수 있고,

캐쉬와 리스너의 경우 WeakHashMap을 이용하여 더 이상 사용하지 않지만 남아있는 객체를 처리할 수 있다.

### 심화 탐구

**출발점**

1. item 7 본문에서 가비지 컬렉터가 놓치는 메모리 누수에 대한 위험성을 말하며 프로그래머가 직접 메모리를 관리하는 방법에 대해 언급한다.

그 중 객체 참조를 null 처리하여 메모리를 관리하는 방법은 바람직하지 않으며, 해당 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이 가장 좋은 방법이라 말하고 있다.

여기서 말하는 `해당 참조를 담은 변수를 유효 범위 밖으로 밀어내는 방식`에 대한 이해가 어려웠기 때문에 `변수를 유효 범위 밖으로 밀어내는 것`이 정확히 무엇인지 알아보도록 하겠다.

2. 캐쉬와 리스너의 경우에서, WeakHashMap을 이용하여 더 이상 사용하지 않지만 남아있는 객체를 처리하는 방법에 대한 이해가 부족하여 WeakHashMap에 대해 탐구해 보겠다.

**설명**

1. 변수를 유효 범위 밖으로 밀어내는 방식에 대한 탐구.

본문에서 '변수를 유효 범위 밖으로 밀어내는 것'은 변수의 범위를 최소가 되게 정의하면 자연스럽게 일어난다고 말한다.

'변수의 범위를 최소가 되게 정의하는 것'은 `이펙티브 자바 3/E`의 `9장 일반적인 프로그래밍 원칙`, `item 57 : 변수의 범위를 최소가 되게 정의하라` 에서 설명하고 있다.

여기서 말하는 `범위`란 처음 보았을 때는 특정 데이터 타입의 크기 같은 것을 말하는 것이라 생각하였지만, item 57을 살펴본 이후 `scope`를 지칭하는 말임을 알게되었다.

즉, `변수의 범위를 최소가 되게 정의`하는 것은 scope를 고려하여 scope 바깥에서 변수를 선언하는 것이 아닌, scope 내에서 선언하여 해당 scope가 끝나면 변수가 자연스럽게 유효 범위 밖으로 벗어나면서 처리되도록 하는 것을 의미한다.

만일 scope 내에서만 사용하는 변수를 scope 바깥에서 선언한다면, scope가 끝나고 더 이상 해당 변수를 사용하지 않더라도 끝까지 존재하게 된다.

코드로 간단하게 표현하자면 다음과 같다.

```java
public class Test{
    public static void main(String[] args){
        Object object;
        while(true){
            object = new Object();
        }
    }
}
```
이렇게 코드를 작성했을 경우, object 변수는 for문이 끝난 이후에 더 이상 사용하지 않지만, 계속해서 메모리를 차지하게 된다.


2. WeakHashMap에 대해.

```java
WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
BigImage bigImage = new BigImage("image_id");
UniqueImageName imageName = new UniqueImageName("name_of_big_image");

map.put(imageName, bigImage);
assertTrue(map.containsKey(imageName));

imageName = null;
System.gc();
```

일반적인 HashMap 자료구조라면, key로 사용되는 imageName를 null처리 하여도 시스템의 많은 메모리를 차지하는 큰 이미지 데이터가 더 이상 사용되지 않아도 가비지 컬렉터로 하여금 처리하도록 만들지 않는다.

그러나 WeekHashMap 자료구조를 사용한다면, key로 사용되는 부분이 null 처리될 경우, key와 묶인 value가 더 이상 사용되지 않는 객체일 때, 가비지 컬렉터로 하여금 처리하도록 만든다.

WeakHashMap이 작동하는 원리는 WeakReference(약한 참조)와 관련되어 있으나, 약한 참조에 대해 찾아보아도 이해가 어려워 WeakHashMap에 대한 구체적인 작동 원리는 추후 약한 참조에 추가적인 탐구 이후 설명하도록 하겠다.


References :

[Guide to WeakHashMap in Java]
https://www.baeldung.com/java-weakhashmap

[Class WeakHashMap<K,​V>]
https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/WeakHashMap.html