---
description: Draw Triangle
---

# 📐 삼각형 및 사각형 그리기

### 정점 입력

정점을 세개 입력하여 삼각형을 그릴 수 있으며, 아래와 같은 방식을 따른다.

* 정점 데이터 준비
* Vertex buffer object(VBO) 준비
* Vertex buffer object에 정점 데이터 입력
  * CPU memory 상에 있는 정점 데이터를 GPU에 옮기는 작업
* Vertex array object(VAO) 준비
  * 우리의 정점 데이터의 구조를 알려주는 descriptor object
* Program, VBO, VAO를 사용해 그림 그리기



### Vertex Buffer Object(VBO)

정점 데이터를 담은 버퍼 오프젝트. 정점에 대한 다양한 데이터를 GPU가 접근 가능한 메모리에 저장해둔다.\
ex) position, normal, tangent, color, texture, coordinate...



### VBO에 복사하는 데이터의 구조

<figure><img src="../.gitbook/assets/스크린샷 2023-09-04 02.28.07.png" alt=""><figcaption></figcaption></figure>

* 정점이 총 3개
* 각 정점의 위치가 기록
* 위치에 대해서 x/y/z 좌표값을 가짐
* 각 좌표값마다 float = 4byte의 크기를 가짐
* 첫번째 정점과 두번째 정점 간의 간격이 12byte만큼 차이남.
* VBO가 가진 정점에 대한 구조(Layout)을 알려줄 방법이 있어야함.



### Vertex Array Object(VAO)

정점 데이터의 구조를 알려주는 오브젝트로, 각 정점이 몇 byte로 구성되었는지, 두 정점이 몇 byte만큼 떨어져있는지, 정점의 0번째 데이터는 어떤 사이즈의 데이터가 몇개 있는 형태인지 등의 정보를 갖고있다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-04 02.31.58.png" alt=""><figcaption></figcaption></figure>



### Element Buffer Object(EBO)

사각형을 그릴때, 정점이 4개가 필요하나, glDrawArrays()를 이용해 그림을 그리려면 정점이 6개가 필요하다. 즉, 정점이 2개가 중복되는것인데, 이를 해결하기 위한 object이다.

* 정점의 인덱스를 저장할 수 있는 버퍼
* 인덱스 버퍼라고도 부름
* 정점 정보와 별개로 정점의 인덱스로만 구성된 삼각형 정보를 전달 가능
* Indexed Drawing



### 사용 가능한 Primitive Types

<figure><img src="../.gitbook/assets/스크린샷 2023-09-04 02.57.20.png" alt=""><figcaption></figcaption></figure>



### 함수 설명

```cpp
glGenVertexArrays(1, &m_vertexArrayObject);
glBindVertexArray(m_vertexArrayObject);
```

#### `glGenVertexArrays()`: VAO 생성

#### `glBindVertexArray()`: 지금부터 사용할 VAO 설정

```cpp
glGenBuffers(1, &m_vertexBuffer);
glBindBuffer(GL_ARRAY_BUFFER, m_vertexBuffer);
glBufferData(GL_ARRAY_BUFFER, sizeof(float) * 9, vertices, GL_STATIC_DRAW);
```

#### `glGenBuffers()`: 새로운 buffer object 생성

#### `glBindBuffer()`: 지금부터 사용할 buffer object 지정

* `GL_ARRAY_BUFFER`: 사용할 buffer object가 vertex data를 저장할 용도임을 알려준다.&#x20;

#### glBufferData(): 지정된 buffer에 데이터를 복사한다.

* 데이터의 총 크기, 데이터 포인터, 용도 지정
* "STATIC | DYNAMIC | STREAM", "DRAW | COPY | READ" 의 조합으로 용도를 지정한다.
* flag 설명
  * `GL_STATIC_DRAW`: 딱 한번만 세팅되고 앞으로 계속 쓸 예정
  * `GL_DYNAMIC_DRAW`: 앞으로 데이터가 자주 바뀔 예정
  * `GL_STREAM_DRAW`: 딱 한번만 세팅되고 몇번 쓰다 버려질 예정

```cpp
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(float) * 3, 0);
```

#### `glEnableVertexAttribArray(n)`: 정점 attribute 중 `n`번째를 사용하도록 설정

#### `glVertexAttribPointer(n, size, type, normailzed, stride, offset)`

* `n`: 정점의 `n`번째 attribute
* `size`: 해당 attribute는 몇개의 값으로 구성되어 있는가?
* `type`: 해당 attribute의 데이터 타입
* `normalized`: 0\~1사이의 값인가
* `stride`: 두 정점간의 간격 (byte 단위)
* `offset`: 첫 정점의 헤당 attribute까지의 간격 (byte 단위)

#### `glDrawArray(primitive, offset, count)`

* 현재 설정된 program, VBO, VAO로 그림을 그린다
* `primitive`: 그리고자 하는 primitive 타입, GL\_TRIANGLES, GL\_LINE\_STRIP 등이 있다.&#x20;
* `offset`: 그리고자 하는 첫 정점의 index
* `count`: 그리려는 정점의 총 개수

```cpp
glDrawArrays(GL_TRIANGLES, 0, 6);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

#### `glDrawElements(primitive, count, type, pointer/offset)`

* 현재 바인딩된 VAO, VBO, EBO를 바탕으로 그리기
* `primitive`: 그려낼 기본 primitive 타입
* `count`: 그리고자 하는 EBO 내 index의 개수
* `type`: index의 데이터형
* `pointer/offset`: 그리고자 하는 EBO의 첫 데이터로부터의 오프셋



### 주의사항

위 함수들은 다음과 같은 순서로 진행이 되어야 한다.

* VAO binding
* **VBO binding**
* vertex attribute setting
* vertex attribute을 설정하기 전에 VBO가 바인딩 되어있을 것

VAO 세팅시, 해당 object가 사용할 buffer를 같이 세팅하기 위함이다.

