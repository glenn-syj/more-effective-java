# 사용자 정의 예외를 통한 매개변수 유효성 검사

## item49 - 매개변수가 유효한지 검사하라

### 🔍 내용 요약

메서드나 생성자를 작성할 때면 그 매개변수들에 어떤 제약이 있을지 생각해야 한다. 그 제약들을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사해야 한다.
예를 들면 인덱스 값은 음수이면 안 되며, 객체 참조는 null이 아니어야 한다. 이런 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다.
오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워진다.

매개변수 검사를 제대로 하지 못하면 몇 가지 문제가 생길 수 있다.

- 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
- 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.
- 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 이 메서드와는 관련 없는 오류를 낼 수 있다.

public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야하며 매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야 한다.

<br>

---

### 🧐 심화 탐구

아이템49의 내용은 충분히 이해하고 공감할 수 있는 간단한 내용이었다는 생각이 들었다.
메서드의 파라미터가 특정 조건을 만족하는 값만 들어오기를 기대한다면 메서드가 어떠한 역할을 수행하기 전에 파라미터가 특정 조건을 만족하는지 확인하고 만족하지 않는다면 예외를 던져서 오류의 발생 지점을 쉽게 알 수 있게 할 뿐만 아니라 실패 원자성까지 지킬 수 있기 때문이다.
실패 원자성(item 76)이란 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다는 의미로 데이터베이스에서 유사한 개념을 이미 학습했을테니 충분히 이해할 수 있는 내용인 것 같다.

이번 심화 탐구는 이전에 초미니 프로젝트로 코딩 테스트 서버 구축 프로젝트를 하며 이번 탐구 주제와 비슷한 고민을 했던 것을 돌아보는 것을 탐구 주제로 선정했다.

개발 당시에는 어떤 방식으로 코딩 테스트 서버를 구축해야 할지 감이 안와서 Front-end에서 유저가 제출한 코드를 서버에 보내면 서버에서는 이를 String으로 받아서 파일로 저장한 후 컴파일하는 방식으로 구현했다. 다음은 Service의 채점을 담당하는 로직의 일부이다.

```java
// 사용자가 제출한 코드를 받아서 처리
public Map<String, List<String>> grade(String userCode) throws CompilationErrorException, IOException, InterruptedException {
    String path = "src/main/java/com/example/demo/test/";
    String className = "Main";

    // 제출 코드를 파일로 저장
    saveUserCode(userCode, path + "Main.java");

    // 코드 컴파일
    try {
        compileUserCode(path + "Main.java");
    }
    catch (CompilationErrorException e){
        throw e;
    }

    // 실행 결과 저장
    List<String> output = runUserCode(className, path + "input1.txt");

    System.out.println(output);

    System.out.println("실행 결과 저장 성공");

    // 정답 저장
    List<String> answer = new ArrayList<>();
    BufferedReader br = new BufferedReader(new FileReader(path + "answer1.txt"));
    String line;
    while ((line = br.readLine()) != null) {
        answer.add(line);
    }
    br.close();

    System.out.println("정답 저장 성공");

    // 컴파일 및 실행 결과 저장할 리스트
    List<String> result = compareOutput(output, answer);

    System.out.println("비교 성공");

    Map<String, List<String>> model = new HashMap<>();
    model.put("answer", answer);
    model.put("output", output);
    model.put("result", result);

    return model;
}
```

여기서 코드 컴파일이라고 써있는 주석에서 던지는 CompilationErrorException의 경우 정상적으로 컴파일이 되지 않는 코드(문자열)를 입력받으면 던지는 예외로 자바에서 적절한 예외를 찾지 못해서 직접 정의한 예외이다.

```java
public class CompilationErrorException extends Exception {
    public CompilationErrorException(String message) {
        super(message);
    }
}
```

CompilationErrorException은 Exception을 상속받아 message를 던지는 것으로 간단하게 구현했다.
해당 프로젝트를 진행하는 동안 동적 컴파일에 대한 이해가 많이 부족하고 예외처리가 어려워서 ChatGPT의 도움을 받아서 구현했다.
이번 item을 학습하며 사용자 정의 예외를 통해 에러가 발생했을 때, 좀 더 원인을 쉽게 찾을 수 있도록 구현하려는 노력은 보였던 것 같지만 특별히 메시지를 던져주거나 자바독 태그를 통해 문서화까지는 생각하지 못해서 보완할 점이 필요해보였다.

개발 과정에서 실제로 해당 에러가 많이 발생해서 디버깅하는 과정에서는 수월했지만, item72 표준 예외를 사용하라 파트를 참고하며 사용자 정의 예외를 반드시 사용할만한 상황인가에 대한 고민은 충분히 필요할 것 같다는 생각이 들었다.

---

### 🧠 어려웠던 점
