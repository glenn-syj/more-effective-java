## SuppressWarnings 애노테이션에 대해

### 정리

**내용 요약**

저자는 코드를 작성함에 따라 발생하는 비검사 경고들을 최대한 제거하라고 말하며,

비검사 경고를 제거해야하는 이유, 그리고 방법과 원칙에 대해 설명하고 있다.

---


경고를 제거할 수는 없지만 타입 안전하다고 확신한다면, @SuppressWarnings("unchecked") 애너테이션을 달아서 경고를 숨길 수 있다.

그러나 타입 안전함을 검증하지 않은 채 경고를 숨기면, 런타임은 여전히 ClassCastException을 던질 수 있기 때문에 타입 안전함을 검증한뒤 애너테이션을 사용해야 한다.

---

반대로 타입 안전함이 검증되었는데도 비검사 경고를 숨기지 않고 그대로 둔다면, 다른 문제를 알리는 새로운 경고가 나와도 이전의 경고 때문에 인식하지 못할 수 있다.

---

@SuppressWarnings 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 '선언'에도 첨부할 수 있다.

그러나, @SuppressWarnings 애너테이션은 가능한 한 좁은 범위에 적용해야만 한다.

검증된 좁은 범위부터 애너테이션을 달지 않고 넓은 범위 또는 클래스 전체 범위에 @SuppressWarnings 애너테이션을 달게 되면 심각한 경고를 놓칠 가능성이 생기기 때문이다.

---

따라서, 한 줄이 넘는 메서드 또는 생성자에 달린 @SuppressWarnings 애너테이션은 지역변수의 선언으로 옮기는 것이 좋다. 이를 위해 지역 변수를 새로 선언해야 한다고 하더라도 안정성을 위해서는 좋은 선택이다.


### 심화 탐구


**출발점**

본문에서 이야기하는 @SuppressWarnings 애너테이션에 대한 개념이 생소하므로, 해당 애너테이션에 대한 개념을 탐구하여 @SuppressWarnings 애너테이션을 앞으로 어떤 곳에 사용할 수 있을지 파악하여 보겠다.


---


**설명**

#### @SuppressWarnings

+ SuppressWarnings인자로 받는 이름의 경고를 억제해준다.

본문에 나온 SuppressWarning("unchecked")는 unchecked 경고에 대한 경고를 억제해주는 것이다.

여기서 unchecked 경고란 타입 안정성을 보장할 수 없다는 것으로, 컴파일러와 런타임 시스템에 충분한 타입 정보가 없기 때문에 필요한 모든 타입 확인을 수행할 수 없다는 의미를 말한다.

자바에서는 아래와 같이 코드를 작성할 경우, unchecked 경고를 확인할 수 있다.

```java
import java.util.List;

public class Test {
    private List versions;

    public void addVersion(String version) {
        versions.add(version);
    }
}
```

```
    Type safety: The method add(Object) belongs to the raw type List. References to generic type List<E> should be parameterized
```

Test 클래스에서 private으로 선언된 List 형태의 version 멤버 변수는 타입 매개변수가 없는 제네릭 타입이기 때문에 위와 같은 경고가 나오게 된다. 

이러한 타입 매개변수가 없는 제네릭 타입인 'raw 타입'은 앞선 item 26에서 사용해선 안되는 이유를 설명하고 있기 때문에, 그것과 해당 경고를 연관지으면 이해하기 쉬울 것이다.

위의 상황에서 경고를 억제하기 위해서는 다음과 같이 애너테이션을 붙여야 한다.

```java
import java.util.List;

public class Test {
    private List versions;

    @SuppressWarnings("unchecked")
    public void addVersion(String version) {
        versions.add(version);
    }
}
```

이러한 SuppressWarnings는 한 번에 여러 경고에 대한 억제를 시행할 수도 있다.

```java
@SuppressWarnings({"unchecked", "deprecation"})
```

해당 SuppressWarnings는 같은 경고에 대한 억제를 여러번 해도 상관없다.

다만, @SuppressWarnings({"unchecked", "unchecked", "unchecked"}) 처럼 애너테이션을 작성한다면, 두 번째 unchecked 부터는 무시된다.

그러나 위처럼 작성한다면, 경고를 억제하기 위한 @SuppressWarnings가 또다른 경고를 발생시키게 되므로 권장되는 방식은 아니다.

```java
// 경고를 중복 적용했을 때.
Unnecessary @SuppressWarnings("unchecked")
```

이외에도 존재하지 않는 경고를 @SuppressWarnings 애너테이션 내부에 매개변수로 넣게 되면, 마찬가지로 경고를 억제하기 위한 @SuppressWarnings 애너테이션이 오히려 경고를 만들어내게 되니 주의해서 사용해야만 한다.

```java
// 존재하지 않는 경고를 적용했을 때.
Unsupported @SuppressWarnings("undeadtimo")
```

**느낀점**

지금까지 코드를 작성하면서 경고창 내부에 있는 내용들을 살펴보아도 이해할 수 없었으며, 굳이 이해해도 쓸모없는 내용들에 대한 것이라 생각하였기에 경고창에 대하여 신경쓰지 않았다.

그러나, item 27을 읽으며 내가 도외시하던 경고이 코드에 대하여 나에게 여러 정보를 주고 있으니, 문제가 없다고 확신이 되는 것에 대해서는 @SuppressWarnings 애너테이션을 적용해 다른 경고들을 파악하기 쉽도록 만들고 그 의미들을 하나하나 이해해야겠다.

---


Reference:

https://docs.oracle.com/javase/8/docs/api/java/lang/SuppressWarnings.html

https://www.baeldung.com/java-suppresswarnings

https://stackoverflow.com/questions/1129795/what-is-suppresswarnings-unchecked-in-java

http://www.angelikalanger.com/GenericsFAQ/FAQSections/TechnicalDetails.html#FAQ001
