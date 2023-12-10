---
description: camera
---

# 📸 Camera

3D 공간을 어느 시점에서 어떤 방향으로 바라볼 것인가를 결정한다. 카메라를 조작하기 위한 파라미터로부터 view transform을 유도한다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-08 16.53.15.png" alt=""><figcaption></figcaption></figure>

#### 카메라 파라미터

* camera position: 카메라의 위치
* camera target: 카메라가 바라보는 중심 위치
* camera up vector: 카메라 화면의 세로 축 방향

#### 결과 행렬

camera의 local-to-world transform(카메라의 지역 좌표계를 world 좌표계로 바꿔주는 행렬)의 inverse가 된다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-08 16.55.31.png" alt="" width="188"><figcaption></figcaption></figure>

#### &#x20;카메라의 3축 결정 방식

<figure><img src="../.gitbook/assets/스크린샷 2023-09-08 16.59.10.png" alt="" width="188"><figcaption></figcaption></figure>

* `z축 벡터`: e 벡터와 방향이 동일해야하는데, 1의 크기를 가져야 하므로, 절대값으로 나눈다.
* `x축 벡터`: u 벡터가 y 벡터 방향과 거의 비슷하나, z축과 수직이 아닐 수 있기에, 이를 외적해서 z축과 u 방향에 수직한 벡터를 구한다.
* `y축 벡터`: z축 벡터와 x축 벡터를 외적해서 구한다.&#x20;



### 카메라를 실제로 연산하는 방법

```cpp
auto cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
auto cameraTarget = glm::vec3(0.0f, 0.0f, 0.0f);
auto cameraUp = glm::vec3(0.0f, 1.0f, 0.0f);

auto cameraZ = glm::normalize(cameraPos - cameraTarget);
auto cameraX = glm::normalize(glm::cross(cameraUp, cameraZ));
auto cameraY = glm::cross(cameraZ, cameraX);

auto cameraMat = glm::mat4(
    glm::vec4(cameraX, 0.0f),
    glm::vec4(cameraY, 0.0f),
    glm::vec4(cameraZ, 0.0f),
    glm::vec4(cameraPos, 1.0f));

auto view = glm::inverse(cameraMat);
```

위의 방식으로 camera x, y, z를 전부 계산하고, 결과 행렬이 world coordinate으로 바꾸는 과정에서 inverse를 시켜줘야 하므로, inverse를 진행한다.

#### glm::lookAt()

위의 카메라 연산을 glm::lookAt()함수를 이용하면 계산을 한번에 할 수 있다.

```cpp
auto cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
auto cameraTarget = glm::vec3(0.0f, 0.0f, 0.0f);
auto cameraUp = glm::vec3(0.0f, 1.0f, 0.0f);

auto view = glm::lookAt(cameraPos, cameraTarget, cameraUp);
```



### 키입력을 통한 카메라 회전

wasdqe의 6가지 키입력을 통해, 전후상하좌우에 대해 카메라의 위치를 조정 한다.

```cpp
void Context::ProcessInput(GLFWwindow* window) {
    const float cameraSpeed = 0.05f;

    if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
        m_cameraPos += cameraSpeed * m_cameraFront;
    if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
        m_cameraPos -= cameraSpeed * m_cameraFront;
        
    auto cameraLeft = glm::normalize(glm::cross(m_cameraUp, m_cameraFront));
    if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
        m_cameraPos -= cameraSpeed * cameraLeft;
    if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
        m_cameraPos += cameraSpeed * cameraLeft;

    auto cameraUp = glm::normalize(glm::cross(m_cameraFront, cameraLeft));
    if (glfwGetKey(window, GLFW_KEY_E) == GLFW_PRESS)
        m_cameraPos += cameraSpeed * cameraUp;
    if (glfwGetKey(window, GLFW_KEY_Q) == GLFW_PRESS)
        m_cameraPos -= cameraSpeed * cameraUp;
}
```

`glfwGetKey(GLFWwindow* window, int key)`함수를 통해,  keypress가 된 key에 따라 위치를 잡는다.

cameraPos가 실제 카메라의 위치를 나타내므로, 카메라의 위치 변경을 통해 위치를 잡는다.

위의 카메라 연산과정에서 보았듯, CameraFront가 정면에서 보이는 벡터였기에, cameraUp 벡터와 m\_cameraFront 벡터를 외적해서 왼쪽으로 이동 가능하게, front 벡터와 left벡터 외적을 통해 위아래로 이동이 가능하게 한다.

위의 실제 연산 코드에서, 각각 cameraZ, cameraX, cameraY의 연산과 동일하다.

이후, 해당 메소드를 main 루프에서 호출해 버튼이 눌렸을때 움직임이 가능하게끔 처리한다.



### 마우스 입력

마우스는 좌, 우 버튼에 대한 입력과 스크롤에 대한 입력 두가지로 나뉘는데, 버튼으로 입력은 아래의 회전각 표현에서 다루기로 하고, 스크롤에 대한 입력만 해당 파트에서 기술한다. 기본 골자는 비슷하다. \
아래의 함수를 통해 scroll이 들어왔을때에 대한 콜백함수를 지정한다.

```cpp
glfwSetScrollCallback(GLFWwindow* window, GLFWkeyfun callback);
```

해당 콜백함수에서는 스크롤 입력시 줌인, 줌아웃을 담당하므로, cameraPos와 cameaFront 벡터를 사용하기 위해 context를 받아오고, 해당 context로 넘겨준다.

```cpp
void OnScrollCallback(GLFWwindow* window, double xoffset, double yoffset) {
    auto context = (Context *)glfwGetWindowUserPointer(window);
    context->MouseScroll(yoffset);    
}
```

