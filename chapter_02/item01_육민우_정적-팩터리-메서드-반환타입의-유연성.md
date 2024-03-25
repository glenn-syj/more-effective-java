# 정적 팩터리 메서드 반환타입의 유연성



## item01 - 생성자 대신 정적 팩터리 메서드를 고려하라

### 🔍 내용 요약

```
클래스의 인스턴스 생성하는 방법은 "생성자"가 유일하지만, 인스턴스를 반환받아 활용하는 데는 "생성자"와 더불어 "정적 팩터리 메서드(static factory method)"를 이용할 수 있다.
```
```
정적 팩터리 메서드는 클래스 내부에서 생성한 인스턴스를 반환하는 메서드로, 생성자를 통해 인스턴스를 생성해 바로 이용하는 방법 대비 몇가지 장단점이 존재하고, 이를 적절히 활용하면 생성자 대비 다양한 이점을 얻을 수 있다.
```

<br>

정적 팩터리 메서드의 장점

1. 이름을 가질 수 있다.
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

<br>

정적 팩터리 메서드의 단점

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

<br>

--------------------------------------------------

### 🧐 심화 탐구

정적 팩터리 메서드의 장점 중 3번인 "반환 타입의 하위 타입 객체를 반환할 수 있다"에서 다형성의 개념에서 조상 클래스로 형변환을 진행했던 것에 비춰보아 왜 상위 타입이 아닌 하위 타입 객체를 반환할 수 있는 것인지에 대해 탐구하는 것을 주제로 선정하였다.

먼저, 정적 팩터리 메서드는 프로그래밍 언어에 내장된 기능이 아닌 개발자가 임의로 정의한 메서드로 "인스턴스를 반환하기로 약속한 메서드"라고 이해했다. 이에 어떠한 인스턴스를 반환하는 것이지 굳이 해당 클래스 타입일 필요는 없겠구나라고 생각해서 다른 타입의 인스턴스가 올 수 있다고 생각했다. 

그러면 여기서 4가지 선택지가 나온다고 생각한다
1. 어떠한 타입의 인스턴스든지 반환 가능
2. 해당 인스턴스의 상위 타입 인스턴스를 반환 가능
3. 해당 인스턴스의 하위 타입 인스턴스를 반환 가능
4. 해당 인스턴스의 상위, 하위 타입 인스턴스 모두 반환 가능

<br>

java code
```java
package item01;

class Vehicle{
    String str = "vehicle";

    public void whatIsThis(){
        System.out.println("vehicle");
    }
}

class Car extends Vehicle{
    String str = "car";

    public void whatIsThis(){
        System.out.println("car");
    }

    public static Car getInstance(int n){
        if(n==1) return new Benz();
        else if(n==2) return new Car();
        else if(n==3) return (Car) new Vehicle();
        else if(n==4) return new Person();
        else return new Car();
    }
}

class Benz extends Car{
    String str = "benz";

    public void whatIsThis(){
        System.out.println("benz");
    }
}

class Person{
    String str = "person";

    public void whatIsThis(){
        System.out.println("person");
    }
}

public class TEST01 {
    public static void main(String[] args) {

        Car carTest1 = Car.getInstance(1);
        System.out.println(carTest1.str);
        carTest1.whatIsThis();

        Car carTest2 = Car.getInstance(2);
        System.out.println(carTest2.str);
        carTest2.whatIsThis();

        Car carTest3 = Car.getInstance(3);
        System.out.println(carTest3.str);
        carTest3.whatIsThis();
        
        Car carTest4 = Car.getInstance(4);
        System.out.println(carTest4.str);
        carTest4.whatIsThis();
    }
}

--------------------------------------------------

출력 결과
car
benz
car
car
```

위와 같이 코드를 구성해 실험을 진행했다.

### Case 1
Car클래스와 Person클래스의 반환 타입이 맞지 않아 컴파일 에러가 발생하였다. 

### Case 2
코드 단계에서 `else if(n==3) return new Vehicle`로 작성을 하였는데 type casting을 요구했고 명시적 형변환 이후 실행했는데 ClassCastException이 발생했다.

### Case 3
하위 클래스인 Benz에 대해 잘 출력이 되었다. 다만 str을 출력했을 때는 똑같이 car가 나오는데 오버라이딩한 메서드를 출력했을 때는 각각 benz와 car가 출력됐다. 첫 실험에서는 `String str`만 가지고 실험했는데 예상과 달리 car만 두 번 나와서 메서드로 다시 실험을 한 결과이고 carTest들은 인스턴스들의 상위 타입인 Car 타입으로 다형성이 적용되어 동일하게 car가 출력된 것으로 생각된다. 위 테스트 결과 정적 팩터리 메서드에서 하위 타입 인스턴스는 반환이 가능한 것으로 보인다.

main 메서드를 보며 정적 팩터리 메서드에서 하위 타입의 인스턴스를 반환할 경우 main 메서드에서 다형성을 적용할 때 자연스러운 것을 볼 수 있었다. 하위 타입의 인스턴스에 대해 상위 타입으로 묵시적 형변환을 하므로 정적 팩터리 메서드는 하위 타입의 인스턴스를 반환할 수 있다는 개념의 어색함이 많이 줄어들었다.

--------------------------------------------------

### 🧠 어려웠던 점

- 다중 상속 클래스에서 인스턴스를 생성할 때 해당 인스턴스의 자료형을 알아내는 방법을 찾는 것이 어려웠다
- 처음에 Vehicle, Car, Benz를 싱글턴 패턴으로 만드려고 했는데, 싱글턴 패턴에서 생성자에 파라미터를 넣는 싱글턴을 만드는게 상당히 어려웠고, 검색을 통해 구현했는데 파라미터가 있는 싱글턴은 싱글턴이 아니라는 글이 있어서 개념적으로 헷갈렸다(https://velog.io/@dogakday/Singleton-%ED%8C%A8%ED%84%B4%EC%97%90%EC%84%9C-%EC%9D%B8%EC%9E%90-%EC%A0%84%EB%8B%AC%ED%95%B4%EC%84%9C-%EC%B4%88%EA%B8%B0%ED%99%94)
- 싱글턴 패턴에서 필드로 int형을 넣으면 구현이 되는데, String 타입을 넣으면 에러가 나는 현상을 발견했는데 이유를 모르겠다
