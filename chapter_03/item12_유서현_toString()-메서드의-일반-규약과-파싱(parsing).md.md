## toString() 메서드의 일반 규약과 파싱(parsing)

`Effective Java` 3장 모든 객체의 공통 메서드 - '아이템 12. toString을 항상 재정의하라' 를 읽고

### 정리

**내용 요약**

Object의 기본 `toString()` 은 단순히 `클래스명@해시코드` 를 반환하므로, 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.

`toString()`의 **일반 규약**(심화 탐구)에 따르면, ‘간결하면서 사람이 읽기 쉬운 형태의 유익한 정보’를 반환해야 하며, ‘모든 하위 클래스에서 이 메서드를 재정의하라’고 하고 있다.

재정의 시 다양한 이점을 가지는데 대표적으로는 디버깅이 쉬워진다. 

- `toString()`은 직접 `.toString()` 으로 호출하지 않더라도, 객체를 `println`하거나 `assert` 구문에 넘길 때, 디버거가 객체를 출력할 때 자동으로 쓰인다.
  
- 그러므로 제대로 재정의하지 않는다면 알아보기 힘든 참조값만 오류 메세지에 남을 것이다.

`toString()` 의 오버라이드 시 반환값의 포맷을 문서화할지도 정해야 한다. 전화번호 등 값 클래스라면 문서화하기를 권한다. 

*포맷을 명시하게 된다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환 가능한 정적 팩터리 · 생성자를 함께 제공해주는 것이 좋다. (이해 부족)*

다만 이 때 주의해야 할 점은, 이렇게 반환값의 포맷을 문서화하는 것은 무조건 추천되는 사항이 아니라는 점이다. 포맷을 한 번 명시한 이후에는 평생 그 포맷에 얽매일 수 밖에 없다.

또한 포맷 명시와 관련 없이, `toString()`이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공해야 한다. (getter를 모든 멤버 변수에 선언해 값에 접근 가능하도록 한다면 문제 없을 듯 하다.) 그런 접근자를 제공하지 않을 시 프로그래머는 toString의 반환값을 **파싱**(심화 탐구)할 수밖에 없다.

### 심화 탐구

**설명**

