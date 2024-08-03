## 매개변수 개수 조정 방법에 대해

### 정리

**내용 요약**

메서드 시그니처 설계 요령은 다음과 같다.
1. 메서드 이름은 표중 명명 규칙을 따르며 이해하기 쉽고, 같은 패키지에 속한 다른 이름들과 일관되게 짓는다.
2. 모든 메서드가 자신의 소임을 다하도록 하자. 쓸데없이 쪼개지 않는다.
3. 매개변수 목록은 가능한 4개 이하로 유지하자.
    a. 여러 메서드로 쪼갠다. 쪼개진 메서드 각각은 원래 매개변수 목록의 부분집합을 받는다.
    b. 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다. 일반적으로 이런 도우미 클래스는 정적 멤버 클래스로 둔다.
    c. 빌더 패턴을 응용하여 세터를 통해 필요한 값만 설정할 수 있도록 한다.
4. 매개변수의 타입은 확장성을 고려하여 클래스보다 인터페이스가 낫다.
5. boolean타입 보다는 원소 2개짜리 열거타입을 통해 의미를 명확히 하고, 확장성을 고려하자.


### 심화 탐구

**출발점**

전체적으로 이해하기 어려운 내용은 아니었지만, 글에서 소개한 매개변수 목록 개수를 줄이는 방법 중 몇 가지의 실제 구현 사례가 떠오르지 않아서 GPT의 예시코드를 기반으로 수정하여 직접 용도를 알아보고자 했다.


**설명**

<hr>

1. 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다. (정적 멤버 클래스)

    원문에서 제시한 예시와 같은 상황을 코드로 작성해보았다.  
    우선 gpt가 특정 메서드에 사용할 매개변수(Card)를 정적 멤버 클래스로 작성해주었다.

    gpt가 제시한 예시 코드에서 추가 수정을 위해 고려한 부분은 2가지이다.  
    - 매개변수 몇 개를 독립된 하나의 개념으로 보고 정적 멤버 클래스로 묶을 수 있다.  
    - 같은 타입의 매개변수를 분리하여 헷갈리지 않도록 한다.

    playCard 메서드가 매개변수로 card 하나만 가지고 있다면 여느 메서드의 매개변수와 별다르지 않다.  
    (굳이 클래스로 만들어서 받는 이유가 없다는 의미, 물론 매개변수의 성격을 명시적으로 나타내는 장점은 있다.)  
    따라서 playCard라는 메서드에 게임 횟수(count)라는 요소를 추가했다.

    ```java
    public class CardGame {
	
        // Card 클래스는 suit(무늬)와 rank(숫자)를 묶어주는 도우미 클래스
        public static class Card {
            private final String suit;
            private final int rank;

            public Card(String suit, int rank) {
                this.suit = suit;
                this.rank = rank;
            }

            public int getRank() {
                return rank;
            }

            public String getSuit() {
                return suit;
            }

            @Override
            public String toString() {
                return rank + " of " + suit;
            }
        }

        public void playCard(Card card, int count) {
            
            // 카드와 카드의 게임 횟수를 가지고 play하는 로직이 들어간다고 가정할 수 있음
            // ex) card를 count번 ___한다....
            
        }

        public static void main(String[] args) {

            CardGame game = new CardGame();
            Card card = new Card("Spades", 1);
            game.playCard(card, 2);
            
        }
    }
    ```

    이렇게 작성하면 Card 클래스를 사용하지 않고 `void playCard(String suit, int rank, int count)`를 시그니처로 했을 때 int형 매개변수끼리 서로 헷갈릴 수 있는 상황을 해결할 수 있다.  
    독립되게 묶을 수 있는 suit와 rank 변수를 Card 클래스로 묶었기 때문이다.

2. 빌더 패턴을 응용하여 세터를 통해 필요한 값만 설정할 수 있도록 한다.

    1번의 코드 예시를 원문 세번째 방법인 빌더패턴에 맞게 수정해보았다.  
    playCard 메서드를 실행하기 위한 목적으로 CardGame 클래스를 구현한 것으로 생각하면 된다.  
    (playCard 메서드만 딸랑 있다면 매개변수 4개가 필요한 상황)  

    수정에 앞서 고려한 점은 3가지이다.
    - 메서드 실행에 있어 매개변수 중 생략 가능한 값이 있다. (replay 변수)
    - 특정 세터에서 같은 개념으로 묶이는 변수 2개를 같이 받을 수 있다. (suit, rank 변수)
    - 메서드 호출에 앞서 매개변수의 유효성을 검사한 후 실행하도록 한다. (count 변수)

    ```java
    public class CardGame {
        private String suit;
        private int rank;
        private int count;
        private boolean replay = false;
        
        public CardGame setCard(String suit, int rank) {
            this.suit = suit;
            this.rank = rank;
            return this;
        }
        
        public CardGame setCount(int count) {
            this.count = count;
            return this;
        }
        
        public CardGame setReplay(boolean replay) {
            this.replay = replay;
            return this;
        }

        public void playCard() {
            
            if (this.count == 0) {
                System.out.println("게임 실행 실패: 게임은 1회 이상부터 실행할 수 있습니다.");
                return;
            }
            
            // 카드와 카드의 게임 횟수를 가지고 play하는 로직이 들어간다고 가정
            // 특정 상황에서 게임을 재실행 할지의 여부를 결정 (기본값 false)
            // ex) card를 count번 ___한다.... ~~의 경우 게임을 재시작할지 replay를 통해 결정한다.
            
        }

        public static void main(String[] args) {

            CardGame game = new CardGame();
            game.setCard("Spades", 1)
                .setCount(3)
                .setReplay(true);
            
            game.playCard();
            
        }
    }
    ```
    위와 같이 작성하면 playCard를 실행하기에 앞서 게임에 필요한 변수들을 세터를 통해 빌드패턴으로 설정하도록 할 수 있다.  
    card에 관련된 무늬(suit), 숫자(rank) 변수는 setCard를 통해 함께 설정하도록 했다. (1번처럼 멤버 클래스를 둘 수 도 있지만, 이렇게 작성해봤다)  
    또, replay는 기본값을 false로 가지고 있기 때문에 설정하지 않아도 된다.  

    playCard 메서드 실행 시 count에 대한 유효성 검사가 실행되며, 조건 미충족시 안내 메시지를 return하고 이후 로직은 건너뛰도록 했다. suit, rank 변수에 대해서도 유효성 검사를 추가할 수 있을 것이다.


<hr>