이후 context 객체 내의 메소드로 스크롤 입력에 대한 이벤트를 처리한다.

```cpp
void Context::MouseScroll(double yoffset) {
    float cameraSpeed = 0.15f;
    if (yoffset < 0) {
        m_cameraPos += cameraSpeed * m_cameraFront;
    }
    else if (yoffset > 0) {
        m_cameraPos -= cameraSpeed * m_cameraFront;
    }
}
```



### 회전각 표현

보통의 경우에서 오일러 각도(Euler Angle)을 사용한다.

roll(z), pitch(x), yaw(y) 3개의 회전각을 지닌다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-11 at 8.09.47 PM.png" alt=""><figcaption></figcaption></figure>

실제 비행기를 조작할때 쓰는 법이다.

* Pitch: 우리나라 말로는 끄덕각이라고도 하며, 앞뒤 전환을 말한다.(x축 회전)
* Yaw: 도리각이라고도 하며, 방향을 바꾸는데 사용하는 각을 말한다.(y축 회전)
* Roll: 갸웃각이라고도 하며, 좌 우로 기울어지는 각을 나타낸다(z축 회전)

#### 실제 카메라 회전

사람이 쓰러지는 것을 표현하는 케이스가 아닌 경우를 제외하곤 거의 roll은 사용하지 않으며, 대신 up vector를 기준으로 roll을 설정한다. 따라서, yaw와 pitch만을 이용해서 camera front 방향을 결정짓는다.

이전 position과 현재 position 사이의 차이를 지정한 속도에 따라 변환이 진행되게 한다.

y축으로 회전시 360도가 넘어가면 0도로 초기화하여, 0 \~ 360 도 사이의 값을 갖게 하며, pitch는 +90 \~ -90 사이의 값을 갖게 한다.

```cpp
auto pos = glm::vec2((float)x, (float)y);
auto deltaPos = pos - m_prevMousePos;
static glm::vec2 prevPos = glm::vec2((float)x, (float)y);

const float cameraRotSpeed = 0.8f;
// yaw와 pitch를 계산한다. rotspeed를 조절하여 너무 빠른 전환이 안 일어나게끔 한다.
m_cameraYaw -= deltaPos.x * cameraRotSpeed;
m_cameraPitch -= deltaPos.y * cameraRotSpeed;

// 사이즈가 각도를 초과하면 0 ~ 360 사이의 값으로 다시 바꿔줌.
if (m_cameraYaw < 0.0f)   m_cameraYaw += 360.0f;
if (m_cameraYaw > 360.0f) m_cameraYaw -= 360.0f;

if (m_cameraPitch > 89.0f)  m_cameraPitch = 89.0f;
if (m_cameraPitch < -89.0f) m_cameraPitch = -89.0f;
```

* `GLFW_MOUSE_BUTTON_LEFT`를 통해서 마우스 좌클릭을 확인 가능하다.

```cpp
if (button == GLFW_MOUSE_BUTTON_CENTER) {
    if (action == GLFW_PRESS) {
      // 마우스 조작 시작 시점에 현재 마우스 커서 위치 저장
      m_prevMousePos = glm::vec2((float)x, (float)y);
      m_cameraControl = true;
    }
}
```

* `glfwGetCursorPos(GLFWwindow* window, double* xpos, double* ypos)`: 창 내 마우스 커서의 위치를 지정 가능하다.
* 각각 `glfwSetCursorPosCallback()`과 `glfwSetMouseButtonCallBack()`은 마우스 커서의 position 변경시와 버튼 클릭 여부에 따라서 해당 callbak 함수를 호출한다.

```cpp
glfwGetCursorPos(window, &x, &y);
glfwSetCursorPosCallback(window, OnCursorPos);
glfwSetMouseButtonCallback(window, OnMouseButton);
```



### 화면 Resizing

아래와 같이 화면이 변경되면, glViewport 값을 수정해서, 그려지는 영역을 재조정하게끔 바꾼다.

```cpp
void Context::Reshape(int width, int height) {
    m_width = width;
    m_height = height;
    glViewport(0, 0, m_width, m_height);
}
```

아래의 코드에서, glfwSetWindowUserPointer() 함수를 통해서, window에 context에 대한 pointer를 세팅할 수 있다. 이를 통해 전역에서 불러오는 것 과 같이 glfwGetWindowUserPointer()를 통해 불러올 수 있다.

```cpp
glfwSetWindowUserPointer(window, context.get());
OnFramebufferSizeChange(window, WINDOW_WIDTH, WINDOW_HEIGHT);
glfwSetFramebufferSizeCallback(window, OnFramebufferSizeChange);
glfwSetKeyCallback(window, OnKeyEvent);
```

onFramebufferSizeChange()함수에서 context를 불러오고,  reshape 함수에서 glViewport()함수를 호출해 변환을 시킨다.

```cpp
void OnFramebufferSizeChange(GLFWwindow* window, int width, int height) {
    SPDLOG_INFO("framebuffer size changed: ({} x {})", width, height);
    // glViewport를 통해 opengl이 그릴 영역을 지정해줌.
    auto context = (Context*)glfwGetWindowUserPointer(window);
    context->Reshape(width, height);
}
```

* `glfwSetWindowUserPointer(GLFWwindow* window, void* pointer)`: Window에 포인터를 세팅할 수 있다. 이를 통해, 전역에서 해당 context 포인터를 가져오는것과 같이 포인터를 세팅하고, 원하는 위치에서 가져올 수 있다.
* `glfwGetWindowUserPointer(GLFWwindow* window)`: setWindowUserPointer 에서 세팅한 포인터를 가져온다.

