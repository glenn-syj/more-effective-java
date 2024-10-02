## java에서의 간단한 성능 및 메모리 테스트 방법

### 정리

**내용 요약**

기본 타입(int, double, boolean 등)과 박싱된 기본 타입(Integer, Double, Boolean 등)의 주된 차이점
1. 박싱된 기본 타입은 식별성(identity)을 갖는다. 
2. 박싱된 기본 타입은 null을 가질 수 있다.
3. 기본 타입이 시간과 메모리 사용면에서 더 효율적이다.

편리함에 가려진 박싱된 기본 타입의 위험성
1. == 연산자를 직접 사용하면 오류 발생 가능성이 있다.
2. 기본타입과의 연산에서 오토 언박싱이 일어나는데, 값이 null이면 NullPointerException이 발생한다. 

박싱된 기본 타입을 써야만 하는 경우
1. 컬렉션의 매개변수 타입
2. 매개변수화 메서드의 타입 매개변수


### 심화 탐구

**출발점**

기본 타입이 박싱된 기본 타입보다 성능이나 메모리 사용면에서 유리하다는 것은 흔히 알고 있는 이야기다.  
일정 수준 이상의 알고리즘 문제만 풀어봐도 그 속도차이를 확연히 느낄 수 있다.  
하지만 체감하는 것 외에 연산속도나 메모리 사용량을 수치화해서 확인해보고 싶었다.  


**설명**

<hr>

프로세스의 속도나 메모리 사용량을 측정하는 데에는 다양한 방법이 있겠지만,  
java에서 제공하는 메서드를 통해 간단하게 임시로 확인해볼 수 있는 코드를 작성해보았다.

1. 성능 비교 - System.currentTimeMillis()

    기본 타입 long과 박싱된 기본 타입 Long의 연산 속도를 비교해보았다.
    ```java
    public class Test {
        public static void main(String[] args) {
            
            // long 타입으로 시간 측정
            long startTime = System.currentTimeMillis();  // 시작 시간
            long sumLong = 0L;
            
            for (long i = 0; i <= Integer.MAX_VALUE; i++) {
                sumLong += i;
            }
            
            long endTime = System.currentTimeMillis();    // 끝난 시간
            System.out.println("long 타입 소요 시간: " + (endTime - startTime) + "ms");		// long 타입 소요 시간: 1194ms

            // Long 타입으로 시간 측정
            startTime = System.currentTimeMillis();       // 시작 시간
            Long sumLongObject = 0L;
            
            for (long i = 0; i <= Integer.MAX_VALUE; i++) {
                sumLongObject += i;
            }
            
            endTime = System.currentTimeMillis();         // 끝난 시간
            System.out.println("Long 타입 소요 시간: " + (endTime - startTime) + "ms");		// Long 타입 소요 시간: 3386ms
        }
    }
    ```
    0부터 Integer.MAX_VALUE까지의 연속된 수들을 더하는 연산이다. 아이템 설명에도 나와있는 간단 예제를 활용했다.
    반복문이 시작되기 전 시작시각을 저장하고, 반복문이 끝난 후의 종료 시각을 저장해서 그 차이를 출력했다.

    같은 연산을 수행하는 데에 long 타입은 1194ms, Long 타입은 3386ms로 약 3배 정도 차이가 났다.
    간단하게 성능 테스트를 해볼 때 유용할 것 같다.

2. 메모리 사용량 테스트 - Runtime 객체 사용

    Runtime객체를 사용해 메모리 상태를 확인할 수 있다.

    ```java
    public class MemoryTest {
        public static void main(String[] args) {
            
            // 런타임 객체를 통해 메모리 상태를 확인
            Runtime runtime = Runtime.getRuntime();

            // 기본 타입으로 메모리 사용량 측정
            long[] classic = new long[1000000];
            long beforeMemory = runtime.totalMemory() - runtime.freeMemory(); // 메모리 사용 전
            
            for (int i = 0; i < classic.length; i++) {
                classic[i] = i;
            }
            
            long afterMemory = runtime.totalMemory() - runtime.freeMemory();  // 메모리 사용 후
            System.out.println("기본 타입(long) 메모리 사용량: " + (afterMemory - beforeMemory) + " bytes");

            // 가비지 컬렉션 수행
            System.gc();
            

            // 박싱된 기본 타입으로 메모리 사용량 측정
            Long[] boxed = new Long[1000000];
            beforeMemory = runtime.totalMemory() - runtime.freeMemory(); // 메모리 사용 전
            
            for (int i = 0; i < boxed.length; i++) {
                boxed[i] = (long) i;
            }
            
            afterMemory = runtime.totalMemory() - runtime.freeMemory();  // 메모리 사용 후
            System.out.println("박싱된 기본 타입(Long) 메모리 사용량: " + (afterMemory - beforeMemory) + " bytes");
        }
    }
    ```
    각각 1000000 길이의 long배열과 Long배열을 생성해서 모든 요소의 값을 업데이트 해주는 로직이다.  
    반복문이 실행되기 전 남아있는 메모리의 크기와 실행 후 남아있는 메모리의 크기를 비교하여 메모리 사용량을 간단히 확인해봤다.
    
    ```
    [결과]
    기본 타입(long) 메모리 사용량: 0 bytes
    박싱된 기본 타입(Long) 메모리 사용량: 25831848 bytes
    ```

    기본 타입의 연산은 메모리 사용량이 0 bytes로 측정될만큼 가벼웠지만, 박싱된 기본 타입은 약 25MB나 사용되었다.  
    예상된 결과이기 때문에 이런식으로 확인해볼 수 있구나 정도로 테스트 해봤지만 생각보다 더 많이 차이나긴 했다.  
    
    예상 외의 결과는 가비지 컬렉션을 수행하는 부분을 주석처리 했을 때가 두번째 메모리 사용량이 눈에 띄게 적었다는 것이다.  
    15015232 bytes 가 출력되었다. 왜 그럴까?  
    
    gc 메서드를 호출하는 것은 가상 머신이 차지하고 있는 메모리를 빠르게 재사용할 수 있게 해준다고 하지만,  
    gc 메서드를 호출하더라도 실제로 가비지 컬렉팅이 이루어진다는 보장이 없고, 오히려 수거할 메모리를 찾는 오버헤드가 크다.  
    수거할지 말지 모르는 메모리들을 확인하는데에 시간을 할애하기 때문에 성능 저하가 생길 수 있는 것이다.  
    그냥 가비지 컬렉터가 자동으로 처리하도록 두는 편이 나을 수도 있다.

<hr>

### 참고자료
- [Guide to System.gc()](https://www.baeldung.com/java-system-gc)