## 제목 close()메서드는 꼭 필요할까?

### 정리

**내용 요약**

자바 라이브러리에는 InputStream, OutputStream 등 close 메서드를 호출해서 직접 닫아줘야 하는 자원들이 많다. 만약 자원이 하나라면 try-finally 를 통해 close했겠지만 만약 자원이 2개 이상이라면 try-finally 문이 복잡해져서 디버깅하기에도 어렵고 코드가 지저분해진다. 하지만 이러한 문제는 7세대 이후인 try-with-resources 덕에 해결되었다.


try-with-resources 구조는 AutoCloseable 인터페이스를 구현을 통해 코드에 close 메서드 작성하지 않아도 자동으로 메서드가 작동한다. 이를 통해 finally 구문으로 close 해주지 않아 코드는 더 짧아지고 디버깅이 쉬워졌다.



try-finally 코드

```java
public static void main(String args[]) throws IOException {
    FileInputStream is = null;
    BufferedInputStream bis = null;
    try {
        is = new FileInputStream("file.txt");
        bis = new BufferedInputStream(is);
        int data = -1;
        while((data = bis.read()) != -1){
            System.out.print((char) data);
        }
    } finally {
        // close resources
        if (is != null) is.close();
        if (bis != null) bis.close();
    }
}
출처: https://mangkyu.tistory.com/217 [MangKyu's Diary:티스토리]
```

try-with-resources 코드

```java
public static void main(String args[]) throws IOException {
    try (FileInputStream is = new FileInputStream("file.txt"); BufferedInputStream bis = new BufferedInputStream(is)) {
        int data;
        while ((data = bis.read()) != -1) {
            System.out.print((char) data);
        }
    }
}
출처: https://mangkyu.tistory.com/217 [MangKyu's Diary:티스토리]
```




```java
try (FileInputStream is = new FileInputStream("file.txt"); BufferedInputStream bis = new BufferedInputStream(is))
```
두 코드의 차이점은 resources 코드는 위의 코드처럼 try() 안에서 객체 선언 및 할당은 한다는 점이다. 만약 코드의 실행 위치가 try 문을 벗어나면 resources가 자동으로 try(..) 안에서 선언된 객체의 close() 메서드들을 호출한다. 따라서 close()를 finally에 명시적으로 표현할 필요가 없는 것이다.



### 심화 탐구

**출발점**

자바에서는 많은 라이브러리들이 사용 후 close를 해야 한다. 우리가 알고리즘을 풀 때 자주 쓰는 scanner 또한 close를 해야 한다고 배웠다. 하지만 필자를 포함한 많은 사람들이
scanner을 사용한 후 close 쓰지 않고 코드를 끝낸다.
물론 close를 써주는 게 가장 좋은 방법이겠지만 close를 사용하지 않아도 코드가 정상적으로 작동하는 걸 보면 굳이 사용할 필요가 없는 거 같기도 하다. 그래서 close의 역할과 어떻게 close를 안 해도 그동안 코드가 잘 돌아갔는지 알아보려고 한다.

**설명**

1. close() 메서드는 어디에 사용될까?

    ​통상 자바에서 close() 메서드는 스트림을 닫고 스트림에서 사용 중이던 리소스가 있는 경우 이를 해제하는 데 사용된다. 만약 스트림이 열려 있으면 리소스를 해제하여 스트림을 닫는 역할을 한다.
    스트림이 닫힌 상태라면 close() 를 해도  아무 효과가 없다. 따라서 close의 역할은 자바에게 스트림의 사용이 끝났다는걸  알려주는 역할을 하는 것이다.

    ```java
    public static void main(String args[]) throws IOException {
        FileInputStream is = null;
        BufferedInputStream bis = null;
        try {
            is = new FileInputStream("file.txt");
            bis = new BufferedInputStream(is);
            int data = -1;
            while((data = bis.read()) != -1){
                System.out.print((char) data);
            }
        } finally {
            // close resources
            if (is != null) is.close();
            if (bis != null) bis.close();
        }
    }
    출처: https://mangkyu.tistory.com/217 [MangKyu's Diary:티스토리]
    ```

    finally을 할 때 들었던 예시 코드처럼 FileInputStream이 사용하던 "file.txt" 자원의 사용을 close()로 명시 함으로 서 자원과의 연결을 끊는 것이다.


