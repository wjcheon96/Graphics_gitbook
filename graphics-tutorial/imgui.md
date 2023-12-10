---
description: Immediate-Mode GUI
---

# 🐧 ImGui

openGL에서 화면을 그리는데는 기본적으로 Shader, Program, VBO, VAO, EBO, Uniform, Texture 등이 필요한데, 이 외에도 button, slider 등을 구현하는데 gui적으로 필요한게 많은 만큼, gui를 사용하게 된다.

이 때, 해당 gui를 처리하는 방식은 두가지가 있다.

* 하나는 gui 프레임워크를 사용하여 gui 화면을 구성하고 그 중에서 openGL surface를 그리는 방식이 있다.\
  ex) win32, MFC, QT, Cocoa, Andriod 등..
* OpenGL 화면 내부에 GUI 컴포넌트를 만들고 이벤트를 처리할 라이브러리를 사용한다.

이 중, 우리가 사용하는 방법은 후자로, ImGUI를 활용한다.



### ImGUI 특징

* Immediate mode GUI.(매 렌더링 루프마다 UI를 그린다)
* Rendering 백엔드가 분리되어, openGL뿐만 아니라 다른 graphics API와 같이 사용이 가능하다.
* graphics programming에 필요한 GUI component를 갖고있다.(vector editor, Color picker 등)
* Nearly zero dependency로, 빌드가 간단하다.



### 사용법

main 함수 내에서 해당 헤더 두개를 추가한다. glfw및 opengl3 버전을 사용하기에 해당 헤더를 둘 다 추가해야한다.

```cpp
#include <imgui_impl_glfw.h>
#include <imgui_impl_opengl3.h>
```

#### ImGui 초기화

OpenGL context 초기화 직후, 즉 gladLoadGLLoader()함수가 호출된 이후에 imgui 관련 context들을 세팅한다.

```cpp
auto imguiContext = ImGui::CreateContext();
ImGui::SetCurrentContext(imguiContext);
ImGui_ImplGlfw_InitForOpenGL(window, false);
ImGui_ImplOpenGL3_Init();
ImGui_ImplOpenGL3_CreateFontsTexture();
ImGui_ImplOpenGL3_CreateDeviceObjects();
```

* `ImGUI::CreateContext()`: imgui를 위한 context를 생성한다.
* `ImGui::SetCurrentContext(imguiContext)`: 위에서 생성한 imgui context를 현재 사용할 context로 세팅한다.
* `ImGui_ImplGlfw_InitForOpenGl(window, false)`: impl\_glfw 내부의 initForOpenGL함수를 호출해 사용하는 window를 세팅하고, 두번째 인자로 callback 함수 세팅이 가능한데, 직접 진행하기 위해 디폴트 세팅은 false로 둔다.
* `ImGui_ImpleOpenGL3_init()`: openGL 3.3 version에 대한 렌더러를 초기화한다.
* `ImGui_ImplOpenGL3_CreateFontTexture()`: Imgui가 사용하고자 하는 font텍스처를 초기화한다.
* `ImGui_ImplOpenGL3_CreateDeviceObject()`: Imgui의 object(쉐이더 프로그램, 텍스처 등)의 초기화를 진행한다.

#### 메인 루프 변경

아래와 같이 Imgui관련 렌더를 추가한다.

```cpp
while (!glfwWindowShouldClose(window)) {
    glfwPollEvents();
    ImGui_ImplGlfw_NewFrame();
    ImGui::NewFrame();
    
    context->ProcessInput(window);
    context->Render();
    
    ImGui::Render();
    ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());

    glfwSwapBuffers(window);
}
```

* `ImGui_ImplGlfw_NewFrame()`: ImGui가 매 프레임마다 새롭게 계산된다. 해당 함수가 화면 크기나 마우스 상태. 즉 화면의 사이즈가 작아지거나 커져도 ImGui가 화면 내에서 안 사라지고 계속 보이게끔 한다.
* `ImGui::NewFrame()`: 새 프레임의 생성을 명시적으로 알려줌.
* `ImGui::Render()`: 실제 렌더 위에 그리는데, 실제로 그리진 않고, 이후에 렌더를 할 리스트를 종합하는 역할을 한다.
* `ImGui_ImpleOpenGL3_RenderDrawData(ImGui::GetDrawData())`: 실제 opengl3 function을 이용해서 ImGui를 그린다. 실제로 그려지는 작업이 `ImGui::GetDrawData()` 함수 내에서 이루어진다.

#### 종료 전 메모리 반환

위에서 초기화를 진행한 모든 Imgui 관련된 것들의 자원을 회수한다.

```cpp
ImGui_ImplOpenGL3_DestroyFontsTexture();
ImGui_ImplOpenGL3_DestroyDeviceObjects();
ImGui_ImplOpenGL3_Shutdown();
ImGui_ImplGlfw_Shutdown();
ImGui::DestroyContext(imguiContext);
```

#### 실제 ImGui 렌더링

아래와 같은 방식으로 ImGui를 렌더링 할 수 있다. void Context::Render() 함수 내부에 작성한다.

