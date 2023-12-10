---
description: graphics pipeline
---

# 🖇 그래픽스 파이프라인

<figure><img src="../.gitbook/assets/스크린샷 2023-09-03 20.11.57.png" alt=""><figcaption><p>그림이 그려지는 과정</p></figcaption></figure>

### Application 프로그램 영역

#### Application : 그리고 싶은 정점의 위치 / 색상을 입력. 3D 그래픽스에서 기본 입력 단위는 삼각형이다. OpenGL함수를 호출하는 영역이다.



### GPU 영역

#### Geometry : 정점 단위의 처리. 정점의 위치 결정

#### Rasterization : 정점 단위로 구성된 삼각형을 픽셀 단위로 변경

#### Pixel : 픽셀 단위의 처리. 픽셀 색상 결정.



### Programmable shader

#### Shader

각 파이프라인 단계마다 GPU 상에서 실행되는 작은 프로그램

화면에 출력할 픽셀의 위치와 색상을 계산하는 함수

GLSL(GL Shading Language)라는 C 기반 프로그래밍 언어로 작성

<figure><img src="../.gitbook/assets/스크린샷 2023-09-03 20.18.41.png" alt=""><figcaption></figcaption></figure>

### OpenGL shader

OpenGL에서 그림을 그리기 위해선 위의 그림에서 보이는 vertext shader와 Fragment shader가 모두 필요하다. Shader 코드는 openGL 코드 내에서 빌드 및 로딩이 진행된다.

#### Vertex shader(정점 쉐이더)

각각의 vertex(정점)의 위치를 계산하는 쉐이더

#### Geometry shader

Vertex shader와 Fragment shader의 사이에 있는 선택적인 shader로, 점이나 삼각형같은 하나의 기본 도형을 이루는 vertex들의 모음을 받고, 이 정점들을 다음 shader 단계에 이들을 보내기 전에 적절한 형태로 변환시켜 사용이 가능하다. 또한, geometry shader는 정점들을 원래 주어진 정점들보다 더 많은 정점들을 생성하는 완전히 다른 기본 타입 도형으로 변환시킬 수 있다는 장점을 지닌다.

#### Fragment shader

Pixel shader라고도 불리며, 각각의 픽셀별로 실행함.



### Shader Class Design

OpenGL shader object를 가지고 있으며, 인스턴스가 생성될 때 로딩할 파일명을 입력받는다. 입력된 파일명으로부터 인스턴스 생성이 실패하면 메모리 할당을 해제하는 방식으로 제작. C++11에서 쓸 수 있는 smart pointer 사용.

