---
description: Program Class Design
---

# 🪄 프로그램 클래스 디자인

### Program

쉐이더가 그래픽스 파이프라인의 vertex 스테이지와 pixel(fragment) 스테이지에 이 두개가 들어가서 하나의 pipeline을 구성하는데, 이 pipeline을 openGL에서 program이라고 부른다.

&#x20;

#### Program Class 설계

* vertex shader, fragment shader를 연결한 pipeline program
* 두개의 shader를 입력받아 program을 링크 시킨다.
* 싱크에 성공하면 openGL program object를 생성, 실패시 메모리 할당 해제를 시킨다.



#### program 객체 생성

기본적인 골자는 Shader 객체와 비슷하다. 아래와 같이 객체를 디자인 하여, Create 함수로만 관리한다.

```cpp
CLASS_PTR(Program)
class Program {
    public:
        static ProgramUPtr Create(const std::vector<ShaderPtr>& shaders);
        ~Program();
        uint32_t Get() const { return m_program; }    
    private:
        Program() {}
        bool Link(const std::vector<ShaderPtr>& shaders);
        uint32_t m_program { 0 };
};
```



### Program 관련 함수

```cpp
m_program = glCreateProgram();
for (auto& shader: shaders)
    glAttachShader(m_program, shader->Get());
glLinkProgram(m_program);
```

`glCreateShader()`와 같이, `glCreateProgram()`을 통해서 프로그램을 생성하고, id값을 반환한다.

이후, `glAttachShader()`를 통해 매개변수로 받아온 shader를 attach 시키며, `glLinkProgram()`을 통해 해당 프로그램과 링킹을 시도한다.

```cpp
glGetProgramiv(m_program, GL_LINK_STATUS, &success);
glGetProgramInfoLog(m_program, 1024, nullptr, infoLog);
```

`glGetProgramiv()`는 glGetShaderiv()와 비슷하게 success로 성공 여부를 받아와서, 링킹이 제대로 실행됐는지를 확인하고, `glGetProgramInfoLog()`함수를 통해 해당 프로그램의 링킹 로그 정보를 받아온다.

```cpp
glDeleteProgram(m_program);
```

`glDeleteProgram()`을 통해 프로그램의 object를 제거한다.

