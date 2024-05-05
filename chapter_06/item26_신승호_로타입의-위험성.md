## 로타입의 위험성 

### 정리

**내용 요약**

저자는 로타입의 사용을 하지 말라 하며 제네릭에서 사용되는 용어들과 로타임을 사용하면 안 되는 이유, 그리고 방법과 대체할 방법까지 설명하고 있다.

제네릭 타입과 로타입의 가장 큰 차이점은 제네릭 타입은 타입 매개변수를 지정해 줘서 다른 타입의 변수가 들어왔을 때 컴파일 에러를 낼 수 있다.

로타입의 경우 제네릭 타입이 나오기 전 사용되던 방식이기 때문에 제네릭 타입이 나온 지금 사용을 권장하지 않는다.

### 심화 탐구

**출발점**

제네릭에 대한 용어들이 생소하고 책에 나온 코드들도 잘 사용하지 않는 문법들이 나와 이해하는 데 어려움이 있었다. 따라서 보다 쉬운 코드 예시를 통해 로타임을 왜 사용하면 안 되는지 알아보고 로타입이 정말로 필요 없는지 탐구해 보았다. 

**설명**

1. 제네릭과 로타입 정의 

   - 제네릭

     ~~~java
     ArrayList<E>
     ArrayList<String> StringList= new ArrayList<>();
     //(1) 
     ArrayList<E> arr= new ArrayList<>();
     ~~~

     - 클래스와 인터페이스 선언에 타입 매개변수를 사용한 클래스와 인터페이스
     - 아무것도 지정하지 않는다면 <E> 라고 뜨는데 타입을 지정해달라 는 뜻이다 만약 (1)처럼 사용되면 타입 매개변수를 써주지 않았기 때문에 오류가 발생한다. 

   - 로타입

     ```java
     ArrayList a = new ArrayList();
     ```

     - 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 타입 <>가 없다.
     - 신기하게도 타입을 지정해주지 않았는데 오류가 발생하지 않는다.
     - 대신 이런 경고가 뜬다(ArrayList is a raw type. References to generic type ArrayList<E> should be parameterized) 자바에서도 raw type을 권장하지 않는걸 볼 수 있다.

     

2. 로타입의 문제점

    그렇다면 로타입의 어떤 문제점 때문에 자바조차 사용하지 말라고 권장하는 걸까? 로타입을 사용했을때는 큰문제가 생긴다.

   ```java
   List stringCollection = new ArrayList(); // String을 넣으려고 만든 컬렉션
   		stringCollection.add("일");
   		stringCollection.add("이");
   		// 이런 저런 문자열 값들이 들어다가
   		stringCollection.add(2); // 컴파일 오류 x
   		
   		
   	
   		for(int i=0; i<3; i++) {
   			
   			System.out.println((String)stringCollection.get(i));
   		}
   
   //결과 
   /*
   일
   Exception in thread "main" 이
   java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String (java.lang.Integer and java.lang.String are in module java.base of loader 'bootstrap')
   	at item26.test1.main(test1.java:19)
   	*/
   ```

     주석을 통해 String이 들어갈 거라고 말은 했지만, 만약 바로 타입을 지정해 주지 않았기 때문에 만약 누군가 string Collection을 사용할 때 String 아닌 다른 형을 넣는다면 넣을 당시에는 에러가 발생하지 않지만 출력할 때는  ClassCastException 오류가 발생하는걸 볼 수 있다. 이럴 경우 코드를 실행시켜야 오류가 발생했다는 걸  알 수 있다.

   만약 제네릭을 통해 타입을 명시해 줬다면 

   ```java
   List<String> stringCollection = new ArrayList(); // String을 넣으려고 만든 컬렉션
   		stringCollection.add("일");
   		stringCollection.add("이");
   		// 이런 저런 문자열 값들이 들어다가
   		stringCollection.add(2); // 컴파일 오류
   
   /* The method add(int, String) in the type List<String> is not applicable for the arguments (int)*/
   
   ```

   int형인 2를 넣는 순간 컴파일 에러가 발생하여 보다 빠르게 코드를 수정 할 수 있다.

   

3. 로타입이 남아있는 이유

   사람들이 제네릭을 받아들이는데 오랜 시간이 걸렸고, 이미 로타입으로 작성된 코드들이 너무 많기 때문에, 그 코드들과의 호환성을 위해서 남겨두었다. 제네릭으로 사용하는것이 모든 측면에서 로타입보다 좋기 때문에 제네릭을 사용하는걸 권장하는 것이다.  

   ```java
   List AllCollection = new ArrayList(); //모든 타입을 담을 Collection
   		AllCollection.add("일");
   		AllCollection.add("이");
   		AllCollection.add(2); // 컴파일 오류 x
   		System.out.println(AllCollection);
   /* 결과 :[일, 이, 2]
   ```

   이처럼 AllCollection이라는 리스트 안에 모든 타입을 넣고 싶다면 로타입을 사용하는것이 좋지 않을까? 라는 의문이 생길수 있는데 로타입을 통해 타입을 지정해주지 않는다면 모든 타입을 넣을 수 있는걸 볼 수 있다. 하지만 이마저도 List<Object>을 통해 완벽히 대체 가능하다.

   ```java
   	public static void main(String[] args) {
   		List<Object> AllCollection = new ArrayList(); 
   		AllCollection.add("일");
   		AllCollection.add("이");
   		AllCollection.add(2); 
   		
   		System.out.println(AllCollection);
   /* 결과 :[일, 이, 2]
   ```

   두코드의 결과값은 같지만 그 의미는 다른데 로 타입은 제네릭 타입에서 아예 발을 뺀 것이고
   Object 제네릭 타입은 모든 타입을 허용한다는 것을 컴파일러에서 명시한 것이다

