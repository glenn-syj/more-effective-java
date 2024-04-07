## equals 메서드 재정의 지양에 관해

### 정리

**내용 요약**

equals 메서드는 Object를 상속받은 모든 클래스들이 각자의 인스턴스들을 비교해, 서로 같은 객체인지 확인할 수 있는 메서드이다.
각 객체의 특성에 따라 재정의 할 수 있지만, 다음과 같은 상황이라면 최대한 지양하도록 한다.

- equals를 재정의 해야할 때
  - 객체 식별성이 아닌 논리적 동치성 확인이 필요한데, 상위 클래스의 equals 메서드가 적절히 재정의 되지 않았을 경우. (주로 값을 표현하는 클래스: Integer, String 등)


- equals 메서드 재정의 시 지켜야 할 규약
  - **반사성(reflexivity)**: null이 아닌 모든 참조값 x에 대해, x.equals(x)는 true다.
  - **대칭성(symmetry)**: null이 아닌 모든 참조값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
  - **추이성(transitivity)**: null이 아닌 모든 참조값 x, y, z에 대해, x.equals(y)가 true이고, y.equals(z)도 true이면 x.equals(z)도 true다.
  - **일관성(consistency)**: null이 아닌 모든 참조값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
  - **null-아님**: null이 아닌 모든 참조값 x에 대해, x.equals(null)은 false다.



### 심화 탐구

**출발점**

>(Object 공통) 메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스를 오동작하게 만들 수 있다.

이번 장에서는 equals 메서드 재정의시 지켜야 할 규약에 대해 이야기 하고있다.  
그렇게 어려운 내용은 아니고 두 객체의 동치성을 확인하려면 당연히 고려할 사항이다.   
하지만 왜 이렇게까지 규약을 정해두는 것이고, 심지어는 웬만하면 재정의하지 말라고 하는 것일까.  
equals 메서드를 재정의 한 예시와 이를 사용하는 다른 클래스/메서드를 살펴봄으로써 해당 장의 주장 이유와 규약의 중요성에 대해 인식하고자 한다.


**설명**

<hr>

1. Object의 equals 메서드
   
    가장 상위클래스인 Object 클래스에서는 다음과 같은 equals 정적 메서드를 제공하며, 해당 정적 메서드를 사용할 때 다시 멤버메서드가 호출된다.
    ```java
    // 정적 메서드
    public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }

    // 멤버 메서드
    // 해당 API문서에는 위 내용 요약에 있는 규약 5가지가 정확히 적혀있다.
    public boolean equals(Object obj) {
        return (this == obj);
    }
    ```
    Object를 상속받는 모든 클래스가 갖는 equals메서드는 후자이다.  
    == 연산자는 두 객체의 참조값을 비교한다.  
    equals 메서드를 재정의(Overriding)하지 않는 경우, 주소값을 기준으로 동치성 여부를 판단하는 것으로 보인다.  
    즉, 주소값만 각 인스턴스를 고유하게 구분할 수 있다는 뜻이다.  
    
    임의의 클래스의 equals 메서드를 따라 api문서로 따라가보면, 분명히 비교방식이 달라야 하는 자료구조의 equals메서드도 위와 같이 Object 클래스의 메서드로 이동한다.   
    이것은 인수로 들어오는 객체의 타입에 따라 equals 메서드가 오버라이딩 된 상태라면 자동적으로 해당 메서드를 사용하기 때문인 것 같다.


2. equals를 재정의한 클래스들
   
   그러나 클래스 특성에 따라 참조값을 동치의 기준으로 삼지 않고 객체가 가진 값들을 비교하여 논리적 동치성 판단이 필요한 경우, equals메서드를 재정의 해야 한다.  
   우리가 가장 자주 사용하는 Integer, String의 equals 메서드를 살펴보자.

   - Integer, String 클래스의 equals 메서드 (출처: jre api)
    ```java
    // Integer 클래스의 equals 메서드
    public boolean equals(Object obj) {

        // 비교할 대상 객체가 Integer의 인스턴스여야 함.
        if (obj instanceof Integer) {

            // 두 객체의 value라는 필드값에 접근하여 비교한다.
            return value == ((Integer)obj).intValue();
        }
        return false;
    }


    // String 클래스의 equals 메서드
    public boolean equals(Object anObject) {

        // 두 객체의 주소값이 같으면 더 비교할 필요없이 같은 객체다.
        // 메서드 효율성을 위해 주소값을 먼저 비교
        if (this == anObject) {
            return true;
        }

        // 매개변수가 String 의 인스턴스이고, 문자 형식/인코딩이 알맞으면 두 객체의 값을 드디어 비교한다.
        return (anObject instanceof String aString)
                && (!COMPACT_STRINGS || this.coder == aString.coder)
                && StringLatin1.equals(value, aString.value);
    }
    ```
    그 특성에 맞게 각 인스턴스의 value를 서로 정확히 비교하도록 오버라이딩 된 것을 볼 수 있다.  

    이러한 과정을 통해 컴퓨터에게 어떠한 두 객체의 주소값이 다르더라도 인스턴스가 갖는 주요값(직접 설정)들이 같으면 같은 객체라고 판단할 수 있게 하는 것이다. 

