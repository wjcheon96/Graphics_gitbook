---
description: openGL simple function
---

# 💿 openGL 기본 함수

OpenGL을 사용하는데 있어 기본적으로 사용되는 내용들을 간단하게 정리한다.



먼저 헤더를 추가할 때, 아래의 두 헤더를 추가하며, glad가 glfw보다 먼저 포함되어야한다.\
Glad 안에는 GL/gl.h 와 같은 openGL 헤더파일이 포함되어있기 떄문이다.

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
```

아래와 같이 glfw 창을 인스턴스화 할 main함수와, glfw를 초기화한다.

```cpp
int main(int ac, char **av) {
    std::cout << "Initialize glfw" << std::endl;
    if (!glfwInit()) {
        const char* description = NULL;
        glfwGetError(&description);
        std::cout << "Failed to initialize glfw: " << description << std::endl;
        return -1;
    }
}
```

`glfwWindowHint()`를 통해 GLFW를 설정한다. \
아래의 hint는 버전명시와, core\_profile을 사용하여 openGL의 기능의 최소집단까지 사용하길 원한다는걸 명시한다.\
`FORWARD_COMPAT` 경우, MacOS 에서 사용하기 위함이다.

```cpp
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
glfwWindowHint (GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
```

아래와 같이 window를 생성한다. 이후 `glfwMakeContextCurrent()`를 통해 `glfwCreateWindow()`가 호출시 생성된 스레드의 context를 glfw에 지정해준다.

```cpp
auto window = glfwCreateWindow(WINDOW_WIDTH, WINDOW_HEIGHT, WINDOW_NAME, nullptr, nullptr);
if (!window) {
    std::cout << "failed to create glfw window" << std::endl;
    glfwTerminate();
    return -1;
}
glfwMakeContextCurrent(window);
```

GLAD를 통해 openGL용 함수 포인터를 관리하는 과정을 거친다. `glfwGetProcAddress()` 함수를 통해 process address를 얻어오고, 이를 glad 라이브러리로 openGL 함수를 로딩한다.

```cpp
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
    std::cout << "failed to initialize glad" << std::endl;
    glfwTerminate();
    return -1;
}
```

#### State-setting function

`glViewport()`는 openGL에게 렌더링 윈도우의 사이즈를 알려주고, `glClearColor()`는 window가 clear 될 때 어떤 색으로 칠해질 것인지를 알려준다. 이러한 부류의 함수들을 state-setting function이라 하며, state가 openGL context에 저장되는 함수들을 나타낸다.

```cpp
glViewport(0, 0, width, height);
glClearColor(0.0f, 0.1f, 0.2f, 0.0f);
```

#### State-using function

`glClear()`는 실제 frame buffer를 클리어하며, openGL context에 저장된 state를 이용하는데, 이러한 함수들을 state-using function이라 한다.

```cpp
glClear(GL_COLOR_BUFFER_BIT);
```

#### Double buffer

현재 모던 그래픽스에선 실제 랜더링 과정시 double buffer 이상의 buffer를 사용하는데, 이때 double buffer는 front / back의 두개의 버퍼를 준비하여 그림을 그리는 방법을 뜻한다. front buffer에 그림을 바로 그리게 될 시 그려지는 과정까지 보여져 다소 더러운 결과값이 보이는데(왼쪽 위 픽셀부터 오른쪽 아래 픽셀까지 그려지는 과정이 보임), 이러한 과정은 사용자 경험이 떨어지는 등의 결함이 있기에, front buffer는 최종 출력 이미지를 담고, back buffer에는 모든 랜더링 명령을 담아 랜더링을 한 뒤, 이 명령이 완료되면 back buffer를 front buffer로 교체(swap) 하는 과정을 거친다. 이때 사용되는 함수가 `glfwSwapBuffer()` 이다.

```cpp
glfwSwapBuffers(window);
```

#### Event callback

* `glfwWindowShouldClose()`: 루프가 시작될때마다 돌면서 glfw가 종료를 명령했는지를 확인하고, true가 반환되면 루프를 종료시킨다.
* `glfwPollEvents()`: loop가 돌면서 window 창이 지속되는데, 이때 생기는 event에 대한 감지를 가능하게 하는 함수.
* `glfwXXXCallback(window, func)`: 이벤트 발생시, 윈도우 상태를 업데이트하며 정해진 콜백함수를 등록하여 호출하는데, glfwXXXCallback()과 같은 방식으로 사용되며, 각각 실행중인 window와 callback으로 등록할 함수를 매개변수로 갖는다.

```cpp
glfwSetFramebufferSizeCallback(window, OnFramebufferSizeChange);
glfwSetKeyCallback(window, OnKeyEvent);

while (!glfwWindowShouldClose(window)) {
    glfwPollEvents();
}
```



#### Terminate

랜더링 루프가 종료될 때, 할당된 모든 자원을 정리 / 삭제 해야한다.\
이를 진행해주는 함수가 `glfwTerminate()`함수이다.

```cpp
glfwTerminate();
```

