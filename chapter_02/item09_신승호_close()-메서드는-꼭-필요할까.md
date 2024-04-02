## 제목 close()메서드는 꼭 필요할까?

### 정리

**내용 요약**

자바 라이브러리에는 InputStream, OutputStream등 close 메서드를 호출해서 직접 닫아줘야 하는 자원들이 많다. 만약 자원이 하나라면 try-finally 를통해 close했겠지만 만약 자원이 2개 이상이라면 try-finally문이 복잡해져서 디버깅 하기에도 어렵고 코드가 지져분해진다. 하지만 이러한 문제는 7세대 이후인 try-with-resources덕에 해결되었다.



이구조는 AutoCloseable 인터페이스를 구현을 통해 코드에 close 메서드작성하지 않아도 자동으로 메소드가 작동한다. 이를통해 finally구문으로 close 해주지 않아 코드는 더 짧아지고 디버깅이 쉬워졌다.



try-finally 코드

```
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

```
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



두코드의 다른점은 resources 코드는 try (FileInputStream is = new FileInputStream("file.txt"); BufferedInputStream bis = new BufferedInputStream(is))

식으로 try() 안에서 객체 선언 및 할당은 한다는 것 입니다. 만약 코드의 실행 위치가 try문을 벗어나면 resources가 자동으로 try(..) 안에서 선언된 객체의 close() 메서드들을 호출합니다. 따라서 close()를 finally에 명시적으로 표현할 필요가 없는것 입니다.



### 심화 탐구

**출발점**

자바에서는 많은 라이브러리들이 사용후 close를 해야한다. 우리가 알고리즘을 풀때 자주 쓰는 scanner 또한 close를 해야한다고 배웠다. 하지만 필자를 포함한 많은 사람들이

scanner을 사용한후 close쓰지않고 코드를 끝낸다. 물론 close를 써주는게 가장 좋은 방법이겠지만 close를 사용하지 않아도 코드가 정상적으로 작동하는걸 보면 굳이 사용할 필요가 없는거 같기도 하다. 그래서 close의 역할과 어떻게 close를 안해도 그동안 코드가 잘 돌아갔는지 알아볼려고 한다. 

**설명**

1. close() 메서드는 어디에 사용될까?

​	통상 자바에서 close() 메서드는 스트림 **을** 닫고 스트림에서 사용 중이던 리소스가 있	는 경우 이를 해제하는 데 사용됩니다.

- 스트림이 열려 있으면 리소스를 해제하는 스트림을 닫습니다.

- 스트림이 이미 닫혀 있으면 아무 효과가 없습니다.

  close를 통해 해당 리소스의 사용이 끝났다라는걸 자바에게 알려주는 역할을 하는것이다. 

2. 그렇다면 close()를 안한다면 어떤일이 생길까?

   ```
   Scanner sc = new Scanner(System.in); 
   ```

   선언후 sc.close()을 하지않는다면 sc노란줄이 생기며 "Resource leak: 'sc' is never closed" 라는 경고가 뜬다. 

   이는 메모리 누수는 **더 이상 사용되지 않는 개체가 힙에 있지만 가비지 수집기가 해당 개체를 메모리에서 제거할 수 없어** 불필요하게 유지 관리되는 상황입니다.

   Scanner 의 사용이 종료되어 불필요 하지만 close를 하지 않으면 가비지컬렉터(gc)가

   해당 개체가 사용중인지 사용이 끝났는지 판단 할 수 없어 close를 하지 않는다면 힙 영역에 계속해서 존재 하여 메모리를 잡아먹고 있는것이다.  그렇기 때문에 사용을 다했다면 close 를 통해 종료를 선언하여 GC가 처리할수 있게 선언 해줘야 한다. 

   

3.  하지만 그동안 close를 하지 않아도 문제가 없던 이유

   우리가 알고리즘을 풀때는 한가지의 class인 작은 단일 프로그램을 사용하기 때문에

   프로그램 종료후 os가 자원을 회수 하기 때문에 그동안 sc.close()안해도

   아무 문제 없이 알고리즘을 풀수 있었던 것이었습니다. 하지만 여러개가 연동된 프로그램을 만들었을때는 memory초과가 나는 것을 막기 위해 close() 하는것을 습관하 하는게 좋습니다. 

참조:

https://www.geeksforgeeks.org/reader-close-method-in-java-with-examples/

https://dzone.com/articles/memory-leak-andjava-code

AutoCloseable 구현 클래스 https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html



