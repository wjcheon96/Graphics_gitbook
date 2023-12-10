---
description: 왜 openGL인가?
---

# ❓ Why openGL?

* 크로스 플랫폼(Windows, linux, Android, Mac OS 등)을 지원한다. Metal이 출시되면서 Mac OS에서는 deprecated 하려는 추세이나, 현재까지는 어느 플랫폼에서든 사용 가능하다는 큰 장점이 있다.
* 자료가 많다. 오랫동안 사용되어온 대형 라이브러리로, 많은 유저층에서 오는 다양한 정보들이 많다는 특장점이 존재한다.

### Vulkan이 아닌 이유

OpenGL보다 low-level으로, GPU 하드웨어가 어떤식으로 작동 하는지에 대한 작동원리 및 관리 등에 대해 잘 알고있어야함. 기본적인 그래픽스에 대한 지식이 없을 경우, 학습에 어려움이 있음.

&#x20;OpenGL은 openGL es라는 모바일 전용 스펙이 따로 있지만, Vulkan은 태생적으로 모바일 기기와 동일한 스펙으로 나온다는 장점과, 멀티스레딩을 고려해서 만들어졌다는 장점이 있어, 배웠을때의 장점이 확실함.

해당 그래픽스 과제들은 3.3 버전을 사용할 것이며, 이유는 다음과 같다.

### openGL 3.3

* 가장 널리 사용되어온 OpenGL 버전
* Core-profile
  * 2.x 버전까지의 불필요하고 비효율 적인 기능 제거
  * 새로운 형태의 OpenGL context
*   Foward compatibility

    이후 버전에서도 동작할 것을 보장함.
