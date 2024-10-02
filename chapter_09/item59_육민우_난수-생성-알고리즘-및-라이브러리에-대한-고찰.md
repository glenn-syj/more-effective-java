# 난수 생성 알고리즘 및 라이브러리에 대한 고찰

## item59 - 라이브러리를 익히고 사용하라

### 🔍 내용 요약

_바퀴를 다시 발명하지 말자._
이번 아이템을 관통하는 표현으로 필자가 핵심 정리에 적어놓았는데, 아주 특별한 나만의 기능이 아니라면 누군가 이미 라이브러리 형태로 구현해놓았을 가능성이 크다. 그런 라이브러리가 있으면 그냥 쓰면 되며 본인이 직접 작성하는 것보다 코드의 품질이 높을 확률이 크다.

<br>

---

### 🧐 심화 탐구

본인도 제법 공감하는 내용으로 본인이 개발하는 핵심 비즈니스 로직의 경우 라이브러리를 찾게 되어도 직접 구현해보며 개발하는 것도 좋다고 생각하지만 보조적인 기능들의 경우 라이브러리가 충분히 좋은 대안, 좋은 정답이 될 수 있다고 생각한다. 난수 생성 예제를 통해 방향성을 잡아주는 내용의 아이템이어서 비슷한 고민을 했던 프로젝트의 경험과 예제의 내용을 풀어내는 것을 이번 심화 탐구의 주제로 삼았다.

난수 생성의 경우 자주 사용하게 되는 로직이지만 개발자가 직접 훌륭한 난수 생성을 개발하는 것은 어렵다고 생각한다. 난수를 생성할 때 흔히 떠올리는 자바의 Random 클래스의 경우도 선형 합동 생성기(Linear congruential generator)라는 의사난수 생성기로 동작한다고 나와있는데 난수에 대한 예측이 가능해서 암호학적으로 안전하지 않음을 언급하고 있다. 생성된 난수의 이미지를 출력했을 때, 일정한 패턴이 보일 것으로 기대했지만 이미지 상으로는 어떠한 패턴까지는 발견하지 못했다. (ChatGPT의 질의응답에서 난수의 주기가 매우 커서 이미지에서 패턴을 확인하기는 어려울 것이라는 답변정도만 받을 수 있었다.)

반면 교재(p.351)의 코드 59-1의 경우 n이 그리 크지 않은 2의 제곱수라면 같은 수열이 반복된다는 점에서 아이디어를 얻어서 이를 자바로 시각화하는 코드를 작성해보았다.

```java
package item59;

import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.Random;

public class Example {

    static Random rnd = new Random();

    static int random(int n){
        return Math.abs(rnd.nextInt()) % n;
    }

    public static void main(String[] args) throws IOException {
        final int SIZE = 1024;
        BufferedImage image = new BufferedImage(SIZE, SIZE, BufferedImage.TYPE_INT_RGB);

        for (int y = 0; y < SIZE; y++) {
            for (int x = 0; x < SIZE; x++) {
                if (random(1024) % 2 == 0) {
                    image.setRGB(x, y, Color.WHITE.getRGB());
                }
            }
        }

        ImageIO.write(image, "png", new File("example.png"));
    }
}
```

BufferedImage 클래스를 통해 1024 \* 1024 사이즈의 png 파일을 만드는 코드로 각 픽셀에 대해 해당 난수가 2의 배수인지 아닌지로 흑백 점을 찍었다. 실제 출력된 이미지를 보면 육안으로 일정한 패턴이 나타남을 확인할 수 있다. 이를 통해 개발자는 난수를 통해 불규칙한 패턴의 이미지를 얻으려고 했지만 예상과 다르게 동작하는 프로그램 때문에 어려움을 겪을 수 있음을 볼 수 있었다.

최근 진행하는 프로젝트에서 6자리의 유니크한 수를 얻어야하는 이슈가 있었다. 각각의 서버에서 동일하게 겪는 이슈였고 수를 얻는 방법이 제각각이어서 본인은 난수 생성을 통해 문제를 해결하고자 했다. 해당 수가 API의 key가 되는 값이어서 ThreadLocalRandom과 SecureRandom, 난수 생성 API 3가지 방안을 고민했고 성능보다는 보안 이슈를 더 중요하게 생각해서 SecureRandom을 통해 난수를 생성을 최종 선택했다. 해당 key의 안전하고 유니크한 생성도 물론 중요하지만 핵심 비즈니스 로직과는 거리가 있었기에 직접 개발하는 것보다 좋은 품질의 코드를 제공하는 라이브러리를 사용했고 비둘기 집의 원리를 통한 중복 확률 판단과 중복 키 예외 처리를 통해 개발을 마쳤던 경험이 있다.

ref : https://docs.oracle.com/javase/8/docs/api/index.html?java/util/Random.html

ref : https://ko.wikipedia.org/wiki/%EC%84%A0%ED%98%95_%ED%95%A9%EB%8F%99_%EC%83%9D%EC%84%B1%EA%B8%B0

<br>

---

### 🧠 고민해볼 점
