# 📜 그래픽스 기본 세팅(cmake)

개발환경 세팅에서 Builder와 Dependency에 대한 설정법을 알아봤고, openGL을 사용하는데 있어 기본적으로 필요한 몇가지 dependency에 대한 세팅을 추가적으로 진행해줘야 한다.

cmake를 기반으로 서술한다.

### GLFW

```cmake
ExternalProject_Add(
    dep_glfw
    GIT_REPOSITORY "https://github.com/glfw/glfw.git"
    GIT_TAG "3.3.8"
    GIT_SHALLOW 1
    UPDATE_COMMAND ""
    PATCH_COMMAND ""
    TEST_COMMAND ""
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX=${DEP_INSTALL_DIR}
        -DGLFW_BUILD_EXAMPLES=OFF
        -DGLFW_BUILD_TESTS=OFF
        -DGLFW_BUILD_DOCS=OFF
)

set(DEP_LIST ${DEP_LIST} dep_glfw)
set(DEP_LIBS ${DEP_LIBS} libglfw3.a)

```

윈도우 생성(OS에 따라 다름), openGL을 위한 surface 생성 및 윈도우와 연결, 키보드 / 마우스 입력 연결 등의 기능을 추가하기 위해 필요한 라이브러리로, 과거에는 GLUT를 사용하였으나, 업데이트 및 서비스가 종료되었다. 현재 가장 많이 상용화된 라이브러리중 하나이다.

