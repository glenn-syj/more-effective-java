## ArrayList 클래스를 통한 제네릭 심화 탐구

### 정리

새로운 타입 설계 시에는 형변환 없이 사용할 수 있도록 할 필요가 있다. 여기에서 제네릭 클래스가 가져다주는 효용성은 크지만, 실체화 불가 타입으로 배열을 만들지는 못한다. 저자는 이에 대해 (1) 제네릭 배열 생성 금지에 대한 우회와 (2) 요소를 이용할 때에 시행되는 형 변환이다.

(1)에서는 주로, 실체화 불가 타입인 `E`에 대해, 배열 `E[]`을 `(E[]) Object[]`와 같은 제네릭 배열 형변환을 이용한다. 여기에서 비검사 형변환이 안전함이 논리적으로 증명되었다면, 타입 안정성을 해치지 않으니 경고를 숨길 수 있다.

(2)에서는 내부적으로는 `Object[]`을 이용하면서,원소를 읽을 때마다 `E`로의 형변환을 진행한다. 마찬가지로, 비검사 형변환이 안전하다는 점이 증명되면 경고를 숨길 수 있다.

또한 지난 28장에서는 배열 대신 List를 이용하라고 했지만, 실은 ArrayList 등에서도 배열을 이용해 구현되었다는 점도 눈여겨 볼 만하다.

### 심화 탐구

**출발점**

`List` 자료 구조를 직접 제네릭으로 구현해 본 적은 있지만, Java 17 Liberica에서 어떤 식으로 `ArrayList`가 구성되어 있는지 살펴본 적은 없다.

이번에는 Java에서 어떻게 `ArrayList` 클래스가 제네릭 타입과 함께 배열을 이용하고 있는지 살펴보려고 한다.

주석을 합한 코드 길이가 약 1700 라인에 이르는 까닭에, 필요한 부분만 따로 떼어내어 탐구하겠다.

**설명**

1. 상속 및 구현

```java

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

`ArrayList`는 해당 절에서 제시되었듯, 제네릭 클래스를 이용해 구현되고 있다. `<E>`는 Element를 대신하는 제네릭 타입이라고 볼 수 있다.

단순히 `List`가 아니라, `List<E>` 인터페이스를 구현하고 있음을 눈여겨 봐야한다. 즉, 제네릭 타입 구현체는 제네릭 타입 인터페이스를 구현한다. 이는 `AbstractList<E>`에서 드러나듯, 상속에서도 마찬가지다. 물론, 인터페이스와 추상 클래스가 모두 제네릭 타입을 이용하고 있다는 조건 하에서다.

2. 필드

```java

@java.io.Serial
    private static final long serialVersionUID = 8683452581122892189L;

    private static final int DEFAULT_CAPACITY = 10;

    private static final Object[] EMPTY_ELEMENTDATA = {};

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    transient Object[] elementData;

    private int size;

```

해당 절의 내용과 비슷하게, `Object[] elementData` 객체 배열을 통해 실제 객체의 삽입, 삭제, 검색이 이루어지고 있음을 추측할 수 있다.

특이한 점은, `EMPTY_ELEMENTDATA`와 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`라는 빈 final 변수의 이용이다. 이는 생성자와 메서드 파트에서 쓰임을 살펴볼 예정이다.

3. 생성자

```java

// 1번 생성자
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                            initialCapacity);
    }
}

// 2번 생성자
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 3번 생성자
public ArrayList(Collection<? extends E> c) {
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        // replace with empty array.
        elementData = EMPTY_ELEMENTDATA;
    }
}

```

- 1번 생성자

  기본형 `int` 타입을 인자로 받는 1번 생성자는 `int initialCapacity`가 양수인 경우, 0인 경우, 음수인 경우에 따라 다른 방식을 취한다.

  양수인 경우에는 `initialCapacity` 만큼의 크기를 가지는 `Object[]`를 `elementData`에 할당한다. 0인 경우에는 크기가 정해져 있지 않은 `EMPTY_ELEMENTDATA`를 이용한다. 음수인 경우에는 `IllegalArgumentException`을 반환한다.

- 2번 생성자

  2번 생성자는 어떤 것도 인자로 받지 않으며, 1번 생서자에서 인자가 0인 경우와 비슷하게 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`를 `elementData`에 할당한다.

- 3번 생성자

  3번 생성자의 파라미터 `Collection<? extends E> c`는 기존 제네릭 타입 클래스 `E`의 하위 클래스를 제네릭 타입으로 하는 `Collection`을 인자로 받음을 의미한다.

  여기에서는 (1) `c`를 배열로 변환하고 (2) 해당 배열의 크기를 검사한 뒤 (3) `c`의 클래스가 `ArrayList`인지 검사함으로써 방식을 나눈다.

  만약 `c`가 비어 있지 않고 `ArrayList` 클래스가 아니라면 아래와 같은 방식으로 `elementData`에 값을 할당한다.

  ```java
  elementData = Arrays.copyOf(a, size, Object[].class);
  ```

4. 메서드

- toArray

```java
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

`toArray()` 메서드는 `ArrayList` 타입인 `a`를 배열로 변환하도록 한다. 원소를 일일이 제네릭 타입으로 변환해 빈 배열에 삽입하는 대신, 형변환 `(T[])`을 이용해 실체화 불가 타입인 `T`에 대해서 금지를 우회하고 있다. 여기에서 비검사 형변환이 안전하다고 확인되므로, `@SuppressWarnings` 어노테이션을 이용해도 괜찮다.

### References

Effective Java 3/E
Java 17 Liberica