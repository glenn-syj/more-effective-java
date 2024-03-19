## 빌더 패턴 연습하기

`Effective Java` 2장 객체 생성과 파괴 - '아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라' 를 읽고

### 정리

**내용 요약**

이 장은 아이템 제목 자체가 한 장의 훌륭한 요약이라고 할 수 있다.

0. 정적 팩터리와 생성자 제약사항
   - 선택적 매개변수가 많을 시 적절한 대응이 어렵다.

1. 점층적 생성자 패턴
   - 프로그래머들은 이럴 때 오버로딩 방식으로 점층적 생성자 패턴을 즐겨 사용한다고 한다.
   - 다만 이 역시 매개변수 개수가 많아지면 코드 가독성이 나빠진다. (각 값의 의미가 무엇인지, 매개변수 갯수 등에 주의를 기울여야 함)

2. 자바빈즈 패턴
   - 매개변수가 없는 생성자로 객체를 만든 수, 세터(setter)메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다.
   - 코드가 길어져 보이더라도 상대적으로 가독성이 좋아진다.
   - 다만 객체 하나를 만들기 위해 여러 메서드를 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다.

3. 빌더 패턴
   - 두 패턴에서 안전성과 가독성 등 장점만 취해 만든 패턴이다.
   - 객체 생성 방법
     - 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자 · 정적 팩터리를 호출해 **빌더 객체 생성**
     - 빌더가 제공하는 일종의 세터 메서드들로 원하는 선택 **매개변수들을 설정**
     - 매개변수가 없는 build 메서드를 호출해 **객체 생성**
       - 이 때 세터 메서드들은 연쇄적으로 호출 가능하여 이런 방식을 *플루언트 API* 또는 *메서드 연쇄* 라 한다.
   - 계층적으로 설계된 클래스와 함께 사용하기에 좋으며 유연하다는 장점이 있다.
   - 다만 객체 생성 이전에 그에 앞서 빌더부터 생성해야 하기에, (빌더 생성 비용이 크지 않음에도 불구하고) 성능에 민감한 상황에서는 문제가 될 수 있다.


### 심화 탐구

**출발점**

위에서 등장한 개념들에 대해 다시 한 번 정리하고, 수제버거를 예로 들어 예시 코드를 작성하며 이해를 해보고자 한다.

**설명**

0. 빌더 패턴 (Builder Pattern)이란?
   - 한 객체의 생성 과정과 표현 방법을 분리하여 다양한 구성의 인스턴스를 만드는 생성 패턴이다.
   - 구체적으로는 생성자에 들어갈 매개 변수를 메서드로 하나하나 받아들이고 마지막에 통합 빌드해서 객체를 생성하는 방식이다.
   - 수제버거로 예시를 들자면, 주문 시 빵 · 패티는 필수겠지만, 속재료들은 주문하는 사람의 마음대로 결정된다. 누군가는 치즈, 누군가는 토마토를 빼 달라고 할 수 있다. 
       - 이처럼 보다 유연하게 재료들을 넣어 다양한 타입의 객체를 생성할 수 있도록, 클래스의 선택적 매개변수가 많은 상황에서 빌더 패턴이 유용하게 사용된다.
   - 탄생 배경 : **점층적 생성자 패턴** → **자바 빈즈 패턴**을 거치며 **단점을 보완하여 탄생되었다.**


1. 점층적 생성자 패턴 (Telescoping Constructor Pattern)
   - 필수 매개변수와 함께 선택 매개변수를 0개, 1개, 2개 .. 받는 형태로, 다양한 매개변수를 입력받아 객체 생성을 원할 때, 사용하던 생성자를 **오버로딩** 하는 방식이다.

   ```java
   class Hamburger {
       // 필수 매개변수
       private int bun;
       private int patty;

       // 선택 매개변수
       private int cheese;
       private int lettuce;
       private int tomato;
       private int bacon;

       public Hamburger(int bun, int patty, int cheese, int lettuce, int tomato, int bacon) {
           this.bun = bun;
           this.patty = patty;
           this.cheese = cheese;
           this.lettuce = lettuce;
           this.tomato = tomato;
           this.bacon = bacon;
       }

       public Hamburger(int bun, int patty, int cheese, int lettuce, int tomato) {
           this.bun = bun;
           this.patty = patty;
           this.cheese = cheese;
           this.lettuce = lettuce;
           this.tomato = tomato;
       }


       public Hamburger(int bun, int patty, int cheese, int lettuce) {
           this.bun = bun;
           this.patty = patty;
           this.cheese = cheese;
           this.lettuce = lettuce;
       }

       public Hamburger(int bun, int patty, int cheese) {
           this.bun = bun;
           this.patty = patty;
           this.cheese = cheese;
       }

       ...
   }
   ```
   ```java
   public static void main(String[] args) {
       // 모든 재료가 있는 햄버거
       Hamburger hamburger1 = new Hamburger(2, 1, 2, 4, 6, 8);

       // 빵과 패티 치즈만 있는 햄버거
       Hamburger hamburger2 = new Hamburger(2, 1, 1);

       // 빵과 패티 베이컨만 있는 햄버거
       Hamburger hamburger3 = new Hamburger(2, 0, 0, 0, 0, 6);
   }
   ```
   - 하지만 이러한 방식은 몇번째 인자가 어떤 필드였는지 혼동하기 쉽고, Hamburger 생성자의 몇번째 인수가 양상추의 갯수인지 토마토의 갯수인지 파악할 필요가 있다.
   - 같은 자료형에 값을 뒤바꾸어 넣어버릴 경우, 런타임 에러도 나타나지 않으므로 디버깅이 매우 어려울 것이라 예상된다.
   - 매개변수 특성상 순서를 따라야 하기 때문에 위의 '빵과 베이컨만 있는 햄버거'를 원할경우 억지로 파라미터에 0을 전달해야 된다. 생성자로만으로는 필드를 선택적으로 생략할 수 있는 방법이 없기 때문이다.
   - 타입이 다양할수록 생성자 메서드 수가 기하급수적으로 늘어나 가독성이나 유지보수 측면에서 좋지 않다. (위의 … 으로 생략된 부분에 다양한 생성자가 추가될 수 있다.)


