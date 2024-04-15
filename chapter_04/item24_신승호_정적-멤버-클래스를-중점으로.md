## 정적 멤버 클래스를 중점으로

### 정리

**내용 요약**

중첩클래스란 다른 클래스 안에 정의된 클래스이다. 중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스(item 25)로 만들어야 한다.

- 정적 멤버 클래스 : 클래스 내부에 static으로 선언된 클래스
- 멤버 클래스 :static이 붙지 않은 멤버 클래스이며 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 그래서 클래스명.this 형태로를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.
- 익명 클래스: 익명 클래스는 바깥 클래스의 멤버도 아니다. 쓰이는 시점과 동시에 인스턴스가 만들어진다. 정적 문맥에서라도 상수 변수 이외의 정적 멤버는 가질 수 없다.
- 지역 클래스 : 유효범위가 지역 변수와 같고, 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있고, 정적 멤버는 가질 수없고, 짧게 작성해야 한다.

정적 멤버 클래스를 제외한 나머지는 내부 클래스(inner class)다. 

멤버 클래스에서 바깥 인스턴스에 접근할 일이 없으면 무조건 static을 붙여서 정적 멤버 클래스로 만드는게 좋다. static을 생략하면 바깥 인스턴스로의 숨은 참조가 생기고, 참조를 저장하기 위해 시간과 공간이 소비되고 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하기 때문이다.

### 심화 탐구

**출발점**

정적 멤버 클래스의 사용법에 대해 책에서는 "정적 멤버 클래스는 흔히 바깥 클래스와 함께 쓰일 때만 유용한 private 도우미 클래스로 쓰인다." 라고 나와있다. 여기서 나오는 도우미 클래스라는 단어의 의미가 이해가 안되었기 때문에 정적 멤버 클래스에 대해 구체적으로 알아보고 어디에 사용되는지 찾아겠다. 

**설명** 

1. **정적 멤버 클래스란?**

   - 클래스 내부에 static으로 선언된 클래스다.
   - 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근 가능하다. 그 외는 일반 클래스와 똑같다.
   - private으로 선언시 바깥 클래스에서만 접근 가능하다.

   ```java
   //다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근 가능하다. 그 외는 일반 클래스와 똑같다.
   public class OuterClass {
   	 private int x = 10;
   	public static void main(String[] args) {
   		InnerClass_Private a= new InnerClass_Private();
   		a.test();
   		
   		InnerClass_public b= new InnerClass_public();
   		b.test();
   		
   	}
   // 출력 100 1000
   	  private static class InnerClass_Private {
   	    void test() {
   	      OuterClass outerClass = new OuterClass();
   	      //바깥 클래스에 private 멤버에 접근하는 중
   	      outerClass.x = 100;
   	      System.out.println(outerClass.x);
   	    }
   	  }
   	  
   	  public static class InnerClass_public {
   		    void test() {
   		      OuterClass outerClass = new OuterClass();
   		      //바깥 클래스에 public 멤버에 접근하는 중
   		      outerClass.x = 1000;
   		      System.out.println(outerClass.x);
   		    }
   		  }
   	}
   
   //private으로 선언시 바깥 클래스에서만 접근 가능하다
   public class test {
   	
   	public static void main(String[] args) {
   		InnerClass a= new InnerClass();
           //InnerClass cannot be resolved to a type 오류뜸 
   	}
   
   }
   
   // public 으로 선언시 다른 클래스에서 접근 가능 
   import item24.OuterClass.InnerClass_public;
   
   public class test {
   	
   	public static void main(String[] args) {	
   		InnerClass_public b= new InnerClass_public();
   		b.test();
   	}
   
   }
   
   
   ```

   - 정적 멤버 클래스는 흔히 바깥 클래스와 함께 쓰일 때만 유용한 private 도우미 클래스로 쓰인다.

2. **언제 사용할까?**

   - 책에서는 "private 도우미 클래스" 로 사용된다고 했다 이 뜻을 예시 코드를 통해 알아보자

     ```java
     // gpt 코드
     
     // "private 도우미 클래스"라는 뜻은 주로 정적 멤버 클래스가 외부 클래스의 기능을 보조하거나 구현 세부 사항을 감추는 데 사용된다는 뜻이다.
     
     
     public class OuterClass {
         
         private static int count = 0;
         
         public static void main(String[] args) {
             helperMethod();
         }
         
         private static void helperMethod() {
             HelperClass helper = new HelperClass();
             helper.printCount();
         }
         // 내부에 만들면 private class를 만들수 있으므로 외부로 부터 완전히 보호된다. 
         private static class HelperClass {
             
             public void printCount() {
                 System.out.println("Count: " + count);
             }
             
         }
     }
     // 정적 멤버 클래스는 외부 클래스에서만 사용되는 도우미 기능을 제공하여 코드의 모듈성을 높이고, 클래스 간의 결합도를 낮추는 데 도움이 된다.
     ```

   - 어디에 사용되고 있을까? 

     - 캡슐화: item 16의 설명처럼 만약 public 클래스가 필드를 public으로 공개하고 싶다면 중첩 클래스를 활용하여 중첩클래스를 private로 만들고 그안에 public 필드를 만들수 있다.  
     - 컬렉션 프레임워크: 자바의 많은 컬렉션 클래스들은 내부적으로 도우미 클래스를 사용하여  구현된다. 

     

**느낌점** 

4페이지의 분량이었지만 4페이지 자체가 요약된 느낌이라서 내용을 이해하는 데 많은 시간이 필요했다. 신기하게도 저번 심화 주제도 중첩 클래스에 관해 썼었는데 이번에도 중첩클래스에 대해 글을 쓰게 되었다. 중첩클래스에 대한 이해도가 상승하였다. 





참조: https://stackoverflow.com/questions/1953530/why-does-java-prohibit-static-fields-in-inner-classes/1954119#1954119%EB%A5%BC

https://www.baeldung.com/java-nested-classes