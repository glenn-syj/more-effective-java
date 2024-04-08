# clone 메서드의 존재 이유에 대한 고찰



## item13 - clone 재정의는 주의해서 진행하라

### 🔍 내용 요약

clone 메서드를 잘 동작하게 하기 위해서는 고려해야할 사항이 많다. 
가장 큰 문제로는 clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected라는 데 있다. 
그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없고, 리플렉션(아이템 65)을 사용해도 100%의 신뢰성을 보여주지는 못한다. 
다음은 cloneable 인터페이스에서 주석을 제거한 코드이다.

```java
package java.lang;

public interface Cloneable {
}

// 인텔리제이 참고
```

이렇게 텅 빈 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정하는 메서드로 Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 
그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다. 
이는 일반적으로 인터페이스를 사용했다고 보기 어렵다. 
clone 메서드의 일반 규약은 허술하기도 해서 기대한대로 동작하게 하기 위해서는 지켜야 할 사항도 많다. 

<br>

--------------------------------------------------

### 🧐 심화 탐구

이번 아이템에서 소개한 clone 메서드를 보며 생성자와 같은 효과를 내려고 하는 메서드라는 생각이 들었다. 그렇다면 생성자가 있음에도 clone 메서드가 존재하는 이유는 무엇인지, 
clone 메서드가 필요한 상황은 어떤 상황일지 혹은 clone 메서드는 사용할 필요가 없는 구시대의 산물인지 다양한 궁금증이 들었다. 
이러한 궁금증 대해 알아보며 clone이 무엇인지 학습하는 것을 이번 심화 탐구의 주제로 삼았다. 

p.81에서 배열은 clone 기능을 제대로 사용하는 유일한 예라고 언급하고 있다. 

```java
// 직접 작성

int[] arr1 = {1, 2, 3};
int[] arr2 = arr1;
int[] arr3 = arr1.clone();

arr1[0] = 0;

System.out.println(arr1);    // [I@b4c966a
System.out.println(arr2);    // [I@b4c966a
System.out.println(arr3);    // [I@2f4d3709

System.out.println(Arrays.toString(arr1));    // [0, 2, 3]
System.out.println(Arrays.toString(arr2));    // [0, 2, 3]
System.out.println(Arrays.toString(arr3));    // [1, 2, 3]
```

다음과 같은 코드를 통해 1차원 배열에서 clone 메서드를 사용했을 경우 사용자가 기대한대로 깊은 복사가 된 것을 볼 수 있었다. 

```java
int[][] arr1 = {{1, 2}, {3, 4}};
int[][] arr2 = arr1.clone();

arr1[0][0] = 0;

System.out.println(arr1);    // [[I@b4c966a
for(int i=0 ; i<2 ; i++){
    System.out.println(arr1[i]);    // [I@2f4d3709 [I@4e50df2e
    System.out.println(Arrays.toString(arr1[i]));    // [0, 2] [3, 4]
}

System.out.println(arr2);    // [[I@1d81eb93
for(int i=0 ; i<2 ; i++){
    System.out.println(arr2[i]);    // [I@2f4d3709 [I@4e50df2e
    System.out.println(Arrays.toString(arr2[i]));    // [0, 2] [3, 4]
}
```

다음은 2차원 배열에서 clone 메서드를 사용한 후 출력된 결과이다. 2차원 배열에서 clone을 하니 2차원 배열 자체는 새로운 메모리 주소를 갖고 있지만 
각각의 1차원 배열은 동일한 메모리 주소를 참조하고 있고 arr1의 값을 변경하니 arr2의 값도 변경된 모습을 볼 수 있다. 

위 코드는 ChatGPT에서 아이디어를 얻어 진행한 실험인데 ChatGPT에서 clone 메서드는 객체의 얕은 복사를 수행한다는 설명이 있었다. 
여기서 말하는 얕은 복사는 배열 객체 자체는 깊은 복사가 되지만 배열의 요소들은 같은 객체를 참조한다는 의미였다. 
이러한 특징 때문에 1차원 배열과 달리 2차원 배열에서는 기대한대로 복사가 진행되지 않았다. 

해당 코드를 작성하며 그래도 clone 메서드가 약간은 의미를 갖을 수 있다는 점을 발견했다. 
바로 내가 선택한 객체와 동일한 속성을 갖는 새로운 객체를 얻고자 할 때 활용할 수 있다는 점이다. 
물론 일일이 생성자를 통해 값을 넣어 동일한 속성을 갖는 새로운 객체를 얻을 수도 있지만 clone 메서드 하나로 같은 효과를 누릴 수 있다면 그것만으로도 존재가치가 있다는 생각이 들었다. 

교재에서 Cloneable을 구현한 class에 대해 clone이 제대로 동작하지 않는 예시들 모두 이러한 얕은 복사가 될 수 있는 부분을 놓져서 생기는 문제로 보인다. 
위 코드 역시 다차원 배열로 갈수록 까다롭게 clone을 재정의 해야 동작할 것으로 보이며 이러한 clone의 단점을 대체하고자 복사 생성자와 복사 팩터리라는 객체 복사 방식을 소개하고 있다. 
나 역시 이미 Cloneable을 구현한 class를 다루게 된다면 충분히 이해하고 활용하는 방안을 고민하겠지만, 처음부터 객체 복사를 염두해두고 class 구조를 짤 경우 
이와 같은 방식이 더 깔끔하고 합리적으로 보인다. 

참고자료 : https://docs.oracle.com/javase/8/docs/api/ (java.lang / Object / Cloneable 등의 키워드)

참고자료 : ChatGPT (배열의 클래스 / Cloneable 번역 등의 키워드)

--------------------------------------------------

### 🧠 어려웠던 점

- 평소 ctrl + click을 통해 내부 구조를 보며 글을 작성해보고자 하였는데 clone의 경우 해당 기능을 사용할 수 없어 구체적인 구조를 파악하기 어려웠다. 
- 자바에서는 모든 것이 객체라는 말을 얼핏 들었던거 같아서 그럼 배열도 어떠한 class에서 구현하고 있고 해당 class가 clone 역시 구현하지 않을까 했는데 공식 문서에서는 Object class라고만 나와서 이에 대한 이해를 충분히 하기 어려웠다. 
- clone이라는 메서드를 한번도 사용해본적이 없어 존재에 대한 정당성을 부여하고자 하였는데, 나의 견해가 clone의 존재 이유와 동일한지에 대한 확신이 없다. 
