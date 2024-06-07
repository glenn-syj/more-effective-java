## 타입 토큰에 대해

### 정리

**내용 요약**

하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수는 제한된다.

그렇기에 데이터베이스에서 하나의 행에 따른 여러개수의 열을 타입 안전하게 이용하는 것과 같은 문제들을 해결하기 위해서는, 다른 유연한 수단이 필요하다.

이 문제를 해결하는 방법이 바로 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 해당 키를 함께 재공하는 것이다.

이 방법을 사용하면 제네릭 타입 시스템은 값의 타입이 키와 같음을 보장해준다.

이 방법을 바로 `타입 안전 이종 컨테이너 패턴`이라 한다.

`타입 안전 이종 컨테이너 패턴`의 예시는 다음과 같다.

```java
// Favorites 클래스 

public class Favorites{

private Map<Class<?>, Object> favorites

public <T> void putFavorite(Class<T> type, T instance){
    favoites.put(Objects.requireNonNull(type), instance);
}

public <T> T getFavorite(Clas<T> type) {
    return type.cast(favorites.get(type));
}

}

```

```java
// Favorites 클래스를 이용해 즐겨찾기를 등록

public static void main(String[] args){
    Favorites f = new Favorites();
    
    f.putFavorites(String.class, "Java");
    f.putFavorites(Integer.class, 0xcafebabe);
    f.putFavorites(Class.class, Favorites.class);

    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);

    System.out.printf("%s %x%n", favoriteString, favoriteInteger, favoriteClass.getName());
}

```

Map 변수의 타입이 <Class<?>, Object> 이기 때문에, 와일드카드 타입이 중첩되고, 이를 통해 모든 키가 서로 다른 매개변수화 타입일 수 있게 되는 것이다.

과정을 살펴보자면, 주어진 class 객체에 해당하는 값을 favorite 맵에서 꺼내면, 해당 객체는 잘못된 컴파일 타임 타입을 가지고 있는 상태이다.

따라서, 이 객체의 타입은 Object에서 T로 바꾸어 반환해줘야 하는데,
이를 위해, cast 메서드를 이용하여 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환 해주고 있는 것이다.

Favorites에서 사용하는 타입 토큰은 어떤 class 객체든 받아들이고 있는데, 특정 타입을 제한하고 싶다면 한정적 타입토큰을 활용하면 가능하다.

item39에서 다룰 애너테이션 API가 한정적 타입 토큰을 적극적으로 사용하는 것으로, 애너테이션된 요소는 가지고 있는 키가 애너테이션 타입인 타입 안전 이종 컨테이너이다.



### 심화 탐구


**출발점**

본문에서 이야기하는 타입 안전 이종 컨테이너에 대한 설명을 이해하는 것이 쉽지 않고, 설명이 와닿지 않으므로, 핵심 키워드가 되는 타입 토큰에 대해 정리하여 타입 안전 이종 컨테이너에 대한 이해를 뒷받침해보겠다.


---


**설명**

#### 타입 토큰이란

`타입 토큰`이란 `클래스 리터럴`을 이용하여 타입을 나타내는 형태를 말한다.

`클래스 리터럴`이란 String.class, Integer.class를 의미하는 것으로, String.class는 Class<String>을 의미하고 Integer.class는 Class<Integer>를 의미한다.

method(Class<?> class)와 같은 메서드가 있다고 가정한다면, 해당 메서드는 method(String.class)처럼 클래스 리터럴을 사용하는 타입 토큰을 인자로 받아서 호출된다.

이러한 타입 토큰은 `타입 안정성`이 필요한 곳에 사용된다.

#### 수퍼 타입 토큰

그러나 타입 토큰은 한계가 존재하는데, List형태의 클래스 리터럴이 존재하지 않기 때문에 List<String> 형태의 타입 토큰을 사용할 수 없다.

이러한 타입 토큰의 한계를 해결하기 위해 Neal Gafter가 슈퍼 타입 토큰을 고안하게 된다.

수퍼 타입 토큰은 상속과 Reflection을 조합하여 List<String>.class를 사용하는 것과 같은 효과를 발생시킨다.

이 때, `getGenericSuperclass()`를 활용하게 된다.

`getGenericSuperclass`는 상위 클래스가 ParameterizedType일 경우, 실제 타입 파라미터드을 반영한 타입을 반환한다.

따라서 해당 메서드를 이용하여 Class<List<String>>를 반환하게 만들어 List<String> 형태의 타입 토큰을 사용할 수 있게 되는 것이다.

수퍼 타입 토큰을 코드로 표현하면 다음과 같다.

```java
class Super<T> {}

class Sub extends Super<List<String>> {}
Sub sub = new Sub();
Type typeOfGenericSuperclass = sub.getClass().getGenericSuperclass();

Type typeOfGenericSuperclass = new Super<List<String>>(){}.getClass().getGenericSuperclass();

Type typeOfGenericSuperclass = new TypeReference<List<String>>(){}.getClass().getGenericSuperclass();
```

### 어려운 점

타입 안전 이종 컨테이너 패턴에 대해 이해하기 위해 핵심 키워드인 타입 토큰에 대해 추가적으로 조사하였으나, 그래도 타입 안전 이종 컨테이너 패턴에 대해 와닿지 않았다.

타입 토큰 또한 어려운 개념이기에 기본적인 원리를 보아도 완전한 이해는 하지 못하였다.

이후 추가로 제네릭 타입에서 시작하여 차근차근 개념을 정리해 타입 안전 이종 컨테이너 패턴과 타입 토큰에 대해 완전한 이해를 해보도록 노력하겠다.

---


Reference:

https://homoefficio.github.io/2016/11/30/%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A6%AC%ED%84%B0%EB%9F%B4-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0-%EC%88%98%ED%8D%BC-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0/


