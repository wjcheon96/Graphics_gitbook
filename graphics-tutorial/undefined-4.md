---
description: Coordinate System
---

# 🏺 좌표계

어떤 정점의 위치를 기술하기 위한 기준으로, 선형 변환은 한 좌표계로 기술된 벡터를 다른 좌표게에 대해 기술하는 변환으로 해석 가능하다.

#### 자주 사용되는 좌표계 용어

* World space: 기준이 되는 좌표계로, view는 world를 기준으로 잡혀있다.
* Local(Object) space: Object에 대한 좌표계
* View(Eye) space: 실제로 보는 방향을 따지는 좌표계
* Screen space: 실제 화면에 나타냈을때의 좌표계'

#### 좌표 공간 간의 변환

OpenGL의 그림이 그려지는 공간은 \[-1, 1] 범위로 normalized된 canonical space(공간)을 그리는데, object들은 local space를 기준으로 좌표를 기술한다.\
Local space -> world space -> view space -> Canonical space로 변환한다.\
Local space -> World space -> View space로의 변환 과정을 model-view transform이라 부른다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-07 at 9.56.03 PM.png" alt=""><figcaption><p>mvp(model-view-projection) matrix</p></figcaption></figure>

#### Transform matrix

* Model: Local을 World로
* View: World를 Camera로
* Projection: Camera를 Canonical로
* Clip space에서 \[-1, 1] 범위 밖으로 벗어난 면들은 clipping

Local space를 Clip space로 좌표로 옮기는 방식은 결과론적으로 다음과 같다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-07 at 10.07.40 PM.png" alt="" width="375"><figcaption></figcaption></figure>

### 직교 투영

원근감 없이 평행한 선이 계속 평행하도록 투영하는 방식으로, 설게 도면 등을 그리는데 주로 쓰인다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-07 at 10.00.15 PM.png" alt="" width="375"><figcaption></figcaption></figure>

* 6개 파라미터: left, right, bottom, top, near, far
* z축에 -1: Clip space 이후에는 오른손 좌표계에서 왼손 좌표계로 변경

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-07 at 10.00.52 PM.png" alt="" width="184"><figcaption></figcaption></figure>

#### 오른손 좌표계 / 왼손 좌표계

x축, y축을 화면의 오른/위 방향으로 했을때, 각각 z축이 화면에서부터 나오는 / 화면으로 들어가는 방향

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-07 at 10.01.53 PM.png" alt="" width="188"><figcaption></figcaption></figure>



### 원근투영

변환 이전에 평행한 선이 변환 후에 한 점에서 만난다(소실점이 존재). 멀리 있을수록 물체가 작아지기에, 원근감이 발생한다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-07 at 10.03.09 PM.png" alt="" width="375"><figcaption></figcaption></figure>

* 4개의 파라미터
  * 종횡비 (aspect ratio)
  * 화각 (field of view, FoV)
  * near, far

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-07 at 10.03.44 PM.png" alt="" width="375"><figcaption></figcaption></figure>

near, far plain이 어디서부터 어디까지 볼 지를 정해주어야 Canonical space(\[-1, 1]사이의 압축된 정육면체 영역)에서 z값이 소실되는 현상을 막을 수 있다.



### Depth Buffer

Z 버퍼(Z buffer)라고도 하며, 픽셀의 컬러 값을 저장하는 버퍼 외에 해당 픽셀의 깊이값(z축 값)을 저장한다.

#### 깊이 테스트

1. 어떤 픽셀의 값을 업데이트 하기 전에, 현재 그리려는 픽셀의 z값과 깊이 버퍼에 저장된 해당 위치의 z 값을 비교한다.
2. 비교 결과 현재 그리려는 픽셀이 이전에 그려진 픽셀보다 뒤에 있을 경우 픽셀을 그리지 않음

#### OpenGL Depth Buffer

OpenGL의 Depth Buffer 초기값은 1으로, 1이 가장 뒤에 있고, 0이 가장 앞에 있다(왼손 좌표계).

* `glEnable(GL_DEPTH_TEST)` / `glDisable(GL_DEPTH_TEST)`로 깊이 테스트를 켜고 끌 수 있다(default는 disable 상태).
* `glDepthFunc()`을 이용하여 깊이 테스트 통과 조건을 변경할 수 있다(기본값은 `GL_LESS`).

```cpp
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
glEnable(GL_DEPTH_TEST);
```

