## readObject 메서드에서 재정의 가능한 함수를 호출해서는 안되는 것에 대해.

### 정리

**내용 요약**

저자는 상속을 하기 위해서는 그에 따른 설계와 문서화가 필요하다고 말한다.

이러한 상속을 위한 설계와 문서화를 지키기 위해서는 세 가지 방법이 있다.

---

+ 첫 째, __메서드를 재정의 할 경우에 어떤 일이 발생하는지 문서화 해야 한다.__

내부 구현 방식을 설명하여야, 설계한 클래스를 다른 프로그래머가 사용하기 위해 상속받으려 할 때, 안전하게 상속 받을 수 있다.

그러나, 내부 매커니즘을 문서로 남기는 것만이 안전한 상속을 위한 설계의 전부는 아니다.

클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅을 잘 선별하여, Protected 메서드 형태로 공개하는 방법도 존재한다.

하지만 Protected 메서드 형태로 공개할 메서드는 어떻게 '잘' 선별할 수 있을까?

이것의 유일한 방법은 바로 __실제 하위 클래스를 만들어서 시험해보는 것__ 이다.

저자가 구체적으로 추천하는 시험 방법은 대략 하위 클래스 3개를 이용하여 검증하고 이 중 하나 이상은 제 3자가 작성해보도록 하는 것이다.

문서화한 내부 사용 패턴과 Protected 메서드와 필드에 대한 결정은 생성한 상속용 클래스에게 영구적으로 영향을 미친다.

---

+ 둘 째, 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.

하위 클래스를 호출하면 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로, 

먼저 실행되는 상위클래스의 생성자 내부에 하위 클래스에서 재정의한 메서드가 호출되고, 

해당 재정의 메서드가 하위 클래스에서 초기화한 값에 의존하고 있다면,

의도대로 동작하지 않거나 에러가 발생할 것이다.

---

+ 셋 째, 상속용으로 설계하지 않은 클래스의 상속을 금지한다.

일반적인 구체 클래스, 즉 final도 아니며 상속용으로 설계되거나 문서화되지 않은 클래스를 상속받으면,

해당 일반적인 구체 클래스가 변화할 때마다, 하위 클래스가 의도와는 다른 동작을 하거나, 에러가 발생할 것이다.

---

### 심화 탐구


**출발점**

본문, '상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.' 라는 것을 저자가 설명한 이후에,

 clone과 readObject 메서드 또한 생성자와 비슷한 효과(새로운 객체를 만드는)를 내기 때문에, 생성자처럼 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다고 말하고 있다.

clone 메서드의 경우, 복제본의 상태를 수정하기도 전에 재정의 메서드를 호출하게 되어 의도와는 다른 동작을 불러일으키거나 예외를 발생시키게 되며,

readObject 메서드의 경우, 하위 클래스의 상태가 전부 역직렬화 되기 이전에 재정의한 메서드를 호출하게 되어 예외를 발생시키게 된다.

clone 메서드는 이전에 깊게 탐구한 경험이 있기 때문에, 이러한 설명이 이해가 되었으나, readObject 메서드의 경우는 개념에 대한 이해도 부족하기에 위 설명이 와닿지 않았다.

따라서 직렬화 역직렬화란 무엇인지, readObject는 무엇인지를 살펴보고, readObject 메서드가 재정의 가능 메서드를 호출해서는 안되는 이유에 대해 깊이 있게 이해해보도록 하겠다.

---



**설명**



직렬화란 객체 또는 데이터를 다른 환경에서도 사용할 수 있도록 해당 객체 또는 데이터를 바이트 스트림 형태의 연속적인 데이터로 변환하는 것을 말한다.

바이트 스트림은 기본단위를 byte로 두고 있는 입출력 통로를 말한다. 따라서 바이트 스트림 형태의 연속적인 데이터로 변환한다는 말은 바이트 스트림을 통해 전송할 수 있는 1byte 형태의 값으로 변환함을 의미한다.

