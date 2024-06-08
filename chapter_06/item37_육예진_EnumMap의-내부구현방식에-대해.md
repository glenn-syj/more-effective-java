## EnumMap의 내부구현방식에 대해

### 정리

**내용 요약**

이따금 배열이나 리스트의 원소들에 어떠한 값들을 연결하기 위해 ordinal 메서드로 인덱스를 얻는 코드들이 있다.  
하지만 이는 해당 배열의 원소들과 연결된 값 사이에 어떤 의미가 있는지 알기 힘들고, 순서나 입력값이 조금이라도 잘못되면 오류가 발생하기 쉽다.  
따라서 EnumMap을 통해 각 원소를 키값으로 하여, 연결된 값들과의 관계를 명확히 하는 것이 좋다.  


### 심화 탐구

**출발점**

EnumMap은 열거형 타입(Enum)을 키로 사용하기 위한 Map의 특수 구현이다. EnumMap의 모든 키는 지정된 하나의 Enum 타입에서 나와야 한다.  
저자는 EnumMap이 내부구현방식을 안으로 숨겨서 타입안정성과 배열의 성능을 모두 얻었다고 말하지만, 곧바로 와닿지 않았다.   
이 말이 의미하는 바가 무엇인지 보다 구체적으로 이해하기 위해 EnumMap의 생성자와 put/get 메서드 내부구현코드를 살펴보기로 했다.  

**설명**

<hr>

1. keyType을 통한 생성자와 데이터 추가

    우선 EnumMap의 필드에는 keyType, key배열, value배열, map사이즈가 있다.

    ```java
    private final Class<K> keyType;         // 해당 EnumMap에서 키로 사용하는 열거형타입
    private transient K[] keyUniverse;      // 모든 Key 목록을 배열화, (Cached for performance.) 내부 구현을 위해 저장해두는 것 같다.
    private transient Object[] vals;        // enum 클래스에서 선언된 순서 인덱스대로 매핑된 값을 배열에 담는다.
                                            // 매핑한 값이 없으면 해당 인덱스는 null인채로 비워둔다.
    private transient int size = 0;
    ```

    생성자에 매개변수로 keyType(enum)을 넘기면, 해당 enum에 정의된 상수의 개수만큼 내부적으로 Object 배열을 생성한다.

    ```java
    public EnumMap(Class<K> keyType) {
        // 키타입 저장
        this.keyType = keyType;
        // 상수들을 배열로 저장
        keyUniverse = getKeyUniverse(keyType);
        // key 개수만큼 값 배열길이 생성
        vals = new Object[keyUniverse.length];
    }

    // 여기서 getKeyUniverse라는 내부 메서드는 해당 enum의 상수들을 배열 형태로 반환함.
    private static <K extends Enum<K>> K[] getKeyUniverse(Class<K> keyType) {
        return SharedSecrets.getJavaLangAccess()
                                        .getEnumConstantsShared(keyType);
    }
    ```
    그리고, put 할때마다 enum 상수의 정렬순서(정의된 순서)를 index로 배열에 값을 할당한다.

    ```java
    public V put(K key, V value) {
        // 생성자에서 저장해둔 키타입과 동일한지 확인하는 메서드
        typeCheck(key);

        // 해당 키의 enum 정의 순서 인덱스를 찾아오는데, 해당 인덱스
        int index = key.ordinal();

        // 해당 인덱스에 값이 없으면 즉, 값이 없어서 entry가 존재하지 않았다면 size 증가시키고 값을 추가
        Object oldValue = vals[index];
        vals[index] = maskNull(value);
        if (oldValue == null)
            size++;
        return unmaskNull(oldValue);
    }

    // null 값 처리와 관련된 내부 메서드 2가지 첨부
    private Object maskNull(Object value) {
        return (value == null ? NULL : value);
    }

    @SuppressWarnings("unchecked")
    private V unmaskNull(Object value) {
        return (V)(value == NULL ? null : value);
    }
    ```
    EnumMap 자료구조에서 내부적으로 배열을 사용하고 있기 때문에 실제로는 배열의 인덱스를 사용하듯이 기능이 구현되어 있다.  
    하지만 해당 코드들은 모두 private으로 클라이언트에 공개되지 않는다.  
    이에 따라 실제 구현에는 배열 인덱싱의 장점을 활용하면서도 타입 안정성을 챙길 수 있는 것이다.   

    get 메서드에서도 키값을 통해 값을 찾으려할 때, enum에서 해당 key의 인덱스를 찾고 해당 인덱스에 연결된 값을 반환하는 것으로 보인다.  

    ```java
    public V get(Object key) {
        // null이 아니고 생성자에 설정된 keyType과 일치하면
        return (isValidKey(key) ?
                // 열거형 타입에서의 인덱스를 통해 매핑해둔 값을 반환
                unmaskNull(vals[((Enum<?>)key).ordinal()]) : null);
    }
    ```


**결론**
    
EnumMap 내부구현 코드를 살펴보면 배열을 사용했고 ordinal() 메서드도 보인다.  
이를 통해 단순히 ordinal 메서드를 사용하지 말라는 것이 아니라, 각 열거타입과 그에 매핑되는 값들 사이의 관계를 명확히 해야한다는 것을 의미한다.  
직접 위와 같은 코드들을 구현해서 사용할수도 있겠지만 우리는 이미 EnumMap이라는 잘 구현된 자료구조를 사용할 수 있다.  
다만, 그 성능을 배열을 통해 어떻게 구현했는지와 이를 내부구현으로 숨김을 통해 클라이언트가 형변환에 신경 쓸 필요 없고, 인덱스 오류를 낼 가능성도 없어졌다는 내용은 이해해 둘 필요가 있다.

<hr>


[참고 자료]
- [Class EnumMap<K extends Enum<K>,V>](https://docs.oracle.com/javase%2F8%2Fdocs%2Fapi%2F%2F/java/util/EnumMap.html)
