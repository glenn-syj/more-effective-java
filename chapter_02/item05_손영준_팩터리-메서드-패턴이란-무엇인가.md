## 팩터리 메서드 패턴이란 무엇인가

### 정리

**내용 요약**

싱글턴 패턴은 상황에 따라 여러 다른 자원을 가용하는 인스턴스를 이용해야 할 때 효율적이지 않다. 게다가 기존의 `final` 제한자를 삭제하고, `instance`를 교체하는 메서드를 추가하더라도, 멀티스레드 환경에서는 해당 메서드를 이용할 수 없다. 즉, 이용하는 자원에 따라 다른 동작을 요구하는 클래스에는 싱글턴 패턴이 적절하지 않다.

저자는 이에 대한 대안으로 의존 객체 주입을 제시한다. 정확히는 생성자에 대한 의존 객체 주입이다. 이는 불변을 보장해 동기화를 이용할 수 있게 하는 방식이기도 하다. 클래스는 유연성, 재사용성, 테스트 용이성에 대해 개선된다.

생성자에서 자원팩터리를 이용하는 팩터리 메서드 패턴(factory method pattern) 역시 의존 객체 주입의 변형이기도 하다. 여기에서 팩터리에 `Supplier<T>`를 입력으로 받으면, 제네릭 하위 타입에 대해 유연하게 생성하는 팩터리를 넘길 수 있다. 게다가 팩터리는 단순히 생성자에만 주입될 수 있는 것은 아니고, 정적 팩터리나 빌더에도 주입될 수 있다.

하지만 대규모 프로젝트에서는 단순히 생성자에 주입되는 의존성이 코드 가독성을 떨어지게 하고, 의존 관계의 파악을 어렵게 만들기도 한다. 따라서 의존성 주입 프레임워크 이용이 요구되기도 한다. 

### 심화 탐구

**출발점**

정적 팩터리 메서드와 팩터리 메서드 패턴은 분명히 다르다. 전자는 인스턴스를 반환하는 방식에 대한 서술이고, 후자는 디자인 패턴의 일부다. 그렇다면 팩터리 메서드 패턴은 무엇인가? 팩터리 메서드 패턴은 어떠한 방식으로 의존성 주입과 관계를 맺고 있는가?

**설명**

1. 팩터리 메서드 패턴이란?

팩터리 메서드 패턴은 객체 생성에서 상위 인터페이스 혹은 추상 클래스를 이용하지만, 클래스 인스턴스화는 하위 클래스에서 결정하도록 하는 패턴을 의미한다. 이는 곧 생성되는 인스턴스의 타입이 하위 클래스에서 오버라이드된 메서드를 통해 결정된다는 뜻이기도 하다. 즉, 팩터리 메서드 패턴은 팩터리  상속 개념에 기대고 있다.

2. 팩터리 메서드 패턴에 대한 예시

```java
// https://en.wikipedia.org/wiki/Factory_method_pattern#cite_note-2

public abstract class Room {
    abstract void connect(Room room);
}

public class MagicRoom extends Room {
    public void connect(Room room) {}
}

public class OrdinaryRoom extends Room {
    public void connect(Room room) {}
}

public abstract class MazeGame {
     private final List<Room> rooms = new ArrayList<>();

     public MazeGame() {
          Room room1 = makeRoom();
          Room room2 = makeRoom();
          room1.connect(room2);
          rooms.add(room1);
          rooms.add(room2);
     }

     abstract protected Room makeRoom();
}

```

위 코드에서 `MazeGame` 생성자는 생성 단계들에 대해 스켈레톤을 제시하는 템플릿 메서드다. `MazeGame`은 `Room`들을 이용하지만, `Room` 인스턴스 생성은 하위 클래스에 위임한다. 여기에서 `makeRoom` 팩터리 메서드는 캡슐화되어, 하위 클래스에서 이용할 수 있다. `protected`는 오직 상속 관계 하위 클래스에서의 사용을 허용함을 떠올려 보자면 이해가 쉬울 것이다.

그렇다면 위 코드에서 `magicRoom` 인스턴스를 갖는 다른 클래스의 구현을 위해서는 `makeRoom()` 메서드를 오버라이드해 작성해야 한다. 반대로 `OrdinaryRoom` 인스턴스를 가지기 위해서는 `makeRoom()` 메서드가 해당 클래스 인스턴스를 반환할 필요가 있다.

```java

// https://en.wikipedia.org/wiki/Factory_method_pattern#cite_note-2

public class MagicMazeGame extends MazeGame {
    @Override
    protected MagicRoom makeRoom() {
        return new MagicRoom();
    }
}

public class OrdinaryMazeGame extends MazeGame {
    @Override
    protected OrdinaryRoom makeRoom() {
        return new OrdinaryRoom();
    }
}

MazeGame ordinaryGame = new OrdinaryMazeGame();
MazeGame magicGame = new MagicMazeGame();

```

3. 팩터리 메서드 패턴의 장점

팩토리 메서드 패턴은 확장에는 열려 있고, 수정에는 닫혀 있어야 한다는 원칙을 잘 지킨다. 바로 위 스니펫에서 우리가 `MazeGame`을 상속하는 새로운 클래스 `DarkMazeGame`을 만들고 싶다고 가정해보자.

```java

public class DarkRoom extends Room {
    public void connect(Room room) {}
}

public class MagicMazeGame extends MazeGame {

    @Override
    protected DarkRoom makeRoom() {
        return new DarkRoom();
    }

}

```

새롭게 `DarkRoom` 클래스를 작성하고, `makeRoom()`가 `DarkRoom` 인스턴스를 반환하도록 하면 끝이다. 즉, 팩터리 메서드 패턴을 이용하면 기존 코드를 수정하거나 복제하지 않고도 새로운 `Room` 하위 클래스를 이용하는 `MazeGame` 구현체를 생성할 수 있다. 이는 `MazeGame`이 생성자 내에서 쓰이는 `Room` 하위 클래스들의 구체적인 정보는 알 필요가 없는 까닭이다. 즉, 코드 내 클래스 결합도가 낮아진다.


### Reference

https://en.wikipedia.org/wiki/Factory_method_pattern

https://home.csulb.edu/~pnguyen/cecs277/lecnotes/factory.pdf