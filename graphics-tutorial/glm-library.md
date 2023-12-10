---
description: glm.h
---

# ⚖ GLM Library

glsl은 내부적으로 행렬 및 벡터와 관련된 다양한 기능 및 내부 함수를 제공하나, c++은 기본적인 수학적 연산(float, double과 같은 scalar 값)을 제외한 선형대수적 기능을 제공하지 않는다. 따라서 선형대수와 관련된 라이브러리가 크게 2개가 있다.

* Eigen3: C++ 선형대수 라이브러리중 가장 많이 사용됨.\
  일반적으로 N차원 선형대수 연산에 쓰인다.
*   GLM: openGL math 라이브러리

    3D 그래픽스에 필요한 4차원 선형대수 연산을 위한 라이브러리다.

따라서 GLM라이브러리를 사용한다.



### 필요한 헤더 추가

```cpp
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
```

glm을 쓸때 기본적인 헤더를 추가한다. 행렬 형변환과, 타입과 관련된 헤더를 추가한다.



### 정점 변환 방식

정점에 대한 변환의 일반적인 방식은 VBO상의 정점은 고정하고, vertex shader에서 변환 행렬을 uniform으로 입력하고, vertex shader내에서 행렬곱 계산을 하는 방식 순서로 이루어진다.

```glsl
uniform mat4 transform;

gl_Position = transform * vec4(aPos, 1.0);
```

위와 같이 VBO상의 정점을 고정하고, 아래의 방식과 같이 transform을 진행하여 uniform으로 보낸다.

```cpp
auto transform = glm::rotate(glm::scale(glm::mat4(1.0f), glm::vec3(0.5f)),
    glm::radians(90.0f), glm::vec3(0.0f, 0.0f, 1.0f)
);
auto transformLoc = glGetUniformLocation(m_program->Get(), "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(transform));
```