2. 그렇다면 close()를 안한다면 어떤일이 생길까?

   ```java
   Scanner sc = new Scanner(System.in); 
   ```

   Scanner을 선언후 sc.close()을 하지않는다면 sc에 노란줄이 생기며 "Resource leak: 'sc' is never closed" 라는 경고가 뜬다. 
   이 경고의 의미는 자원 누수(Resource leak) 가 발생했다는 뜻으로 더 이상 사용되지 않는 개체가 힙에 있지만 가비지 수집기가 해당 개체를 메모리에서 제거할 수 없어 불필요하게 메모리가 낭비 되고 있다고 자바가 경고해 주는 것이다. Scanner의 사용이 종료되어 이제 불필요한 자원 이지만  close를 하지 않으면 가비지 컬렉터(gc)가
    해당 개체가 사용 중인지 사용이 끝났는지 판단할 수 없어 힙 영역에 계속해서 존재하여 메모리를 잡아먹고 있는 것이다.
   만약 Scanner(File source)처럼 file 자원을 불러와서 사용하고 있다면 close()를 해주지 않는다면 해당 File source을 다른 프로그램에서 접근하거나 컴퓨터 내부에서 수정을 할 때 오류가 발생할 수 있다.
    그렇기 때문에 사용을 다했다면 close를 통해 종료를 선언하여 GC가 처리할 수 있게 해줘야 한다.


3. try-with-resources 구조를 통한 AutoCloseable가 중요한 이유

    2번에서 close()을 안 했을 때 문제점에 대해서 알아봤다. 하지만 scanner의 경우 크게 해당 불러온 file만 손상시키지 않는다면 큰 문제가 생기지 않을 것이다. 하지만 만약 file이 아닌 DB라고 생각해 보자

    ```JAVA
    	/**
     * 사용한 리소스들을 정리한다.
     * Connection, Statement, ResultSet 모두 AutoCloseable 타입이다.
     * ... 을 이용하므로 필요에 따라서
     * select 계열 호출 후는 ResultSet, Statement, Connection
     * dml 호출 후는 Statement, Connection 등 다양한 조합으로 사용할 수 있다.
     *
     * @param closeables
     */
    public void close(AutoCloseable... closeables) {
        for (AutoCloseable c : closeables) {
            if (c != null) {
                try {
                    c.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    ```
    JDBC에서 서버와 DB를 연결할 때 DB를 불러온 후 자동으로 닫게 해주는 메서드를 우리는 이미 접한 적이 있다. 만약 DB가 닫히지 않는다면 Statement 닫히지 않아, 생성된 Statement 개수가 증가하여 더 이상 Statement를 생성 불가능할 것이다. 또한 DBMS에 연결된 새로운 Connection을 생성 불가능하게 될 것이다. 이처럼 db 사용 후 close는 필수적이기 때문에 AutoCloseable을 통해 자원 사용 후 close를 자동으로 할 수 있게 도와주는 try-with-resources 구조가 중요한 것이다.
    

   

4.  하지만 그동안 close를 하지 않아도 문제가 없던 이유는?

    우리가 알고리즘을 풀 때는 한 가지의 class인 작은 단일 프로그램을 사용하기 때문에 프로그램 종료 후 os가 자원을 회수 해 준다. 또한 알고리즘 풀이시 input으로 입력 값만 받아서 푸는 코드를 짠다. 이러한 이유들 때문에 그동안 sc.close() 안 해도 아무 문제 없이 알고리즘을 풀 수 있었던 것이다. 하지만 여러 개가 연동된 프로그램을 만들었을 때는 memory 초과가 나는 것을 막기 위해 close() 하는 것을 습관화 하는 게 좋다. 

참조:

https://www.geeksforgeeks.org/reader-close-method-in-java-with-examples/

https://dzone.com/articles/memory-leak-andjava-code

AutoCloseable 구현 클래스 https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html



