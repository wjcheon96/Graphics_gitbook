---
description: What is OpenGL
---

# 💡 OpenGL이란

### openGl

* 3D 그래픽을 화면에 표현하기 위해 필요한 기본적인 기능들을 모은 오픈 API
* 버전 업그레이드를 거쳐 현재 4.6버전까지 존재.
* SGI(실리콘 그래픽스)사의 그래픽 라이브러리 GL을 오픈소스화 한 툴.
* NVIDEA, Apple, Intel, Amd, 삼성 등의 기업들과 함께 Khronos Group이라는 운영회의를 만들어서 관리중임.

### Khronos Group

<figure><img src=".gitbook/assets/스크린샷 2023-08-27 18.33.36.png" alt=""><figcaption><p>Kronos Group에서 관리중인 툴</p></figcaption></figure>

### openGL의 사용처

* Real-Time 2D/3D rendering
  * 2D/3D game engine
  * VR/AR engine
  * Photo-realistic rendering
  * GUI component rendering
  * General-purpose computing
    * Image processing, neural network etc.

### OpenGL로 불가능한 것들

* Window management(OS part)
  * window를 만드는건 os의 역할. \
    ex) Windows의 경우 Win32 + wgl을 이용해서 openGL과 연결. Mac OS의 경우 Cocoa Framework에 요청.
* Input (keyboard/mouse) management(OS part)
* GUI framework(only for drawing)

OpenGL은 그리는 역할만을 진행하기에, GUI 컴포넌트가 어떤 역할을 하는데에 있어 따로 제공하는 기능이 없기에, 보통의 경우, 이를 제공하는 GUI 프레임워크 혹은 라이브러리를 이용한다. 크로스 플랫폼을 요구하는 경우에는 qt를 주로 사용한다.&#x20;



### OpenGL 특징

* Low-level API\
  저수준의 API로서 다양한 고수준의 Graphic API의 기반이 된다.\
  High level API의 예시로는 Unreal engine, unity, three.js(web)등이 있다.
* Cross-language & Cross-platform\
  C/C++이 기반이나, C#, JAVA, python, rust 등 다양한 언어에서 다 사용 가능하며, Cross platform을 지원하기에, OS에 구애받지 않는다.
* Standard / Specification only\
  Khronos Group에서 API의 스펙을 관리.\
  실제 OpenGL driver는 각 벤더에서 스펙에 맞게 관리.

### OpenGL 버전별 특징

* 1.x: Immediate mode, Standard pipeline\
  똑같은 형태의 그림을 그릴 수 있는 기본 파이프라인이 제공됨. Immediate mode는 프로그램이 그래픽을 디스플레이에 직접 담당하는 기능을 말한다.\
  파이프라인이 고정되어있고, 직접적으로 그리기에 너무 느림.
* 2.x: Vertex/fragment shader\
  파이프라인을 마음대로 변경 가능한 shader(programmable pipe) 기능이 추가됨. 이때도 immediate mode는 살아있었음.
* 3.x: New context, Forward compatibility
* 4.x: Compute shader

