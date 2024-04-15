## 인터페이스 상속과 디폴트 메서드 심화 탐구

### 정리

인터페이스 내에서의 디폴트 메서드 선언은 항상 좋은 결과만을 가지고 있는 것은 아니다. 이는 당연하게도, 인터페이스의 디폴트 메서드가 구현 클래스를 고려할 수 없기 때문이다.

주요한 예시로는 `Collection` 인터페이스에 추가된 `removeIf` 메서드가 있다. 이 메서드는 인자로 주어진 조건식을 만족하는 경우의 원소를 모두 제거한다. `removeIf` 메서드의 이용은 `Collection` 인터페이스의 구현체를 불안정하게 만들거나, 런타임에러를 일으킬 수도 있다.

자바 플랫폼 라이브러리에서는 (1) 디폴트 메서드 재정의 (2) 디폴트 메서드 호출 전 사전 작업 수행 등을 통해 예방적 조치를 취했다. 물론, 많은 구현체들은 여전히 변화하지 않은 채로 남아있다. 

따라서 인터페이스 설계는 세심해야 하며, 디폴트 메서드를 추가할 때에는 특히나 더욱 이용자 측면을 고려해야 한다. 디폴트 메서드에는 표준적인 구현에 대해 기능을 손쉽게 제공할 수 있는 이점도 있기에 더욱 그렇다. 또한, 언제나 예방이 조치보다 좋음을 잊어서는 안된다.

### 심화 탐구

**출발점**

인터페이스의 디폴트 메소드는 그렇다면 언제, 어떻게 쓸 수 있을까? 이러한 질문은 인터페이스 설계가 아닌 인터페이스 이용 측면에서도 유효허다.

언젠가 이용하고자 하는 인터페이스의 특정 디폴트 메소드가 생성할 클래스의 동작과 맞지 않는 상황을 맞닥뜨릴지도 모른다. 또는, 인터페이스를 상속하여 새로 이용해야 할 때, 목적에 맞추어 디폴트 메소드를 변경해야 할 수도 있다.

따라서 기존 인터페이스 설계 시에서의 디폴트 메서드를 고려할 만 아니라, 상속하여 이용할 때의 고민 역시 필요하다.

**설명**

자바8 공식 문서에 따르면, 디폴트 메소드는 해당 인터페이스의 구현체들에 추가적인 기능을 제공하고 싶을 때 이용할 수 있다. 동시에 인터페이스는 다른 인터페이스과 상속 관계를 맺을 수 있기도 하다. 즉, 디폴트 메소드 역시 상속될 수 있다. 

인터페이스 상속에서는 디폴트 메소드를 다음 세 가지 방식으로 이용할 수 있다.

1. 정의된 대로 이용

디폴트 메소드를 조작하지 않고 인터페이스에 정의된 채로 이용할 수 있다. 하위 인터페이스에서 디폴트 메소드에 대해 어떠한 내용도 적지 않았을 때, 해당 메소드 호출 시에 상위 인터페이스의 디폴트 메소드를 호출한다.

2. 디폴트 메서드 재선언: 추상화

해당 디폴트 메서드에 대해 이용할 계획이 없거나 이용해서는 안된다면, 추상화할 수 있다. 하위 인터페이스에서 디폴트 메서드와 동일한 이름과 반환 타입, 인자를 가진 메서드를 내용 없이 작성한다. 구현체 내에서 해당 메소드 호출 시, 메소드가 비어있으므로 어떠한 작용도 하지 않는다. 

```java
public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                               int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
    
    static ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
        
    default ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }
}
```


```java
public interface AbstractZoneTimeClient extends TimeClient {
    public ZonedDateTime getZonedDateTime(String zoneString);
}
```

코드 출처: https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html

3. 디폴트 메서드 재정의: 오버라이드

디폴트 메소드에 `@Override` 어노테이션을 붙여 재정의한다면, 상위 인터페이스가 아니라 하위 구현체의 메소드가 호출된다. 이는 일반적인 오버라이드와 같이 동작한다.

위와 같은 방식을 통해서 인터페이스 상속 시, 목적에 따라 디폴트 메소드의 사용 여부나 구체적인 내용을 다룰 수 있다.

### 더 나아가기

1. 인터페이스 다중 상속의 문제점

인터페이스에서는 클래스와 다르게 다중 상속이 가능하다. 여기에서의 상속은 implementation을 의미한다. 인터페이스의 다중 상속은 곧 동일한 이름을 가진 디폴트 메소드의 상속으로 향한다.

만약 상위 인터페이스들 사이에서 같은 이름을 가진 디폴트 메서드가 존재한다면, 모호성(ambiguity) 탓에 하위 구현체는 어떤 메서드를 호출해야 할 지 결정할 수 없다.

```java
public interface Floatable {
    default void repair() {
    	System.out.println("Repairing Floatable object");	
    }
}
```
```java
public interface Flyable {
    default void repair() {
    	System.out.println("Repairing Flyable object");	
    }
}
```
```java
public class ArmoredCar extends Car implements Floatable, Flyable {
    // this won't compile
}
```
코드 출처: https://www.baeldung.com/java-inheritance

위 예시에서, `Folatable` 인터페이스와 `Flyable` 인터페이스를 상속받은 `ArmoredCar` 클래스는 컴파일 단계에서 오류가 발생한다.

2. 오버라이드를 통한 문제 해결

이러한 문제를 해결하기 위해서는, 하위 구현체 내에서 오버라이드를 통해 새로이 메서드를 작성하거나 어떠한 디폴트 메서드를 이용할 것인지 명시할 필요가 있다. 

### References

https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html

https://turreta.com/blog/2016/11/28/java-8-redeclare-default-methods/

https://www.baeldung.com/java-inheritance