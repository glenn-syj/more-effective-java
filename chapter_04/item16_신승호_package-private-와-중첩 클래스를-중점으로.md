## package-private 와 중첩 클래스

### 정리

**내용 요약**

자바의 객체지향의 특징 중 하나인 캡슐화를 위해서는 외부에서 데이터 필드에 직접 접근하는 것을 막아야 한다. class을 선언할 때는 public으로 해야 하기 때문에 직접 데이터 필드에 접근하는 것을 막기 위해서는 2가지 방법이 있다. 

필드값은 private로 선언하여 getter와 setter을 통해 간접적으로 접근할 수 있게 하거나 public final로 지정하여 접근은 가능하나 수정을 할 수 없게 하여 직접 노출하였을 때의 단점을 줄일 수 있다.

**public class+public  필드**

```java
package item16;

public class Point {
	public int 가로;
	public int 세로;

	Point(int x, int y){
		this.가로=x;
		this.세로=y;
	}
}

package item16;

public class Test {
	
	public static void main(String[] args) {
		Point point= new Point(10,20);
		
		System.out.println(point.가로);
        // point에 담긴 데이터가 보호되지 않아 직접 접근 가능하여 캡슐화가 되지 않는다.
		
	}

}
```

**public class+ private 필드**

```java
public class Point {
	private int 가로;
	private int 세로;

	Point(int x, int y){
		this.가로=x;
		this.세로=y;
	}

	public int get가로() {
		return 가로;
	}

	public void set가로(int 가로) {
		this.가로 = 가로;
	}
}

```

**public class+ public final 필드**

```java
class Point {
	public final int 가로;
	public final int 세로;

	Point(int x, int y){
		this.가로=x;
		this.세로=y;
	}


}

```

하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 문제가 없다. 



### 심화 탐구

**출발점**

package-private 클래스 혹은 private 중첩 클래스의 경우 데이터 필드가 노출된다 하여도 문제가 없다고 하였는데 두 클래스에 대해 알아보고 데이터 필드가 노출되었을 때 왜 문제가 되지 않는지 알아보겠다.

**설명**

1. package-private 클래스

- package-private 클래스란?

    package-private라는 말에 생소한 사람도 있겠지만 사실 접근 제어자중 하나인 default이다.

    default 접근자는 아무것도 붙이지 않는 기본 접근제어자 (package-private)의 경우 패키지 내부에서는 `public` 키워드와 다르지 않지만, 패키지 외부에서는 자유롭게 접근하지 못하게 한다.

    **package-private 예시 코드**

    ``` java
    package item16;
    
    class pack_Test{
    	public int 대각선;
    }
    
    public class Point {
    	private int 가로;
    	private int 세로;
    
    	public Point(int x, int y){
    		this.가로=x;
    		this.세로=y;
    	}
    
    
    }
    // 다른 페키지의 Test2 생성 
    package package_privatetest;
    
    import item16.Point;
    
    public class Test2 {
    	public static void main(String[] args) {
    		Point a= new Point(10,20);
    		pack_Test b= new pack_Test();
            // pack_Test cannot be resolved to a type라는 오류가 뜬다.
    		
    	}
    
    }
    ```

    위와 같이 public으로 만들어진 Point 클래스는 외부의 패키지에서 자유롭게 사용 가능하지만 Point 내부에서 선언한 default pack_Test 클래스는 외부의 패키지에서는 접근할 수 없기 때문에 public으로 필드 변수를 선언하여도 문제가 없다.

    캡슐화를 하지 않았을 때의 문제인 클라이언트가 내부 데이터에 직접 접근할 수 있다는 문제점이 default 선언함으로써 다른 패키지에서는 접근을 완전히 차단하기 때문이다.

    이러한 default 선언 방식은 내 객체의 메서드를 테스트 할때와 api의 구현 세부 사항을 숨기고 싶을때 사용된다. 

    

- 중첩 클래스란? 
    
    java에서는 다른 클래스 내에 클래스를 정의할 수 있다. 이러한 클래스를 중첩 클래스라고 한다.
    
    ```java
    클래스 외부 클래스 {
        ...
        클래스 NestedClass {
            ...
        }
    }
    ```
    
    
    
    중첩 클래스는 가장 외각에 있는 부분인 외부 클래스 그리고 외부 클래스의 내부에 있는 내부 클래스로 나누어 지며 내부 클래스는 선언되 위치, static 키워드 유뮤에 따라 4가지로 분류 된다.  (인스턴스 클래스, 스태틱 클래스, 지역 클래스, 익명 클래스 등으로 분류 되며 이번 글에서는 중첩 클래스가 왜 데이터 필드가 노출되었을 때 왜 문제가 되지 않는지에 대한 탐구 이므로 각각의 특성을 자세히 다루지는 않겠다.)
    
    ```java
    클래스 외부 클래스 {
        ...
        클래스 내부 클래스 {
            ...
        }
        정적 클래스 StaticNestedClass {
            ...
        }
    }
    ```
    
    
    
    내부 클래스의 경우 내부 클래스에 private 제어자를 적용해줌으로써, 캡슐화를 통해 클래스를 내부로 숨길 수있다.
    즉, 캡슐화를 통해 외부에서의 접근을 차단하면서도, 내부 클래스에서 외부 클래스의 멤버들을 제약 없이 쉽게 접근할 수 있어 구조적인 프로그래밍이 가능해 진다. 그리고 클래스 구조를 숨김으로써 코드의 복잡성도 줄일 수 있게 되는 것이다. 
    
    ```java
    class Creature {
        private int life = 50;
    	
        // private class 로, 오로지 Creature 외부 클래스에서만 접근 가능한 내부 클래스로 설정
        private class Animal {
            public String name = "호랑이";
    
            int getOuter() {
                return life; // 외부 클래스의 private 멤버를 제약 없이 접근 가능
            }
        }
    
        public void method() {
            Animal animal = new Animal(); 
    
            // Getter 없이 내부 클래스의 private 멤버에 접근이 가능 하지만
            // 위의 예시코드는 public으로 지정하여 데이터 필드가 노출 되었을때도
            // 문제가 되지 않는것을 보여주기 위해 public으로 하였다.
            System.out.println(animal.name); // 호랑이
    
            // 내부 클래스에서 외부 클래스이 private 멤버를 출력
            System.out.println(animal.getOuter()); // 50
        }
    }
    출처: https://inpa.tistory.com/entry/JAVA-☕-내부-클래스Inner-Class-장점-종류 [Inpa Dev 👨‍💻:티스토리]
    ```
    
    이또한 Animal class가 private 로 되어 있기 때문에  public String name 가 public 으로 되어 있어도 직접 접근하는것을 막을수 있기 때문에 데이터 필드가 노출되었을 때 문제가 되지 않는다.
    
    
    
    **참조**
    
    https://dev.to/cauchypeano/why-you-should-avoid-package-private-scope-in-java-anl
    https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html
    
    https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4Inner-Class-%EC%9E%A5%EC%A0%90-%EC%A2%85%EB%A5%98
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    

