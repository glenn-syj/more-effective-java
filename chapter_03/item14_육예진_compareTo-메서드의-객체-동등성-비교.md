## compareTo 메서드의 객체 동등성 비교

### 정리

**내용 요약**

compareTo는 Comparable 인터페이스의 하나뿐인 메서드이다. (Object 소속 메서드 아님)  
Comparable을 구현했다는 것은 해당 클래스의 인스턴스들에 순서가 있음을 의미하며, 순서 뿐만 아니라 동치성 비교 기능도 가지고 있다.  
순서를 고려해야 하는 클래스를 작성한다면 Comparable 인터페이스를 구현하여, 정렬/검색/비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다.


- compareTo 메서드 구현시 지켜야 할 규약
  1. x가 y보다 크면 y는 x보다 작아야 한다.
  2. x > y 이고, y > z 이면, x > z 여야 한다. 
  3. x == y 이고, x > z 이면 y > z 여야 한다. (크기가 같다면 어떤 객체와 비교해도 두 결과가 같아야 한다)
  4. compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.

- compareTo 사용시 유의점
  - 인스턴스들의 필드값을 비교할 때, <와 > 연산자보다는 박싱된 기본 타입 클래스의 정적 compare 메서드나, Comparator 인터페이스의 비교자 생성 메서드를 사용하자.



### 심화 탐구

**출발점**

> 4. compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.  
> 이번 권고가 필수는 아니지만 꼭 지키는 게 좋다. Comparable을 구현하고 이 권고를 지키지 않은 모든 클래스는 그 사실을 명시해야 한다.  
> "주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다."

저자는 이 장의 처음부터 compareTo 메서드를 equals 메서드와 나란히 이야기하며, 규약 설명시 두 메서드의 일관성에 대해서도 권고한다.  

정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용한다고 했다.  
그렇다면 두 메서드의 일관성을 지키지 않은 클래스는 컬렉션 사용시 어떤 모습을 하게 될까?  

직접 클래스를 생성하여 테스트를 통해 해당 규약의 필요성에 대해 확인해보기로 했다.


**설명**

<hr>

1. Student 클래스 정의
   
    각각의 인스턴스가 고유한 id값을 갖고, score를 통해 인스턴스들을 정렬할 수 있는 Student 클래스를 정의했다.
    - id 값으로 동치성을 판단하도록 equals 메서드를 재정의
    - score 값 비교를 통해 인스턴스간 순서를 매길 수 있도록 Comparable 인터페이스를 구현하여 compareTo 메서드를 정의
    ```java
    class Student implements Comparable<Student>{
        int id;
        String name;
        int score;
        
        public Student(int id, String name, int score) {
            this.id = id;
            this.name = name;
            this.score = score;
        }

        @Override
        public boolean equals(Object obj) {
            if (this == obj)
                return true;
            if (obj == null)
                return false;
            if (getClass() != obj.getClass())
                return false;
            Student other = (Student) obj;
            return Objects.equals(this.id, other.id);
        }

        @Override
        public int compareTo(Student o) {
            return Integer.compare(this.score, o.score);
        }

        @Override
        public String toString() {
            return "Student [id=" + id + ", name=" + name + ", score=" + score + "]";
        }
    }
    ```