glfw를 추가하는 방법에는, [https://www.glfw.org/](https://www.glfw.org/) 위 주소에서 바이너리 파일을 가져와서 사용하는 방법과, 위와 같이 dependency에 github주소를 직접 가져와서 사용하는 등의 여러 방법중, 레포를 가져오는 방식을 택했다.



### GLAD

GL/GLES Loader-Generator로, openGL은 spec과 구현체(driver, dll)가 따로 존재하므로, openGL함수를 사용하기 전 해당 함수의 구현체가 어디있는지 로딩하는 과정이 필요하다.

```cmake
ExternalProject_Add(
    dep_glad
    GIT_REPOSITORY "https://github.com/Dav1dde/glad"
    GIT_TAG "v2.0.4"
    GIT_SHALLOW 1
    UPDATE_COMMAND ""
    PATCH_COMMAND ""
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX=${DEP_INSTALL_DIR}
        -DGLAD_INSTALL=ON
    TEST_COMMAND ""
    )
set(DEP_LIST ${DEP_LIST} dep_glad)
set(DEP_LIBS ${DEP_LIBS} glad)
```

위의 GLFW와 같은 방식으로 dependency.cmake에 위와 같이 추가해준다.\
GLAD는 openGL의 포인터를 관리하는데 사용될 것인데, 이는 OS마다 다른 OpenGL의 함수 포인터 주소를 로드하기에, GLAD를 통해서 이 과정을 관리한다.



### stb

Sean Barrett라는 인디 게임 제작자가 만든 라이브러리로, 이 중 stb\_image를 사용할것이며, 이는 jpg, png, tga, bmp, psd, gif, hdr, pic 포맷을 지원하는 이미지 로딩 라이브러리이다.

```cmake
# stb 설치
ExternalProject_Add(
    dep_stb
    GIT_REPOSITORY "https://github.com/nothings/stb"
    GIT_TAG "master"
    GIT_SHALLOW 1
    UPDATE_COMMAND ""
    PATCH_COMMAND ""
    # 헤더파일만 가져오기에 build 와 configure를 막아둔다.
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    TEST_COMMAND ""
        INSTALL_COMMAND ${CMAKE_COMMAND} -E copy
        ${PROJECT_BINARY_DIR}/dep_stb-prefix/src/dep_stb/stb_image.h
        ${DEP_INSTALL_DIR}/include/stb/stb_image.h
)
set(DEP_LIST ${DEP_LIST} dep_stb)
```

동일한 방식으로 stb 라이브러리도 설치가 가능하다.\
차이점은, stb는 single-file public domain library이기에, 헤더파일만 가져오므로, CONFIGURE\_COMMAND와 BUILD\_COMMAND도 ""으로 막아둔다.\
대신, install command를 따로 지정하는데, windows의 경우 copy 명령어를, linux/unix 계열의 경우 cp 명령어를 써서 복사해와도 되나, ${CMAKE\_COMMAND} -E copy 명령어를 통해서 OS에 구분 없이 cmake 명령어를 통해서 복사해오는 과정을 거친다.



### GLM

Glsl은 내부적으로 행렬 및 벡터에 대한 다양한 기능을 제공하나, C++은 해당 기능이 없어, 선형대수 관련 기능을 제공하는 라이브러리 중, openGL math 라이브러리이자, 3D그래픽스에 필요한 4차원 선형대수 연산을 가능하게 하는 glm을 사용한다.

```cmake
# glm
ExternalProject_Add(
    dep_glm
    GIT_REPOSITORY "https://github.com/g-truc/glm"
    GIT_TAG "0.9.9.8"
    GIT_SHALLOW 1
    UPDATE_COMMAND ""
    PATCH_COMMAND ""
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    TEST_COMMAND ""
    INSTALL_COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${PROJECT_BINARY_DIR}/dep_glm-prefix/src/dep_glm/glm
        ${DEP_INSTALL_DIR}/include/glm
)
set(DEP_LIST ${DEP_LIST} dep_glm)
```

glm은 stb와 동일하게, 헤더 라이브러리이므로, 디렉토리를 카피해와서 사용한다.



### ImGUI

ImGui는 위의 다른 dependency와 달리, 내부적으로 cmake나 makefile이 존재하지 않기에, 기본적으로 externalproject\_add의 방식의 사용이 불가능하다. 이를 2가지 방식을 통해 imgui를 가져올 수 있는데, 첫번째는 아래와 같다.

* [github.com/ocornut/imgui](https://github.com/ocornut/imgui)에 접속
* 최신 태그 선택 후 코드 다운로드
* 프로젝트 루트에 `imgui` 디렉토리 생성
* 라이브러리 루트의 소스 코드 및 라이센스 파일을 `imgui`에 복사
* `backend` 디렉토리에 있는 `imgui_impl_glfw.*` 파일을 `imgui`에 복사
* `backend` 디렉토리에 있는 `imgui_impl_opengl3.*` 파일을 `imgui`에 복사

이후 라이브러리를 직접 추가하는 방식을 통해 cmake에서 compile을 진행한다.

```cmake
#imgui
add_library(imgui
    imgui/imgui_draw.cpp
    imgui/imgui_tables.cpp
    imgui/imgui_widgets.cpp
    imgui/imgui.cpp
    imgui/imgui_impl_glfw.cpp
    imgui/imgui_impl_opengl3.cpp
)
target_include_directories(imgui PRIVATE ${DEP_INCLUDE_DIR})
add_dependencies(imgui ${DEP_LIST})
# Imgui 디렉토리를 include 헤더 디렉토리로 추가.
set(DEP_INCLUDE_DIR ${DEP_INCLUDE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/imgui)
set(DEP_LIST ${DEP_LIST} imgui)
set(DEP_LIBS ${DEP_LIBS} imgui)
```

해당 방식은, 외부 프로젝트가 아닌, 내부 프로젝트의 형태로 build를 했기에, /build/debug 디렉토리 내부에 lib 파일이 생성되며, 헤더는 프로젝트 경로에 있는 헤더 파일을 사용하게 된다.

두번째는, FetchContent를 사용하는 방식이다. FetchContent는 일반적으로 간단한 외부 라이브러리나 별도의 빌드 시스템이 없거나, 필요하지 않은 경우에 주로 쓰이는 방식이며, 사용방식은 아래와 같다.\
이렇게 빌드하게 되면 위 방식과 같이 별도로 파일을 로컬에 저장하고 있을 필요가 없다.

```cmake
#imgui
include(FetchContent)
FetchContent_Declare(
    imgui
    GIT_REPOSITORY https://github.com/ocornut/imgui.git
    GIT_TAG v1.82
)
FetchContent_GetProperties(imgui)
if (NOT imgui_POPULATED)
    FetchContent_Populate(imgui)
endif()
add_library(imgui
    ${imgui_SOURCE_DIR}/imgui_draw.cpp
    ${imgui_SOURCE_DIR}/imgui_tables.cpp
    ${imgui_SOURCE_DIR}/imgui_widgets.cpp
    ${imgui_SOURCE_DIR}/imgui.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp
)
target_include_directories(imgui PRIVATE ${DEP_INCLUDE_DIR})
add_dependencies(imgui ${DEP_LIST})
target_include_directories(imgui PRIVATE ${imgui_SOURCE_DIR})
set(DEP_LIST ${DEP_LIST} imgui)
set(DEP_LIBS ${DEP_LIBS} imgui)
```

### assimp

Open Asset Import Library의 약자로, 다양한 3D 에셋 파일 포멧을 위한 크로스 플랫폼 API이다. COLLADA(.dae), 3DS, DirectX X, Wavefront OBJ, Blender 3D(.blend) 등 다양한 파일 포멧을 지원한다.

```cmake
# assimp
ExternalProject_Add(
    dep_assimp
    GIT_REPOSITORY "https://github.com/assimp/assimp"
    GIT_TAG "v5.0.1"
    GIT_SHALLOW 1
    UPDATE_COMMAND ""
    PATCH_COMMAND ""
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX=${DEP_INSTALL_DIR}
        -DBUILD_SHARED_LIBS=OFF
        -DASSIMP_BUILD_ASSIMP_TOOLS=OFF
        -DASSIMP_BUILD_TESTS=OFF
        -DASSIMP_INJECT_DEBUG_POSTFIX=OFF
        -DASSIMP_BUILD_ZLIB=ON
    TEST_COMMAND ""
)
set(DEP_LIST ${DEP_LIST} dep_assimp)
if (APPLE)
    set(DEP_LIBS ${DEP_LIBS} assimp IrrXML zlibstatic)
endif()
if (WIN32)
    set(DEP_LIBS ${DEP_LIBS}
        assimp-vc142-mt$<$<CONFIG:Debug>:d>
        zlibstatic$<$<CONFIG:Debug>:d>
        IrrXML$<$<CONFIG:Debug>:d>
    )
endif()
```

