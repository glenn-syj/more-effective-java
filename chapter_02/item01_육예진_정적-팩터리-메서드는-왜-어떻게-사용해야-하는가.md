## 정적 팩터리 메서드는 왜, 어떻게 사용해야 하는가

### 정리

**내용 요약**

클래스의 인스턴스를 얻는 수단으로는 <u>public 생성자</u>와 <u>정적 팩터리 메서드</u>가 있다.  
두 가지 방법을 사용함에 있어서 각각의 장단점을 이해하고, 상황에 맞는 수단을 사용하는 것이 좋다.

정적 팩터리 메서드란?   
객체 생성 없이 접근할 수 있는 static(정적) 키워드를 사용하여, 해당 클래스의 인스턴스를 반환해주는 메서드이다.

>단순히 인스턴스를 반환하는 메서드라면, 왜 사용해야 하는가?
1. 사용자로 하여금 전달하는 매개변수를 통해 어떤 객체가 반환되는지 명시적으로 알 수 있게 한다.
   - 코드 가독성을 높이고, 유지보수에 용이하다.
2. 인스턴스의 생성을 통제할 수 있다.
    - 불필요한 중복 객체 생성을 방지하고, 성능을 향상시킬 수 있다.
3. 하위 타입에 한하여, 반환 객체의 클래스를 자유롭게 선택할 수 있다.
    - 실제 구현 클래스를 알지 못하더라도 API를 보다 다양하고 쉽게 사용할 수 있다.
4. ~~메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.~~ (탐구중)

> ❗️사용시 유의점
1. public/protected 생성자가 없는 경우(정적 팩터리 메서드만 제공하는 경우), 상속이 불가하다.
    - 의도적인 경우라면 장점이 될 수 있다.
2. 사용자 정의 메서드이기 때문에, 다른 프로그래머가 메서드의 여부와 사용법을 찾기 어렵다.
    - API문서와 메서드명 컨벤션을 통해 보완 가능하다.

