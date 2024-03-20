

## 요약

클라이언트가 클래스의 인스턴스를 얻는 전통적인 방법은 생성자를 이용하여 생성하나 정정적 팩토리 메서드라는 방식이 존재한다!

정적 팩토리 메서드란 클래스 내에 선언되어 있는 메서드를 내부의 new를 이용해 객체를 생성해 반화해주는 방법이다. 즉 정적 팩토리 메서드를 통해서 new를 간접적으로 사용하는 방법.

이러한 정적 팩토리 메서드를 생성자 대신 사용할때는 5가지 장점이 있다. 

장점

1. 이름을 가질수 있어 객체가 어떤 특징이 있는지 알 수 있다.
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있다.
4. 입력 매개변수에따라 매번 다른 클래스의 객체를 반환할수 있다
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

단점

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 펙토리 메서드만 제공하면 하위 클래스를 만들수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

## 정적 팩토리 메서드를 사용해야 할 때

1. 생성자에 넘길 매개변수가 많아서 매개변수들이 어떤 의미인지 모르니깐 메서드 이름을 통해 알릴수 있게한다.
2. 반환 타입의 하위 타입 객체를 반환할때 조건에 맞는 하위타입의 정보가 나올수 있게 할수 있다.



## 🤷‍♀️심화탐구

출발점

장점 2: 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다. 

나의 이해:

팩토리 메서드를 활용하야 new를 적게 사용하면 인스턴스를 그때마다 생성시키지 않는다는건데 미리 인스턴스들을 다 만드는 과정에서 이미 new를 한번씩 사용하기 때문에 정적 팩토리 메서드도 new를 활용해서 인스턴스를 생성해 내는데 생성자와 어떤 차이가 있다는 것인지 이해가 되지 않는다. 



따라서 인스턴스을 생성자를 통했을 때와 팩토리를 통했을때 인스턴스 생성에서 어떠한 차이가 있는지 탐구해보겠다. 



### 생산자를 통한 인스턴스 생성

``` java
public class Settings {

	public static void main(String[] args) {
		Settings set1 = new Settings();
		Settings set2 = new Settings();

		System.out.println(set1);
		System.out.println(set2);
	}

}
// 결과 
item1.Settings@5ca881b5
item1.Settings@24d46ca6

```

생성자를 통해 객체를 생성하면 계속해서 다른 인스턴스가 생성된다. 

### 정적 팩토리 메서드(싱글톤)

```java
public class Settings {
	private static final Settings instance = new Settings();

	// 싱글톤 인스턴스 생성
	private Settings() {
	};

	public static Settings getInstance() {
		return instance;
	}

	public static void main(String[] args) {
		Settings set1 = Settings.getInstance();
		Settings set2 = Settings.getInstance();

		System.out.println(set1);
		System.out.println(set2);
	}

}
item1.Settings@5ca881b5
item1.Settings@5ca881b5
```

정적 팩토리 메서드를 활용하여 싱글턴으로 새로운 객체를 생성할시 이미 생성된 인스턴스를 활용하기 때문에 새로운 인스턴스를 생성시키지 않아도 된다. 

그렇기 때문에 호출될때 마다 새로운 인스턴스를 생성하지 않아도 된다는것이다.



#### 확장 

하지만 정적 팩토리 메서드 안에 싱글톤이 들어가는 것은 맞으나 모든 정적 팩토리 메서드가 싱글톤의 특징을 가지는건 아니다. 다양한 인스턴스를 가지는 경우에 어떻게 효율적이라고 할수있을까?  

요일마다의 특징을 가진 day 인스턴스를 미리 정적 팩토리 메서드를 통해 생성해보자

```java
public class Day {

    private static final Map<String, Day> days = new HashMap<>();

    static {
        days.put("mon", new Day("Monday"));
        days.put("tue", new Day("Tuesday"));
        days.put("wen", new Day("Wednesday"));
        days.put("thu", new Day("Thursday"));
        days.put("fri", new Day("Friday"));
        days.put("sat", new Day("Saturday"));
        days.put("sun", new Day("Sunday"));
    }

    public static Day from(String day) {
        return days.get(day);
    }

    private final String day;

    private Day(String day) {
        this.day = day;
    }

    public String getDay() {
        return day;
    }
}
```

정적 팩토리 메서드를 활용

``` java
public static void main(String[] args) {
       Day day1 = Day.from("mon");
      Day day2 = Day.from("mon");
    }
```

그냥 생성자를 활용

```java
public static void main(String[] args) {
        Day day1 = new DAY();
        Day day2 = new DAY();
    }
```

월요일의 특징을 가진 day1,2 를 생성할때 정적 팩토리 메서드는 인스턴스를 

한번만 생성하면 되는걸 볼수있다.  



###  결론

정적 팩토리 메서드 와 생성자 모두 new 를 활용하여 객체를 생성한다.

하지만 정적 팩토리 메서드의 경우 인스턴스를 한번만 생성하여 그인스턴스를 활용하지만 생성자는 계속해서 새로운 인스턴스를 생성 해줘야 한다. 물론 만약 public class Day 에서 필요한 요일이 월,화,수 밖에 없었다면 단 3번의 인스턴스를 생성하면 끝인 그냥 생성자 방법이 더 효율적일 것이다. 

따라서 정적 팩토리 메서드는 여러 객체가 생성될 여지가 있는곳에서 사용되는데 대표적인 예가 스프링이다.

스프링은 싱글톤을 사용하여 여러번 빈을 요청하더라도 매번 동일한 객체를 돌려주는데 

싱글톤을 사용하지 않는다면 매번 클라이언트에서 요청이 올 때마다 각 로직을 처리하는 빈을 새로 만들어야 할것이다 따라서 빈을 싱글톤 스코프로 관리하여 1개의 요청이 왔을 때 여러 쓰레드가 빈을 공유해 처리하도록 하였다.



### 느낀점

싱글톤 패턴에 대해 자세히 공부해 본적 없었는데 싱글톤 패턴이 왜 필요하고 스프링에서는 왜 싱글톤 패턴을 사용하는지 공부할수 있었다. 



참고-https://injae-kim.github.io/dev/2020/08/06/singleton-pattern-usage.html