1. `toString()` 메서드의 일반 규약

    본문에서 `toString()` 메서드의 일반 규약이 언급되는데, 자바 공식 문서 중 어느 부분에 해당 서술이 있을지 찾아보았다.

    ```java
        /**
        * Returns a string representation of the object.
        * @apiNote
        * In general, the
        * {@code toString} method returns a string that
        * "textually represents" this object. The result should
        * be a concise but informative representation that is easy for a
        * person to read.
        * It is recommended that all subclasses override this method.
        * The string output is not necessarily stable over time or across
        * JVM invocations.
        * @implSpec
        * The {@code toString} method for class {@code Object}
        * returns a string consisting of the name of the class of which the
        * object is an instance, the at-sign character `{@code @}', and
        * the unsigned hexadecimal representation of the hash code of the
        * object. In other words, this method returns a string equal to the
        * value of:
        * <blockquote>
        * <pre>
        * getClass().getName() + '@' + Integer.toHexString(hashCode())
        * </pre></blockquote>
        *
        * @return  a string representation of the object.
        */
        public String toString() {
            return getClass().getName() + "@" + Integer.toHexString(hashCode());
        }
    ```

    `* It is recommended that all subclasses override this method.`에서 해당 규약을 확인 가능하다.

    또한 결과는 사람이 읽기 쉬운 간결하지만 유익한 표현이어야 하며, Override 하지 않을 시 객체 클래스의 이름, `@`, 해시 코드(주소값, 16진수로 구성된 문자열)을 반환함을 알 수 있다.

    한 편 이클립스에서는 우클릭 후 Source → Generate toString() 을 통해 간편하게 toString()을 오버라이드 할 수 있다. 지금까지 객체를 만들고 테스트 시 애용해온 기능이기도 하다. 이 자동 생성 기능은 클래스에 의미에 맞추어 생성해주는 기능이 아니므로, 어떤 클래스에서는 의미를 나타내는 데에 부적합할 수도 있지만, 단순 참조값만을 반환해주는 클래스보다는 훨씬 유용하다.

    예를 들어 `Person` 이라는 클래스를 아래와 같이 싱글턴으로 작성하고 toString에 대한 테스트를 진행해 보겠다.

    ```java
    public class Person {
        String name;
        int age;

        private Person() {

        }

        private static Person person = new Person();

        public static Person getInstance() {
            return person;
        } // 싱글턴 패턴

        @Override
        public String toString() {
            return "Person [name=" + name + ", age=" + age + "]";
        }

        // getter, setter
        public String getName() {
            return name;
        }

        ...
    ```

    ```java
    public class test {
        public static void main(String[] args) {

            Person p = Person.getInstance();

            p.setName("유서현");
            p.setAge(27);

            System.out.println(p.toString());
            // 오버라이드 전: Person@5ca881b5
            // 오버라이드 후: Person [name=유서현, age=27]
        }
    }
    ```

    오버라이드 전에는 객체의 클래스명과 참조값 주소만을 알 수 있지만, 이클립스 기본 기능으로 오버라이드 이후에는 간단하게 객체의 멤버 변수들의 값을 조회 가능하다.

    다만 기본 오버라이딩 틀을 사용하기보다는, 개발자의 의도대로 주요 정보들을 반환하며 디버깅도 용이하도록 오버라이드를 틈틈히 연습해 봄이 좋을 듯 하다.

    ```java
    @Override
        public String toString() {
            return "클래스명: Person, 이름(name): " + name + ", 나이(age): " + age;
        }
    ```

    실행결과: `클래스명: Person, 이름(name): 유서현, 나이(age): 27`

    반환값의 포맷을 문서화할지도 결정해야 하는 사항이다. 

    예시로 휴대폰 번호의 정보를 저장하는 객체를 멤버 변수 3개로 나누어 `toString()`을 오버라이드 한다면, `PhoneNumber(areaCode=010, prefix=1234, lineNum=5678)` 와 같이 출력하기보다는 문서화하여 `010-1234-5678` 과 같이 출력한다면 이 리턴값 그대로 입출력에 사용도 가능하며, txt 등 데이터 객체로 저장도 가능하다. 추가적으로 `*/ 이 메서드의 반환값 포맷은 "국가코드-지역번호-번호" /*` 와 같은 주석을 달아 의도를 정확히 밝힘이 좋다고 한다. 


<br>


2. 파싱 (parsing)

    파싱이라는 단어를 보고, `parseInt()` 등 파싱이 가능한 메서드들이 생각나서 추가적인 조사를 시작했다. 

    파싱은 기본적으로 ‘특정한 내용을 추출한다’는 의미를 내포하는 단어다. 자바에서 파싱(parsing)이란 문자열인 String 값을 기본 자료형 값으로 변경하는 것이다. `parse()` 메서드가 존재하는 많은 자바 클래스들이 있다. 

    예를 들어 `parseInt()` 메서드는 특정 문자열에서 기본 데이터 유형을 가져오는, 문자열을 숫자로 변환하는 데 사용된다.

    `Period` 는 년, 월, 일로 시간을 표시하는 Java 클래스이며, 역시 `parse()`가 존재한다. `String age = "P17Y9M5D";` 와 같이 파싱을 위한 날짜를 형식에 맞게 입력하고 `Period p = Period.parse(age);` 이렇게 p에 저장한 멤버 변수 값에 접근 가능하다. 아래는 [codegym](https://codegym.cc/ko/groups/posts/ko.451.yejega-pohamdoen-javaui-11gaji-parse-meseodeu)의 예시 코드이다.

    ```java
    import java.time.Period;

    public class test {

        public static void main(String[] args) {
            // Here is the age String in format to parse
            String age = "P17Y9M5D";

            // Converting strings into period value
            // using parse() method
            Period p = Period.parse(age);
            System.out.println("the age is: ");
            System.out.println(p.getYears() + " Years\n" + p.getMonths() + " Months\n" + p.getDays() + " Days\n");
        }
    }
    ```

    출력 결과: `the age is: 17 Years, 9 Months, 5 Days`

    `StringTokenizer` 클래스에서도 객체를 생성 뒤 토큰 분리자를 지정하여 문자열 파싱이 가능하며, `split()`의 경우 따로 객체를 생성할 필요 없이 문자열 뒤에 메서드를 붙여 파싱이 가능하다.

    본문에서 나온 내용으로, 클래스에서 getter를 제대로 지정하지 않을 시 문자열 파싱을 통해 값을 얻어오는 과정을 아래와 같은 예시 코드로 작성해 보았다.

    ```java
    public class PhoneNumberParser {
        public static void main(String[] args) {
            // toString()으로부터 반환된 문자열
            String phoneNumber = p.toString();
            
            // '-'를 기준으로 문자열을 분리하여 추출
            String[] parts = phoneNumber.split("-");
            
            // 각각 변수에 저장
            int firstNum = Integer.parseInt(parts[0]);
            int secondNum = Integer.parseInt(parts[1]);
            int thirdNum = Integer.parseInt(parts[2]);
            
            // 추출된 정보 출력
            System.out.println("지역 코드: " + firstNum);
            System.out.println("프리픽스 : " + secondNum);
            System.out.println("가입자 번호: " + thirdNum);
        }
    }
    ```

    이런 추출과 변환, 출력 과정들이 불필요하게 자원을 사용하기 때문에 꼭 적절하게 접근 가능한 접근자를 생성해두도록 하자.


<br>

**참고 자료**

[https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html#toString()](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html#toString())

https://codegym.cc/ko/groups/posts/ko.451.yejega-pohamdoen-javaui-11gaji-parse-meseodeu

[https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Integer.html#parseInt(java.lang.String)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Integer.html#parseInt(java.lang.String))

[https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#split(java.lang.String)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#split(java.lang.String))