3. equals 메서드를 활용하는 다른 클래스/메서드

    제네릭 타입을 사용하는 자료구조 클래스들은 객체가 특정 원소를 가지고 있는지 판단하기 위해 contains 메서드에 equals 메서드를 사용하고 있다.  

    가장 익숙한 자료구조 중 하나인 ArrayList 클래스를 살펴보자.  
    Arrays의 contains메서드를 호출하면 아래와 같이 contains > indexOf > indexOfRange 순서대로 메서드가 재호출된다.

    ```java
    // indexOf(o)가 0보다 작으면 같은 값을 찾지 못했다는 의미
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    // 특정 범위 안에서 매개변수와 같은 객체를 찾아 그 인덱스를 반환하는 indexOfRange 인덱스를 호출
    public int indexOf(Object o) {
        return indexOfRange(o, 0, size);
    }

    // 최종적으로 equals 메서드는 이곳에서 하게 됨
    int indexOfRange(Object o, int start, int end) {
        Object[] es = elementData;
        if (o == null) {
            for (int i = start; i < end; i++) {
                if (es[i] == null) {
                    return i;
                }
            }
        } else {
            for (int i = start; i < end; i++) {
                if (o.equals(es[i])) {      // equals 메서드 호출
                    return i;
                }
            }
        }
        return -1;
    }
    ```
    최종적으로 호출되는 indexOfRange에서는 매개변수가 null이라면 원소중에 null이 있는지 찾아보고, null이 아니라면 같은 객체가 존재하는지 하나씩 비교한다.  

    이때, 코드에서 equals메서드를 타고 들어가면 Object클래스의 것으로 연결된다. 하지만, 보이는 것처럼 단순히 Object의 원본 equals 메서드를 사용하고 있는 것이라면 다음과 같은 일이 일어날 것이다.
   
    ```java
    // String 클래스의 equals 메서드가 오버라이딩 되지 않았다고 가정할 경우 예상 결과
    String str1 = "육예진";
    String str2 = new String("육예진");
    
    List<String> list1 = new ArrayList<>();
    list1.add(str1);
    
    System.out.println(list1.contains(str2));   // false (실제 값 true)
    ```
     객체의 값을 비교하는 equals가 제대로 구현되어 있지 않다면 메서드는 가지고 있는 값을 없다고 대답할 수도 있다.
     하지만 실제로 위와 같이 객체를 생성하고 contains 메서드를 사용했을 때, true가 반환된다.  
     
     이 결과를 통해 우리는 위 indexOfRange에서 호출된 equals가 매개변수 Object o (위 예시의 경우 String 타입의 객체가 매개변수로 들어올 것이다.)의 하위 타입 클래스에서 재정의된 equals를 사용하고 있다고 예상할 수 있다. 

     ArrayList만 살펴봤지만 출발점의 인용구에서 말한 `대상 클래스가 이 규약을 준수한다고 가정하는 클래스`는 더 다양하다.
    

**결론**

이처럼 equals는 Object를 상속한 모든 객체가 갖는 공통 메서드 중 하나이다. 두 객체가 같은 값인지를 비교하는 단순 기능을 넘어 객체를 사용하는 자료구조 등의 메서드에서도 사용된다. 

규약에 맞춰 재정의하지 않을 경우(또는 필요없는데 재정의 할 경우) 당연하게 사용했던 자료구조들도 모두 제대로 동작하지 않을 수 있다.  

예시로 봤던 Integer, String과 같은 값클래스 외에 객체의 주소값만으로 각 인스턴스를 구별할 수 있다면 굳이 건드리지 않는 편이 좋겠다.   
엄밀히 따지면 equals메서드 하나만 재정의하면 끝나는 것도 아닐 뿐더러(책에서는 hashCode 를 함께 재정의 해야한다고 말한다. 관련 내용으로 참고링크를 첨부한다.) 재정의로 인해 너무도 많은 영역에 영향이 가기 때문이다.  

<hr>


[참고 자료]
- [Collection 인터페이스 공식문서](https://docs.oracle.com/javase/8/docs/api/index.html?overview-summary.html)
- [equals와 hashCode 재정의 관련(stackoverflow)](https://stackoverflow.com/questions/2265503/why-do-i-need-to-override-the-equals-and-hashcode-methods-in-java)
- [String equals 메서드 코드 해석 참고_블로그](https://velog.io/@musimco/equals-%EB%A9%94%EC%86%8C%EB%93%9C%EB%A5%BC-%EA%B9%8C%EB%B3%B4%EC%9E%90)