```cpp
if (ImGui::Begin("my first ImGui window")) {
    ImGui::Text("This is first text...");
} 
ImGui::End();
```



### UI/Parameter Binding

카메라 파라미터, 카메라 초기화 버튼, 클리어 색상등의 기능 추가.

해당 함수는 Context::Render() 함수 내부에 젤 윗줄에 집어넣는다.

```cpp
// window가 접혀있으면 해당 코드가 실행되지 않는다.
if (ImGui::Begin("UI Window")) {
    // 색 변환 함수. 각각의 파라미터는 label name, 그 뒤의 인자는 color를 집어넣는다.
    // color 값이 바뀌면 해당 로직이 실행된다.
    if (ImGui::ColorEdit4("clear color", glm::value_ptr(m_clearColor))) {
        glClearColor(m_clearColor.r, m_clearColor.g, m_clearColor.b, m_clearColor.a);
    }
    // 중간 구분선 추가.
    ImGui::Separator();
    // drag를 통해 카메라를 움직이게끔 할 수 있다.
    // 각각 pos, yaw, pitch 정보를 담아 드래그 시 값이 변경되면 그 값이 적용된다.
    ImGui::DragFloat3("camera pos", glm::value_ptr(m_cameraPos), 0.01f);
    ImGui::DragFloat("camera yaw", &m_cameraYaw, 0.5f);
    ImGui::DragFloat("camera pitch", &m_cameraPitch, 0.5f, -89.0f, 89.0f);
    ImGui::Separator();
    // reset 버튼이 눌리면 yaw, pitch, camera pos의 값을 초기화 해서 리셋시킨다.
    if (ImGui::Button("reset camera")) {
        m_cameraYaw = 0.0f;
        m_cameraPitch = 0.0f;
        m_cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
    }
}
ImGui::End();
```

* `ImGui::Begin(const char * name)`: 파라미터는 ui의 이름을 나타내고, boolean 값을 리턴한다. 이는 최소화와 같이 화면의 상태에 따라서 안 보일 때 해당 gui가 안 보이게 하기 위함이다.
* `ImGui::End()`: `ImGui::Begin()`함수가 있을때 꼭 있어야하며, ImGui를 종료시켜준다.
* `ImGui::Seperator()`: 중간의 구분선을 넣어준다.
* `ImGui::ColorEdit4(const char *label, float *col)`: x, y, z, a 네개의 값, 즉 rgba값을 변경할 수 있게 한다.
* `ImGui::DragFloat3(const char *label, float *y, float v_speed)`: Drag시 값을 변경시킬 수 있다. 첫번째가 ui 이름, 두번째 인자가 pointer 변수, 세번째부터는 오버로딩으로 각각 차이가 있는데, v\_speed는 속도를 지정할 수 있다.
* `ImGui::DragFloat(const char *label, float *y, float v_speed, float v_min, float v_max)`: 위의 dragFloat과 동일하게 사용하며, 최대, 최소 값을 지정한다.
* `ImGui::Button(const char *label)`: 이름을 가진 버튼을 만든다. 버튼이 눌릴 시 if 스코프 내부를 실행시킨다.



### ImGui Callback Setting

ImGui를 콜백으로 쓰기 위해서, 몇가지 사전 작업이 필요한데, 이를 위한 mouse scroll 콜백과, char 콜백을 지정해야 한다.

#### 마우스 스크롤

마우스 스크롤을 처리하는데 있어, glfw에서 scroll에 대한 callback을 등록해줘야 한다.

```cpp
glfwSetScrollCallback(window, OnScrollCallback);
```

그리고 OnScrollCallback은 다음과 같다.&#x20;

```cpp
void OnScrollCallback(GLFWwindow* window, double xoffset, double yoffset) {
    ImGui_ImplGlfw_ScrollCallback(window, xoffset, yoffset);
}
```

* `ImGui_ImplGlfw_Scrollcallback(GLFWwindow *window, double xoffset, double yoffset)`: 내부적으로 ImGUI에서 스크롤 이벤트 발생 시 스크롤이 처리되게끔 해준다.

#### Char 콜백

위와 동일하게 Char에 대한 콜백을 지정해줘야한다. ImGui 내에서 글자입력을 가능하게끔 하는데 쓰인다.

```cpp
glfwSetCharCallback(window, OnCharEvent);
```

OnCharEvent 콜백 함수는 아래와 같다.

```cpp
void OnCharEvent(GLFWwindow* window, unsigned int ch) {
    ImGui_ImplGlfw_CharCallback(window, ch);
}
```

* `ImGui_ImplGlfw_CharCallback(GLFWwindow *window, unsigned int c)`: 한 문자씩 입력된 글자를 ImGUI에 전달해준다.

#### 마우스 입력

드래그, 혹은 입력창을 누르는데 사용하기 위해 OnMouseButton 함수에 해당 콜백함수를 등록한다. ImGui내에 마우스 클릭을 통해 입력이 전달되게 한다.

```cpp
ImGui_ImplGlfw_MouseButtonCallback(window, button, action, modifier);
```