[참고링크](https://velog.io/@holidenty/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EA%B0%9D%EC%B2%B4%EC%9D%98-%EC%83%9D%EC%84%B1%EA%B3%BC-%ED%8C%8C%EA%B4%B4-Item1-nul1xqx9) : 팩터리라는 패턴의 유래부터 시작해서, 이해하기 쉬운 용어들로 풀어서 해석되어 있었다.

### 심화 탐구

**출발점**

위 요약에서 정적 팩터리 메서드의 장단점에 대해 책 내용을 기반으로 이해한 내용을 정리했다.  
생성자보다 유리한 경우가 많다고는 하지만, 적절한 시점에 적절한 방식으로 사용하기 위해서는 사례 중심의 이해가 필요해 보인다.

**설명**

<hr>

1. 사용자로 하여금 전달하는 매개변수를 통해 어떤 객체가 반환되는지 명시적으로 알 수 있게 한다.

    정적 팩터리 메서드 뿐만 아니라 모든 메서드명은 명시적으로 어떠한 역할을 하는지 나타내는 것이 중요하다.  
    이름을 짓는 데에 다소 시간이 걸리더라도 추후 유지보수와 협업을 위해 코드 가독성은 언제나 고려되어야 한다.

    정적 팩터리 메서드가 생성자보다 유리한 첫번째 이유가 바로 이것이다.  
    매개변수가 필요한 생성자나, 싱글턴 등과 같은 경우(기본 생성자가 아닌 경우 대부분) 생성자는 단순히 '객체생성'의 의미만을 갖지 않는다.

    ```
    단편적인 예시로, 싱글턴을 생각해보자.  
    기본생성자와 인스턴스를 private으로 클래스 안에 감춰두고, getInstance라는 메서드를 사용해 모든 호출에서 단 하나의 인스턴스만을 사용하도록 한다.  
    이때, getInstance 라는 메서드명에서 우리는 '새로운 객체가 아닌 기존에 존재하는 인스턴스를 전달받는다'라는 것을 유추할 수 있다.
    ```

    또 다른 예시로, 생성자가 여러개일 때 모든 생성자는 각각 다른 시그니처를 갖는다. 
    
    >시그니처란? 메서드의 반환 자료형, 매개변수의 개수와 자료형/순서 이 모든것의 상태를 말한다.   
    생성자의 반환자료형은 하나이므로 여기서는 매개변수의 상태만을 이야기 한다.

    상황에 따라 다른 상태의 객체를 반환하고 싶다면 시그니처를 다르게 할 수 있지만, 결국 사용자 입장에서는 같은 이름의 생성자다.  
    어떤 매개변수를 통해 어떤 상태의 객체를 반환 받을지 유추할 수 없다. 한마디로 '매우 헷갈린다.'

    따라서 이러한 경우에 정적 팩터리 메서드를 고려해볼 수 있다.   

<hr>

2. 인스턴스의 생성을 통제할 수 있다.

    1번에서 설명한 싱글턴을 그 예시 중 하나로 이야기할 수 있다.  
    매번 새로운 객체를 생성하지 않고, 하나의 인스턴스만을 반환해가며 해당 객체를 모든 사용자가 공유한다.

    싱글턴처럼 하나의 인스턴스만 사용할 수 있는 것은 아니다.  
    인스턴스 생성 통제는 목적에 따라 다양하게 활용할 수 있다. 

    ```
    [인스턴스가 2개인 사례]
    Boolean 클래스는 각각 true와 false의 필드값을 갖는 Boolean 객체를 상수로 선언해두고,
    매개변수의 상태에 따라 해당하는 값을 반환한다. 따라서 새로운 객체는 생성하지 않는다.  
    이는 두 상수가 변하지 않는다는 특성으로 인해 가능하다. 
    ```

    ~~Enum 자료형을 활용하면 생성가능한 객체 종류를 한정해두고 필요에 따라 중복없이 인스턴스를 관리할 수 있지 않을까 생각했다..~~  
    *확인되지 않은 개인의 추축입니다. 탐구 후 확인이 되면 구체적으로 재업로드 하겠습니다.

    쓸모없거나 중복되는 객체 생성을 막음으로, 메모리를 효율적으로 사용하고 프로그램의 성능을 향상시킬 수 있다.


<hr>

3. 하위 타입에 한하여, 반환 객체의 클래스를 자유롭게 선택할 수 있다.

    실제 책에서는 세번째/네번째 장점으로 나누어 설명하고 있지만, 결국 반환 객체의 유연성으로 일맥상통하는 이야기다.
    하위 타입이라는 개념은 상속(extend)과 구현(implement)에서 사용하게 된다.  
    객체 호출 위치에서 클래스를 직접 선택하는 것이 아니라 클래스 선택과 반환 역할을 정적 팩터리 메서드에 맡기는 것이다.  
    (반면, 생성자는 본인 클래스의 인스턴스만 반환할 수 있다.)  

    단순하고 재밌게 흐름을 이해하기 위해 한 예시를 찾았다.
    [코드 출처](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%83%9D%EC%84%B1%EC%9E%90-%EB%8C%80%EC%8B%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90)

    ```java
    interface SmarPhone {
        public static SmarPhone getSamsungPhone() {
            return new Galaxy();
        }

        public static SmarPhone getApplePhone() {
            return new IPhone();
        }

        public static SmarPhone getChinesePhone() {
            return new Huawei();
        }
    }
    
    class Galaxy implements SmarPhone {}
    class IPhone implements SmarPhone {}
    class Huawei implements SmarPhone {}
    ```
    위 코드에서 SmartPhone의 구현체들을 인터페이스에서 직접 반환하는 것을 볼 수 있다.  
    이를 각각의 메서드가 아니라 하나의 메서드에서 매개변수에 따라 선택하도록 할 수도 있다.

    아래 예시를 보자.
    ```java
    interface SmarPhone {
        public static SmarPhone getPhone(int price) {
            if(price > 100000) {
                return new IPhone();
            }

            if(price > 50000) {
                return new Galaxy();
            }

            return new Huawei();
        }
    }
    ```
    가격에 따라 해당하는 종류의 스마트폰 구현체를 반환한다.   
    스마트폰마다 기능이 조금씩 다를 것이다.   
    사용자는 각 스마트폰이 어떤 기능을 갖는지 구체적으로 알아보고 원하는 객체를 얻을 수도 있겠지만,  
    그냥 예산에 맞는 새로운 스마트폰이 필요할 뿐이라면 어떤 스마트폰이 적절한지 선택하는 일은 인터페이스에 맡겨버릴수도 있는 것이다.

    유사한 다른 예시로 상상해보자.  
    만약 점수에 따라 제공되는 서비스나 허용되는 권한이 다른 객체(등급)가 설정된 회원카드를 부여한다고 가정한다.  
    사용자는 똑같이 생긴 회원카드를 받았을 뿐, 자신의 등급이 무엇인지는 알 필요가 없다.  
    회원카드를 사용하면서 권한이 허용되는 한에서만 서비스를 이용하면 되는 것이다.

    ```java
    interface Grade {
        String toText();
    }

    class A implements Grade {
        @Override
        public String toText() {return "A";}
    }

    class B implements Grade {
        @Override
        public String toText() {return "B";}
    }

    class C implements Grade {
        @Override
        public String toText() {return "C";}
    }

    class D implements Grade {
        @Override
        public String toText() {return "D";}
    }

    class F implements Grade {
        @Override
        public String toText() {return "F";}
    }
    ```
    위 코드는 등급을 나타내는 인터페이스와 각 등급 구현체를 보여준다.

    ```java
    class giveLevel {
        // 정적 팩토리 메서드
        public static Grade of(int score) {
            if (score >= 90) {
                return new A();
            } else if (score >= 80) {
                return new B();
            } else if (score >= 70) {
                return new C();
            } else if (score >= 60) {
                return new D();
            } else {
                return new F();
            }
        }
    }
    ```
    시스템은 각 회원의 점수에 따라 해당하는 등급을 회원카드에 설정할 것이다. 
    ```java
    public static void main(String[] args) {
        String jeff_score = giveLevel.of(36).toText();
        String herryPorter_score = giveLevel.of(99).toText();

        System.out.println(jeff_score); // F
        System.out.println(herryPorter_score); // A
    }
    ```
    등급의 종류에 대해 알지 못하더라도, giveLevel이라는 팩터리 메서드만으로 회원마다 점수에 맞는 기능을 가진 객체를 부여할 수 있는 것이다.

    정리하자면, 사용자는 구현체에 대한 지식없이도 필요에 따라 다양한 기능을 유연하게 이용할 수 있다.
    또한 제공자는 구현체를 공개하지 않고도(정보 은닉) 기능을 제공할 수 있고, 객체간 의존성을 완화하는 효과를 얻을 수 있다.