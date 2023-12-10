---
description: Shadowing
---

# 그림자

그림자는 입체감을 만드는 중요한 요소이다.

가장 간단하게 그릴 수 있는 그림자는 projective shadow 인데, 아래와 같이, 평면에 빛이 비추는 영역을 그려내는 방법이다. 다만, 해당 방법은 평면에 지는 그림자만 그릴 수 있다는 단점이 있다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 오후 8.44.40.png" alt="" width="375"><figcaption></figcaption></figure>

이에 따라, shadow map을 사용한다.



### shadow map

가장 많이 사용되는 그림자 렌더링 알고리즘으로, 구현 난이도가 높지 않으면서, 성능을 크게 잡아먹지 않는다. 또한, 고급 알고리즘으로의 확장이 편리하다. (ex. omnidiractional shadow map, cascaded shadow map)

방식은 다음과 같다.

일단, 빛의 시점에서 렌더링을 해본다. 빛이 볼 수 있는 부분은 그림자가 지지 않고, 빛이 볼 수 있는 부분은 그림자가 진다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 오후 8.47.15.png" alt="" width="375"><figcaption></figcaption></figure>

이후, light source의 위치에서 물체를 depth map만 렌더링한다(shadow map). Shadow map을 바탕으로 지금 그리고 있는 픽셀이 빛을 받는지를 판별한다. Light로부터 거리와 depth 값을 비교하여 계산하게 된다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 오후 8.48.47.png" alt="" width="563"><figcaption></figcaption></figure>

