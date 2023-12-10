---
description: Shader Class Design
---

# 🕸 쉐이더 클래스 디자인

쉐이더 클래스를 디자인하기에 앞서, C++11에 도입된 스마트 포인터를 사용하여, 생길 수 있는 문제를 최소화 한다.

아래와 같이 메크로 함수를 이용하여, 스마트 포인터에 대한 메크로를 재정의한다.

```cpp
#define CLASS_PTR(className) \
class className; \
using className ## UPtr = std::unique_ptr<className>; \
using className ## Ptr = std::shared_ptr<className>; \
using className ## WPtr = std::weak_ptr<className>;
```

#### std::unique\_ptr<> : 해당 메모리 블록을 단독으로 소유

#### std::shared\_ptr<> : 해당 메모리 블록의 소유권을 공유

#### std::weak\_ptr<> : 해당 메모리 소유권은 없으나, 접근은 가능



#### LoadTextFile

쉐이더 파일을 읽어올 함수. optional을 통해 값이 있거나 없는 경우를 포인터 없이 표현.

```cpp
std::optional<std::string> LoadTextFile(const std::string& filename) {
    std::ifstream fin(filename);
    if (!fin.is_open()) {
        SPDLOG_ERROR("failed to open file: {}", filename);
        return {};
    }
    std::stringstream text;
    text << fin.rdbuf();
    return text.str();
}
```



#### Shader 객체 생성

아래와 같이 shader 객체를 생성하여, createFormFile만으로 쉐이더를 관리.

```cpp
class Shader {
    public:
        static ShaderUPtr CreateFromFIle(const std::string& filename, GLenum shaderType);

        ~Shader();
        // shader 오브젝트의 생성 관리는 Shader 내부에서만 관리.
        uint32_t Get() const { return m_shader; };
    private:
        // CreateFormFile() 함수 외의 방식으로 Shader 인스턴스의 생성을 막는다.
        Shader() {}
        // 생성에 실패할 경우 false를 리턴하기 위해 bool타입으로 리턴.
        bool LoadFile(const std::string& filename, GLenum shaderType);
        uint32_t m_shader {0};
};
```



#### Shader 파일 예제

#### vertical shade(simple.vs)

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

void main() {
  gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
}
```

#### Fragmental shade(simple.fs)

```glsl
#version 330 core
out vec4 fragColor;

void main() {
  fragColor = vec4(1.0, 0.0, 0.0, 1.0);
}
```

자세한 glsl파일에 대한 기술은 뒤에서 하기로 한다.



### Shader 관련 함수

```cpp
m_shader = glCreateShader(shaderType);
glShaderSource(m_shader, 1, (const GLchar* const*)&codePtr, &codeLength);
glCompileShader(m_shader);
```

`glCreateShader()`를 통해서 shader 오브젝트를 생성한다.

`glShaderSource()`로  shader에 소스 코드설정. 2, 3, 4번째 매개변수를 통해 몇개의 코드를 집어넣을 지를 결정한다.

`glCompileShader()` 함수를 통해 shader를 컴파일한다.

```cpp
glGetShaderiv(m_shader, GL_COMPILE_STATUS, &success);
glGetShaderInfoLog(m_shader, 1024, nullptr, infoLog);
```

`glGetShaderiv()`는 쉐이더에 대한 정수형 정보를 얻어온다.&#x20;

`glGetShaderInfoLog()`는 쉐이더에 대한 로그를 얻어오며, 컴파일 에러를 얻어내는데 사용된다.

```cpp
glDeleteShader(m_shader);
```

마지막으로, `glDeleteShader()`는 쉐이더의 object를 없애준다.