2. HashSet과 TreeSet 비교
   
   HashSet은 원소들의 동치성을 비교(equals)하여 중복을 허용하지 않는 자료구조이며, 원소의 순서에는 관여하지 않는다. (정렬 불가)
   TreeSet은 HashSet과 마찬가지로 원소의 중복을 허용하지 않지만, 정렬 기능을 제공하는 자료구조다. 

   위에서 정의한 Student 클래스의 인스턴스를 생성하고 자료구조에 원소로 추가하여, 결과를 비교해보았다.

    ```java
    public class Test {
        public static void main(String[] args) {
            
            Student s1 = new Student(1, "육예진", 50);
            Student s2 = new Student(2, "육민우", 50);
            
            Set<Student> hash = new HashSet<>();
            
            hash.add(s1);
            hash.add(s2);
            System.out.println(hash.toString());
            // 결과: [Student [id=2, name=육민우, score=50], Student [id=1, name=육예진, score=50]]
            
            Set<Student> tree = new TreeSet<>();
            
            tree.add(s1);
            tree.add(s2);
            System.out.println(tree.toString());
            // 결과: [Student [id=1, name=육예진, score=50]]
            
        }
    }
    ```

    두 원소의 id와 name은 값이 달라 서로 다른 객체라고 판단해야 하지만 TreeSet에서는 원소가 한 개 뿐이다.

    원소간 정렬에 관여하지 않고 equals 메서드만을 사용해 동치성을 판단하는 HashSet과 달리, TreeSet에서는 equals메서드가 아닌 compareTo 메서드로 원소간 동치성을 판단하여 중복을 제거하는 것을 알 수 있다.

    두 인스턴스의 score가 같은 경우, id값 추가 비교를 통해 정렬하는 규칙이라면 어떨까.

    ```java
    @Override
	public int compareTo(Student o) {
		int result = Integer.compare(this.score, o.score);

        // 점수가 같다면 id값으로 정렬해라
		if (result == 0) {
			result = Integer.compare(this.id, o.id);
		}
		return result;
	}
    ```
    compareTo 메서드에 비교 대상 필드값을 추가해봤다.
    다시 실행해본 결과는 다음과 같았다.

    ```java
    public class Test {
        public static void main(String[] args) {
            
            Student s1 = new Student(1, "육예진", 50);
            Student s2 = new Student(2, "육민우", 50);
            
            Set<Student> hash = new HashSet<>();
            
            hash.add(s1);
            hash.add(s2);
            System.out.println(hash.toString());
            // 결과: [Student [id=2, name=육민우, score=50], Student [id=1, name=육예진, score=50]]
            
            Set<Student> tree = new TreeSet<>();
            
            tree.add(s1);
            tree.add(s2);
            System.out.println(tree.toString());
            // 결과: [Student [id=1, name=육예진, score=50], Student [id=2, name=육민우, score=50]]
            
        }
    }
    ```
    compareTo에서 비교하는 필드중 일치하지 않는 값이 있기 때문에 이번에는 중복으로 처리되지 않았다.   
    다만, HashSet의 결과는 id값 크기에 상관없이 출력된 반면 TreeSet은 score 값이 동일할 때 id값을 통해 오름차순으로 정렬되도록 정의되었기 때문에 순서대로 출력되었다.

3. 추가 테스트

    그렇다면, id와 score가 모두 일치하는 경우에는 어떤 결과일지 추가로 테스트해봤다.

    ```java
    public class Test {
        public static void main(String[] args) {
            
            Student s1 = new Student(1, "육예진", 50);
            Student s2 = new Student(2, "육민우", 50);
            Student s3 = new Student(2, "송창용", 50);
            
            Set<Student> hash = new HashSet<>();
            
            hash.add(s1);
            hash.add(s2);
            hash.add(s3);
            System.out.println(hash.toString());
            // 결과: [Student [id=2, name=육민우, score=50], Student [id=1, name=육예진, score=50], Student [id=2, name=송창용, score=50]]
            
            Set<Student> tree = new TreeSet<>();
            
            tree.add(s1);
            tree.add(s2);
            tree.add(s3);
            System.out.println(tree.toString());
            // 결과: [Student [id=1, name=육예진, score=50], Student [id=2, name=육민우, score=50]]
            
        }
    }
    ```
    TreeSet은 예상대로 compareTo에서 비교자로 다루는 필드의 값이 동일할 경우 중복으로 간주되어 나중에 등록한 원소가 없는 것을 볼 수 있다.

    그런데, HashSet의 결과가 예상과 달랐다. 분명 equals 메서드에서는 id값으로만 동치성을 판단하도록 재정의 했는데, id가 같은 s2와 s3 모두 담겨있다.
    
    왜 그럴까?

    Student 클래스에 hashCode 메서드를 재정의하지 않아 기본구현을 사용하기 때문이다. 
    
    HashSet은 동일성 검사를 위해 equals뿐만 아니라 hashCode 메서드도 함께 사용한다.  
    item11에서 equals 메서드와 함께 hashCode 메서드 재정의를 강조한 이유다.  
    hashCode를 equals와 일관되게 재정의 하지 않으면 hashCode() 결과를 기반으로 중복을 제거하지 못하고 중복된 요소를 포함하게 된다.

    이는 인스턴스 비교에 있어 '동등성(equality)'과 '동일성(identity)'라는 개념으로 이해해볼 수 있다. 글 하단에 참고할만한 링크를 첨부한다.
    

**결론**

정렬된 자료구조의 compareTo 메서드를 정의할 때에는 equals 메서드를 대신하여 객체의 동치성을 판단하여 예상과 다른 결과가 나올 수 있기 때문에 유의해야한다.  

각 자료구조의 동치성 판단 원리를 제대로 구분하고, 클래스의 목적에 맞게 각 메서드들을 구현할 필요가 있겠다.

<hr>


[참고 자료]
- [동일성과 동등성 관련 참조 글](https://creampuffy.tistory.com/m/140)
