---
description: Cross Platform build system setting
---

# 💫 크로스 플랫폼(CMake)

https://cmakge.org에서 다운 가능하며, windows는 인스톨러, MacOS에서는 brew, Linux(ubuntu)에서는 apt를 통해 설치 가능하다.

* MacOS

```bash
brew install cmake
```

* Linux(ubuntu)

```bash
sudo apt-get install cmake
```



### CMake 작동 방식

#### Meta-project description

* 플랫폼 / 선호하는 빌드 방식에 맞는 프로젝트 파일 생성
* Windows: visual studio project
* Linux: Makefile
* macOS: Xcode / Makefile
* Andriod: Ninja Build



### CMake 프로젝트 생성

CMakeLists.txt 파일 생성. 대소문자 구분이 되어야 한다.

```cmake
# 설치 된 cmake 최소 버전 이상에 대한 제약.
cmake_minimum_required(VERSION 3.27.3)

# set을 통해 변수 지정 가능
set(PROJECT_NAME cmake_project_example)
set(CMAKE_CXX_STANDARD 17)

# 세팅할 프로젝트에 대한 이름 설정
project(${PROJECT_NAME})

# 만들 실행파일에 대한 소스 파일 지정
add_executable(${PROJECT_NAME} src/main.cpp)
```

아래와 같은 방식(예시)으로 빌드한다.

```bash
cmake -S ${CMAKELISTS.TXT} -B ${BUILD_DIR}
# CMakeLists.txt가 위치한 위치, cmake가 빌드될 위치 지정

cmake -Bbuild . -DCMAKE_BUILD_TYPE=[Debug|Release]

cmake --build build --config Debug
```



### Visual Studio에서 configure 세팅하는 법

`cmd` + `shift` + `p` 를 누르고, cmake를 검색해서 컴파일 환경을 지정하는 방식을 사용할 수 있다.

<figure><img src="../../.gitbook/assets/스크린샷 2023-08-29 00.01.09.png" alt=""><figcaption></figcaption></figure>

이후 cmake build또한 같은 방식으로 빌드 설정이 가능하다.

<figure><img src="../../.gitbook/assets/스크린샷 2023-08-29 00.02.12.png" alt=""><figcaption></figcaption></figure>

Build: `F7`

Debug: `ctrl` + `F5` &#x20;

와 같이 configure가 되있는 상태에서 위와같이 편리하게 사용이 가능하다.

{% hint style="warning" %}
MacOS(M1)의 경우 `ctrl` + `F5` 사용시 디버깅에서 gdb가 없어 에러가 발생한다.\
따라서 디버깅 없이 실행(`shift` + `F5)`를 사용하여 실행하도록 한다.
{% endhint %}

{% hint style="danger" %}
CMake configure, build시 fail이 되는 경우 cmake에 대한 PATH 설정이 제대로 안 되어있는 경우가 있다. 이 경우 `cmd`+ `,` 를 눌러 system preference를 열고, cmake path 검색을 통해 CMake Path 에 실제 존재하는 cmake path 경로를 추가해준다.
{% endhint %}

<figure><img src="../../.gitbook/assets/Screen Shot 2023-08-29 at 4.25.31 PM.png" alt=""><figcaption><p>Path 설정 예시</p></figcaption></figure>



### CMakeLists.txt 예시

```cmake
cmake_minimum_required(VERSION 3.27.2)

set(PROJECT_NAME scop)
set(CMAKE_CXX_STANDARD 17)

project(${PROJECT_NAME})

# 내부 소스를 건드릴 시 해당 위치를 사용.
add_executable(${PROJECT_NAME} src/main.cpp)

# ExternalProject 관련 명령어 셋 추가
include(ExternalProject)

# 외부 라이브러리 추가시 Dependency.cmake를 건드린다.
include(Dependency.cmake)

# 우리 프로젝트에 include / lib 관련 옵션 추가
target_include_directories(${PROJECT_NAME} PUBLIC ${DEP_INCLUDE_DIR})
target_include_directories(${PROJECT_NAME} PUBLIC ${DEP_INSTALL_DIR})
target_link_directories(${PROJECT_NAME} PUBLIC ${DEP_LIB_DIR})
target_link_libraries(${PROJECT_NAME} PUBLIC ${DEP_LIBS})

# Dependency들이 먼저 build 될 수 있게 관계 설정
add_dependencies(${PROJECT_NAME} ${DEP_LIST})
```

### Dependency.cmake 예시

외부 라이브러리 사용시 아래 파일에 추가적인 설정을 추가함으로서 관리. 해당 dependency에는 spdlog(로그 관련 외부 라이브러리)에 대한 내용이 예시로 들어있다.

```cmake
# Dependency 관련 변수 설정
set(DEP_INSTALL_DIR ${PROJECT_BINARY_DIR}/install)
set(DEP_INCLUDE_DIR ${DEP_INSTALL_DIR}/include)
set(DEP_LIB_DIR ${DEP_INSTALL_DIR}/lib)

# spdlog: fast logger library
# spdlog를 다운받아서 가져옴. 로그를 보는데 용이.
ExternalProject_Add(
    dep-spdlog
    GIT_REPOSITORY "https://github.com/gabime/spdlog.git"
    GIT_TAG "v1.x"
    GIT_SHALLOW 1
    UPDATE_COMMAND ""
    PATCH_COMMAND ""
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${DEP_INSTALL_DIR}
    TEST_COMMAND ""
)

# Dependency 리스트 및 라이브러리 파일 리스트 추가
set(DEP_LIST ${DEP_LIST} dep-spdlog)
# OS 별 생성 파일이 .lib 와 .a 파일로 달라지므로, Mac 기준에 맞춰서 해당 디렉토리에 있는 링크를 제대로 맞춰줌.ㄴ
if (APPLE)
    set(DEP_LIBS ${DEP_LIBS} libspdlog.a)
endif()
if (WIN32)
# Windows 에서 spdlog를 위의 방식으로 받아올 시 사용
    set(DEP_LIBS ${DEP_LIBS} spdlog$<$<CONFIG:Debug>:d>)
endif()
```



### 플랫폼 간 차이가 있을때 조건 주는 방법

apple, windows 간 OS 차이로 cmake 설정을 달리 할 수 있다.

if (APPLE) 혹은 if (WIN32) 와 같이 OS를 지정해주면 cmake가 OS를 읽고 해당 OS에 맞춰서 빌드 세팅을 진행해준다. 조건을 다 쓰고 난 뒤에는 endif()와 같이 끝에 ()를 넣어주어야 한다.