또한 직렬화의 반대 개념인 역직렬화는 바이트를 원래의 객체 또는 데이터로 변환하는 것을 말한다.

직렬화 가능한 객체를 가져와서 직렬화 하는 것, 즉 바이트 스트림의 형태로 변환해주는 메서드가 'writeObject'이고,

반대로 바이트 스트림 형태를 가져와 객체 또는 데이터로 변환해주는 메서드가 'readObject'이다.

그렇다면 readObject 메서드가 재정의 가능 메서드를 호출하면 안되는 이유를 살펴보자.

바이트 스트림 값을 가지고 기존의 객체 또는 데이터를 생성하는 readObject 내부에 하위 클래스에서 재정의한 메서드가 있다면,

readObject 메서드를 통해 하위 클래스를 역직렬화 하려고 하지만, 역직렬화가 완료되기 이전에 하위 클래스에서 재정의한 메서드가 호출되는 것이고,

이 때 이 재정의한 메서드가 하위 클래스의 초기화한 값을 의존하고 있다면 완전히 역직렬화가 되지 않은 상태의 하위 클래스에서 초기화한 값을 전달해주지 못해 예외가 발생하게 되는 것이다.

아래의 예제 코드를 보면 이해가 쉬울 것이다.


원래라면 하위 클래스의 변수인 '송창용'이 출력되야 하지만,
역직렬화가 끝나기전에 재저의한 메서드가 호출되면서 null값이 출력된다.

```java
package test;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

class SuperClass implements Serializable {
    private int data;

    public SuperClass(int data) {
        this.data = data;
    }

    // 하위 클래스에서 재정의될 메서드
    protected void postDeserialize() {
        System.out.println("상위 클래스에서 호출");
    }

    private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        ois.defaultReadObject();
        postDeserialize(); // 역직렬화 후 재정의한 메서드 호출
    }
}

class SubClass extends SuperClass {
    private String additionalData;

    public SubClass(int data, String additionalData) {
        super(data);
        this.additionalData = additionalData;
    }

    // 슈퍼 클래스의 postDeserialize 메서드를 오버라이드
    @Override
    protected void postDeserialize() {
        System.out.println("SubClass: postDeserialize method");
        System.out.println("하위 클래스의 변수 : " + additionalData);
    }
}

public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        SubClass subOrigin = new SubClass(7, "송창용");

        // 객체를 파일에 직렬화
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("data.bin"));
        out.writeObject(subOrigin);
        out.close();

        // 파일에서 객체 역직렬화
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("data.bin"));
        SubClass objDeserial = (SubClass) in.readObject();
        in.close();
    }
}
```

테스트를 위한 Main 클래스의 마지막 줄에서 역직렬화가 일어난다.
`SubClass objDeserial = (SubClass) in.readObject();`

그러나 하위 클래스가 완전히 생성되어, additionalData 멤버 변수를 초기화하기 이전에 postDeserialize 메서드가 호출되어 버리게 된다.

따라서, 출력은 다음과 같다.

```
SubClass: postDeserialize method
Additional Data: null
```


---

**어려웠던 점**

readObject 메서드가 하위 클래스를 역직렬화를 하려고 하지만 역직렬화 완료 이전에 하위 클래스에서 재정의한 메서드가 호출되는 경우를 예제 코드로 표현해보고 싶었지만, Serialization에 대한 이해가 부족하여 스스로 만들지 못하였다.

타인의 코드를 참고해서라도 예제 코드를 첨부하려 했으나, 이런 상황을 나타내는 코드를 찾아낼 수 없었다.

~~추후에 해당 상황을 표현하는 코드를 찾아내어 참고를 하거나 또는 Serialization에 대하여 더 깊게 이해한 이후, 스스로 예제 코드를 생성하여 첨부하도록 하겠다.~~



+ 스스로 예제 코드를 만들어보고 싶었으나, 시간이 오래 걸리고 쉽지 않았기에 chatGpt를 이용하여 예제 코드를 만들었다. 

---




Reference:
https://www.baeldung.com/java-serialization