4. 로타입은 정말 필요하지 않을까? 

   > 제네릭 타입에서 아예 발을 뺀 것이고
   > Object 제네릭 타입은 모든 타입을 허용한다는 것을 컴파일러에서 명시한 것이다

   이 두차이가 잘 안느껴질수도 있지만 이와같은 예시에서 큰차이를 불러온다.

   ```java
   public static void main(String[] args) {
   	    List<String> stringCollection = new ArrayList<>();
   	    stringCollection.add("로타입 살아남을수 있을것인가?!");
   	    rawTypeMethod(stringCollection); // 컴파일 에러 발생x
   	    //objectTypeMethod(stringCollection); //컴파일 에러
   	}
   
   	private static void rawTypeMethod(final List stringCollection) {
   		
   		System.out.println(stringCollection.get(0));
   	}
   
   	private static void objectTypeMethod(final List<Object> stringCollection) {}
   
   //결과 로타입 살아남을수 있을것인가?!
   ```

   이처럼 어떤 타입의 배열이든 메서드에서 받을려고 할때 로타입으로 List<String>을 받으면 오류가 생기지 않지만 List<Object>으로 받으면 컴파일 에러가 발생하는걸 볼 수 있는데 이는 List<Object>와  List<String>를 보면 Object가 String의 부모이기 때문에 가능할것 같지만 List<Object>가 List<String>의 부모는 아니기 때문에 오류가 발생하는것이다

   이렇게 로타입을 사용해야하는 이유가 생기나 싶었지만 자바는 로타입의 생존을 허락하지 않았다.

   

5. 비한정 와일드카드 타입

   1. 위의 제약을 없애기 위해 모든 타입을 받을 수 있는 List<?>와일드카드를 사용하면 된다.

      ```JAVA
      public static void main(String[] args) {
      	    List<String> stringCollection = new ArrayList<>();
      	    stringCollection.add("로타입 살아남을수 있을것인가?!");
      	    rawTypeMethod(stringCollection); // 컴파일 에러 발생x
      	    objectTypeMethod(stringCollection); //컴파일 에러 발생x
      	}
      
      	private static void rawTypeMethod(final List stringCollection) {
      		
      		
      		System.out.println(stringCollection);
      		
      	}
      
      	private static void objectTypeMethod(final List<?> stringCollection) {
      		
      		System.out.println(stringCollection);
      	}
      ```

      이러면 문제없이 코드가 잘 실행되는걸 볼수있다. 하지만 비한정 와일드카드 타입에도 약점이 있는데 **`타입 불변식`을 훼손하지 못하게 null외의 아무 값도 넣지 못한다.**

      main에서 stringCollection은 List<String>인데, 메소드의 매개변수는 List<?>이다.
      List<?>에선, 모든 제네릭을 다 받을 수 있지만, 해당 컬렉션 객체가 어떤 타입의 제네릭이었는지 알 수 가 없기 때문에 add를 하지 못하게 한다. **잘못된 타입의 값을 넣었다가 타입 불변식을 훼손할 수 있기 때문**이다. 하지만 로타입은 타입 불변식을 철저히 무시해 버리는 모습을 볼 수있었다. 

      ```JAVA
      	public static void main(String[] args) {
      	    List<String> stringCollection = new ArrayList<>();
      	    stringCollection.add("로타입 살아남을수 있을것인가?!");
      	    rawTypeMethod(stringCollection); // 컴파일 에러 발생x
      	    objectTypeMethod(stringCollection); //컴파일 에러 발생x
      	}
      
      	private static void rawTypeMethod(final List stringCollection) {
      		
      		stringCollection.add(123);// INT형 넣어버리기
      		System.out.println(stringCollection);
      		//결과 [로타입 살아남을수 있을것인가?!, 123] 
      	}
      
      	private static void objectTypeMethod(final List<?> stringCollection) {
      		stringCollection.add(123);// INT형 넣어버리기
      		System.out.println(stringCollection);
      	}
      ```

      

   

6. 로타입을 꼭 써야하는경우

   이렇게 코드의 안정성과 타입불변성까지 해치는 로타입을 써야하는 경우가 2가지가 있다고 이펙티브 자바의 책에서는 소개하고 있다.

   - class 리터럴에서는 class 리터럴에는 배열과 기본타입은 허용하지만 매개변수화 타입을 사용할 수 없다.
   
   - instanceof 연산자 런타임에는 매개변수화 정보가 지워진다. 컴파일 단계에선 잘못된 타입에 대한 체크가 끝나기 때문에 `instanceof List`와 `instanceof List<?>`가 똑같이 작동 한다고 한다. 
   
     
   

**느낌점** 

로타입과 제네릭에 대해 탐구하다보니 평소에 그냥 넘어갔던 <E>표시와 <?>가 어떤 의미인지 제대로 알게 되었고 배열의 타입불변식이 로타입을 통해 깨질수도 있다는게 신기했다. 또한 지금까지 자바에서는 한 리스트안에는 한타입의 타입만 넣을수 있는줄 알았는데 List<object>를 통해 다양한 타입을 한 배열 안에 넣을수 있다는걸 알게되었다.   





참조: https://docs.oracle.com/javase/tutorial/java/generics/rawTypes.html

https://stackoverflow.com/questions/2770321/what-is-a-raw-type-and-why-shouldnt-we-use-it









