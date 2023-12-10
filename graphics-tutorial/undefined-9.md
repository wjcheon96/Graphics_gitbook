---
description: Normal Mapping
---

# ⚱ 노말 맵핑

해상도가 좋은 텍스쳐를 입혀도, 물체 표면의 요철 모양은 vertex의 갯수가 늘려져야 제대로 표현이 가능하다.

표면을 렌더링할 때 물체의 디테일을 결정하는 요소는 표면의 normal vector이기에, normal vector에 대한 텍스쳐 맵을 만들어서 각 픽셀의 normal방향을 변경시킨다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 오후 8.56.55.png" alt=""><figcaption></figcaption></figure>

아래와 같이 vertex의 갯수를 늘리지 않고, normal 벡터에 대한 texture map을 만들어서 geometric detail을 표현해낸다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 오후 8.58.18.png" alt=""><figcaption></figcaption></figure>













