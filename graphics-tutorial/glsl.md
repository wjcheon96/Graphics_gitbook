---
description: OpenGL Shading Language
---

# 🔥 GLSL

### OpenGL Shader Language

Shader: GPU 상에서 동작하는그림을 그리기 위한 작은 프로그램

정점별 / 픽셀별 병렬 수행을 통해 성능을 높임.

GLSL: OpenGL에서 쉐이더를 작성하기 위해 제공하는 C기반 언어.

이 외의 다른 shader language

* HLSL: DiretX용 shader language
* Metal: Metal용 shader language
* cg: nVidia가 제시한 shader language



### GLSL에서 사용 가능한 data type

기본 타입: int, float, double, uint, bool

벡터 타입:

* vecX: float형 벡터
* bvecX: bool형 벡터
* ivecX: int 형 벡터
*  uvecX: uint형 벡터
* dvecX: double형 벡터

X에는 2, 3, 4가 들어가며, 1이 필요한 경우엔 기본 타입을 사용한다.

행렬 타입:

* matX: float형 행렬
* bmatX: bool형 행렬
* imatX: int형 행렬
* umatX: uint형 행렬
* dmatX: double형 행렬



### 벡터 원소 접근

.x, .y, .z, .w인덱스를 사용

swizzling 가능

* 얻어오고 싶은 인덱스를 연속으로 쓰기 가능
* ex) .xyz => vec3

.rgba, .stpq도 동일한 방식으로 사용 가능.

```glsl
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

#### 벡터 초기값 선언

* 생성자 사용
* 다른 벡터를 섞어서 사용 가능

```glsl
vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0);
vec4 otherResult = vec4(result.xyz, 1.0);
```



### IN / OUT 한정자

Shader의 입출력 구조로, shader는 용도에 맞는 입출력이 선언되어있어야하고, 이를 in, out으로 shader의 입출력을 가리키게하는 한정자로써 쓴다.

#### Vertex shader

* 각 정점별로 설정된 vertex attribute를 입력받는다.
* 몇번째 attribute를 입력받는지에 대한 추가적인 설정이 가능하다.
  * layout ( location = n )
* 정점의 출력 위치 gl\_Position값을 계산해주어야 한다.

#### Rasterization

* Vertex shader의 출력값을 primitive(삼각형, line 등 내가 그리고자 하는 것)에 맞게 보간하여 픽셀별 값으로 변환하는 과정

#### Fragment shader

* Rasterization 과정을 거쳐 픽셀별로 할당된 vertex shader의 출력값이 입력된다.
* 각 픽셀의 실제 색상 값이 출력되어야 함.



### Uniform 한정자

Shader에 전달 가능한 global value

* 뱅렬로 수행되는 모든 shader thread들이 동일한 값을 전달받는다.
* 변수 선언 앞에 uniform type qualifier를 써서 선언.

```glsl
#version 330 core
out vec4 FragColor;

uniform vec4 ourColor; // uniform 변수

void main() {
  FragColor = ourColor;
}
```

#### Uniform variable에 값을 입력하는 방법

glGetUniformLocation()을 사용해 program object로부터 uniform handle을 얻는다. 이후, program이 바인딩 된 상태에서 glUniform...()을 사용해 값을 입력한다.

```cpp
auto loc = glGetUniformLocation(m_program->Get(), "color");
m_program->Use();
glUniform4f(loc, 1.0f, 1.0f, 0.0f, 1.0f);
```

#### glGetUniformLocation()

프로그램 인스턴스의 아이디를 가져와서 세팅하고, 두번쨰 인자로 uniform variable에 들어갈 변수 명을 string형태로 입력한다.

#### glUniform...()

위의 예시에서는 glUniform4f를 썼는데, uniform 뒤에 오는 내용을 통해 floating 포인트의 갯수를 지정한다거나 하는 등의 인자의 개수와 타입을 지정할 수 있게끔 한다.



### 예시

위가 Vertex Shader, 아래가 Fragment Shader로, 중간에 Rasterization stage를 거쳐 vertex shader 의 out이 fragment shader로 넘어가면서 색상을 지정하게 된다. vertex shader의 out과 fragment shader의 in의 type과 변수명이 같아야 컴파일이 제대로 진행된다.

```glsl
#version 330 core
layout (location = 0) in vec3 aPos; // 0번째 attribute가 정점의 위치

out vec4 vertexColor; // fragment shader로 넘어갈 컬러값

void main() {
  gl_Position = vec4(aPos, 1.0); // vec3를 vec4 생성자에 사용, 기본으로 써줘야 한다.
  vertexColor = vec4(0.5, 0.0, 0.0, 1.0); // 어두운 빨간색을 출력값으로
}
```

```glsl
#version 330 core

in vec4 vertexColor; // vs로부터 입력된 변수 (같은 변수명, 같은 타입)
out vec4 FragColor; // 최종 출력 색상

void main() {
  FragColor = vertexColor;
}
```



### Vertex attribute가 여러개일때 지정하는 방법

하나의 vertex가 지니는 정보(ex. position, normal, tangent, color etc..)가 여러가지일 수 있다. 각각이 하나의 vertex attribute가 된다.

```cpp
float vertices[] = {
  0.5f, 0.5f, 0.0f, 1.0f, 0.0f, 0.0f, // top right, red
  0.5f, -0.5f, 0.0f, 0.0f, 1.0f, 0.0f, // bottom right, green
  -0.5f, -0.5f, 0.0f, 0.0f, 0.0f, 1.0f, // bottom left, blue
  -0.5f, 0.5f, 0.0f, 1.0f, 1.0f, 0.0f, // top left, yellow
};

m_vertexBuffer = Buffer::CreateWithData(GL_ARRAY_BUFFER, GL_STATIC_DRAW,
  vertices, sizeof(float) * 24);

m_vertexLayout->SetAttrib(0, 3, GL_FLOAT, GL_FALSE,
  sizeof(float) * 6, 0);
m_vertexLayout->SetAttrib(1, 3, GL_FLOAT, GL_FALSE,
  sizeof(float) * 6, sizeof(float) * 3);
```

vertex 정보가 여섯개로 늘어나면서, 세개씩 끊어서 앞의 세개를 SetAttribute함수 내의 glVertexAttribPointer()내부에서 각각 0번, 1번 offset을 사용하게 되며, stride 값이 하나의 점의 정보가 2개의 attribute를 담기에, 각각 3개씩 쓰므로 6개의 값을 가지게 된다.

각각의 attribute와 stride 값은 다음과 같다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-05 11.58.01.png" alt=""><figcaption></figcaption></figure>

이에 맞춰 각각 glsl파일을 수정한다.

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;

out vec4 vertexColor;

void main() {
    gl_Position = vec4(aPos, 1.0);
    vertexColor = vec4(aColor, 1.0);
}
```

```glsl
#version 330 core
in vec4 vertexColor;
out vec4 fragColor;

void main() {
    fragColor = vertexColor;
}

```

attribute index에 맞춰서 location 값을 수정해줘야 한다.