2. 자바 빈즈(Java Beans) 패턴
   - 이러한 단점을 보완하기 위해 Setter 메소드를 사용한 자바 빈즈 패턴이 고안되었다. 
   - 매개변수가 없는 생성자로 객체를 생성 후 Setter 메소드를 이용해 클래스 필드의 초깃값을 설정하는 방식으로 지금까지 SSAFY에서의 자바 실습에도 가장 자주 사용했던 방식이다.

    ```java
    class Hamburger {
        // 필수 매개변수
        private int bun;
        private int patty;

        // 선택 매개변수
        private int cheese;
        private int lettuce;
        private int tomato;
        private int bacon;
        
        public Hamburger() {}

        public void setBun(int bun) {
            this.bun = bun;
        }

        public void setPatty(int patty) {
            this.patty = patty;
        }

        public void setCheese(int cheese) {
            this.cheese = cheese;
        }

        public void setLettuce(int lettuce) {
            this.lettuce = lettuce;
        }

        public void setTomato(int tomato) {
            this.tomato = tomato;
        }

        public void setBacon(int bacon) {
            this.bacon = bacon;
        }
    }
    ```

    ```java
    public static void main(String[] args) {
        // 모든 재료가 있는 햄버거
        Hamburger hamburger1 = new Hamburger();
        hamburger1.setBun(2);
        hamburger1.setPatty(1);
        hamburger1.setCheese(2);
        hamburger1.setLettuce(4);
        hamburger1.setTomato(6);
        hamburger1.setBacon(8);

        // 빵과 패티 치즈만 있는 햄버거
        Hamburger hamburger2 = new Hamburger();
        hamburger2.setBun(2);
        hamburger2.setPatty(1);
        hamburger2.setCheese(2);

        // 빵과 패티 베이컨만 있는 햄버거
        Hamburger hamburger3 = new Hamburger();
        hamburger3.setBun(2);
        hamburger3.setPatty(1);
        hamburger3.setBacon(8);
    }
    ```

    - 전 방식에서 보였던 가독성 문제점이 사라지고, 각 매개변수에 해당되는 Setter 메서드를 호출함으로써 유연적으로 객체 생성이 가능해졌다. 
    - 다만 이러한 방식은 객체 생성 시점에 모든 값들을 주입하지 않으므로 일관성(consistency) 문제와 불변성(immutable) 문제가 나타날 수 있다.
      - 일관성 : 필수 매개변수란 객체가 초기화 될 때 반드시 설정되어야 하는 값이다. 하지만 개발자가 실수로 setBun() 이나 setPatty() 메서드를 호출하지 않았다면 이 객체는 일관성이 무너진 상태가 된다. (즉, 객체가 유효하지 않은 것)
      - 불변성 : Setter 메서드는 객체를 처음 생성 시 필드값 설정을 위해 존재하는 메서드지만, 객체를 생성했음에도 여전히 외부적으로 Setter 메소드를 노출하고 있으므로, 협업 과정에서 누군가 Setter 메서드를 호출해 함부로 객체의 필드값을 변경 가능하다.



3. 빌더 패턴 (Builder Pattern)
   - 이러한 문제들을 해결하기 위해 별도의 `Builder` 클래스를 만들어 메소드를 통해 step-by-step으로 값을 입력받은 후에 최종적으로 `build()` 메소드로 하나의 인스턴스를 생성하여 리턴하는 패턴이 빌더 패턴이다.

   - 사용법: 빌더 클래스의 메서드를 체이닝(Chaining) 형태로 호출함으로써 자연스럽게 인스턴스를 구성하고 마지막에 `build()` 메서드를 통해 최종적으로 객체를 생성한다.

    ```java
    public static void main(String[] args) {

        // 생성자 방식
        Hamburger hamburger = new Hamburger(2, 3, 0, 3, 0, 0);

        // 빌더 방식
        Hamburger hamburger = new Hamburger.Builder(10)
            .bun(2)
            .patty(3)
            .lettuce(3)
            .build();
    }
    ```
    - 더 이상 생성자 오버로딩 열거를 하지 않아도 되며, 데이터의 순서에 상관없이 객체를 만들어내 생성자 인자 순서를 파악할 필요도 없고 잘못된 값을 넣는 실수도 하지 않게 된다. 
    - 즉 점층적 생성자 패턴과 자바빈즈 패턴 두 가지의 장점만을 취했다.


**추가적으로 공부해 보고 싶은 개념**

읽으면서 바로 이해하지 못해 추후 다시 정독해 보고 싶은 개념들을 적었다.

- 계층적으로 설계된 클래스
- 추상 클래스
- 셀프 타입 관용구
- 공변 반환 타이핑 (covariant return typing)
- 플루언트 API(fluent API), 메서드 연쇄(method chaining)



**참고 자료**

*https://en.wikipedia.org/wiki/Builder_pattern*

*https://refactoring.guru/design-patterns/builder*

*https://softinbit.medium.com/builder-design-pattern-constructing-complex-objects-with-ease-61bce2df3135*

*https://stackoverflow.com/questions/29881135/difference-between-builder-pattern-and-constructor*

*https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EB%B9%8C%EB%8D%94Builder-%ED%8C%A8%ED%84%B4-%EB%81%9D%ED%8C%90%EC%99%95-%EC%A0%95%EB%A6%AC*


