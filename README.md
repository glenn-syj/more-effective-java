# What is more-effective-java?

**최종 수정일: 2024/03/15**

## 1. Organization

### 1-1. Introduction

**more-effective-java** 는 2024년 SSAFY (Samsung Software Academy For Youth) 내에서 조직된 자바 스터디 그룹입니다. `Effective Java` (3판)을 읽으며 자신의 관심에 맞춰 정리하고, 심화적인 탐구를 진행합니다. (물론 내용 모두를 요약하지는 않습니다!) 토론/논의는 비정기적인 대면 모임이나 github 내 issue 탭에서 진행합니다!

### 1-2. Contributors

**ready to be written**

## 2. Conventions

### 2-1. Core Values

```java
(1) 목적: Effective Java를 통한 자바 학습 및 비판적 사고력 향상
(2) 책임: 주어진 분량을 모두 읽고, 적어도 읽은 아이템의 반에 대해 문서 작성
(3) 도전: 내용 이해에서 그치지 않고 이유와 비판점을 고민
(4) 효율: 타인의 글에 대해서 꼼꼼히 읽고 활발한 논의하기
(5) 기한: "일요일~월요일 넘어가는 자정까지" 문서 제출 하기
(6) 위약: 미인정 사유로 2회 불참 시 퇴장 (인정 사유는 적용 X)
```

### 2-2. Organization Rules

```java
(1) 타인의 노고와 작성 글에 대해 쉽게 말하지 않기
(2) 논의를 넓히는 방식으로 질문하고 대답하기
(3) 사람이 아닌 내용에 대해 칭찬하기
(4) 자신의 관점이나 의견에 대해 trade-off를 고려해보기
(5) docs, issue, ... 어떤 경우에든 공적인 말투로 게시하기
```

### 2-3. Submission

**Project/Pacakge/File Format**

```java
📂 chapter_{챕터 번호}
 └── 📄 item{아이템 번호}_{이름}_{글 제목}.md
ex) 📄 item01_손영준_정적-팩터리-메서드란-무엇인가.md

chapter_01에 대해서는 item00_{이름}_{제목}으로 통일
Item{아이템 번호}: item01, item02, ..., item90
{제목}: 띄어쓰기 자리에 hypen('-') 이용

```

**Fork and Pull Request**

```java
(1) 각자 원본 Repository에서 "자신의 브랜치" 생성한 뒤에 Fork 하기
(2) 이후 "자신의 Forked Repository"에서 Commit, Push 등 작업하기
(3) Forked Repository에서 원본 Repository 내 "자신의 브랜치"로 Pull Request
    -> Convention 검사를 위함입니다
```


### 2-3. Conventional Commits

```java
// 1. 기본 커밋 메시지 형식

(1) git commit -m "[MEJ-<주차>] <Type>: <Title>"

    예를 들어서 1주차의 item01에 관해 팩토리 메서드를 다루는 작성했다면
    O: git commit -m "[MEJ-001] Docs: item04 정적 팩토리 메소드 관련 글 작성"
    X: git commit -m "[MEJ-001] Docs: item04" / "MEJ-001 Docs: itme04" / "item04"
		
// 2. 커밋 메시지 컨벤션

(1) 본문과 꼬릿말은 선택적 (궁금하신 분은 질문... 쓸 일 크게 없을 것 같기도)
    - 그래도 아래와 같이 본문에 간단한 요약을 추가해둬도 좋을 것 같습니다 
    ```
    git commit -m "[MEJ-001] Docs: 정적 메소드 팩토리에 관한 글 작성

    - 정적 메소드 팩토리에 대한 개념
    - 정적 메소드 팩토리를 쓰는 이유
    - 다른 디자인 패턴과의 비교"
    ```
(2) <주차>는 000, 001, 002, ..., 010, 011, ... 
(3) <Type> 작성법

        A. <Type> 종류
        
            Init: Repository 초기화 작업에 사용
            Docs: 새로운 문서를 작성해서 제출할 때
            Fix: 문서 상에 존재하는 오타, 내용 오류, 코드 등을 고칠 때
            Chore: 파일명 변경 등 내용에 영향을 미치지 않는 작업 
            
        B. <Type> 시작은 대문자로 쓰기

            O: Docs, Fix, Chore, ...
            X: docs, fix, chore, ...

(4) <Title> 작성법

    - "item04 정적 팩토리 메소드 관련 글 작성" 등
    - "<item> <contents>" 만 지켜지면 OK
    - 끝에 마침표('.') 찍지 않기

// 3. 기타 문의사항

간단한 것이라도 좋으니 무조건 물어보기!
Issue 탭을 적극적으로 활용하자!
git 관련 어려운 것도 언제나...
```