---
description: Material map
---

# 💨 Lighting map

재질을 구성하는 ambient, diffuse, specular를 각각 지정할 수도 있으나, 텍스처 맵으로 이를 대체하면 더 간편하게 사용이 가능하다. 우리가 흔히 사용하는 오브젝트의 texture map은 결국 diffuse map으로 사용하게 된다.

텍스처를 이용하기에, 먼저 리팩토링을 진행한다.

#### Program 클래스 변경

쉐이더 파일을 읽어서 한번에 초기화를 진행.

```cpp
static ProgramUPtr Create(
    const std::string& vertShaderFilename,
    const std::string& fragShaderFilename);
```

```cpp
ProgramUPtr Program::Create(const std::string& vertShaderFilename, const std::string& fragShaderFilename) {
    ShaderPtr vs = Shader::CreateFromFile(vertShaderFilename, GL_VERTEX_SHADER);
    ShaderPtr fs = Shader::CreateFromFile(fragShaderFilename, GL_FRAGMENT_SHADER);
    if (!vs || !fs)
        return nullptr;
    return std::move(Create({vs, fs}));
}
```

이후, Context의 init 부분에서 아래와 같이 멤버 변수를 추가한 뒤, simple.vs와 simple.fs를 한번에 초기화한다.

```cpp
ProgramUPtr m_simpleProgram;
```

```cpp
m_material.diffuse = Texture::CreateFromImage(Image::Load("./image/container2.png").get());
m_material.specular = Texture::CreateFromImage(Image::Load("./image/container2_specular.png").get());
```

#### 쉐이더 변경

이후, simple.vs와 simple.fs에 transform matrix uniform을 추가한다.

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

uniform mat4 transform;

void main() {
    gl_Position = transform * vec4(aPos, 1.0);
}
```

```glsl
#version 330 core
uniform vec4 color;
out vec4 FragColor;

void main() {
    FragColor = color;
}
```

이후, lighting.fs는 이와같이 수정한다. Meterial의 defuse와 specular를 texture로 받아서 처리한다. 이때, Meterial의 ambient color와 diffuse color는 통상적으로 diffuse color하나로 동일하게 쓴다. texture에서 Meterial 정보를 받아와서 사용하는 방식이다.

```glsl
struct Material {
    // diffuse map을 사용한다.
    // material의 diffuse color와 ambient color는 통상적으로 동일한 형태로 많이 쓴다.
    sampler2D diffuse;
    sampler2D specular;
    // 표면의 반짝거림을 표현하기에 meterial로 들어간다.
    float shininess;
};

void main() {
    vec3 texColor = texture(material.diffuse, texCoord).xyz;
    vec3 ambient = texColor * light.ambient;
    
    vec3 diffuse = diff * texColor * light.diffuse;

    vec3 specColor = texture(material.specular, texCoord).xyz;
    vec3 specular = spec * specColor * light.specular;
}
```

#### Context 수정

simple.vs, simple.fs에 uniform 값을 전달하여, 기본 큐브를 생성하는 방식으로 변경한다.

```cpp
m_simpleProgram->Use();
m_simpleProgram->SetUniform("color", glm::vec4(m_light.ambient + m_light.diffuse, 1.0f));
m_simpleProgram->SetUniform("transform", projection * view * lightModelTransform);
glDrawElements(GL_TRIANGLES, 36, GL_UNSIGNED_INT, 0);
```

이후 context의 Meterial 구조체를 수정한다.

```cpp
struct Material {
    TextureUPtr diffuse;
    TextureUPtr specular;
    float shininess { 32.0f };
};
```

texture를 전달하기 위해, 각각 0번, 1번 슬롯에 해당 텍스쳐를 활성화 시킨다음, 바인딩하여 쉐이더에 전달될 수 있게끔 한다.

```cpp
m_program->SetUniform("light.specular", 0);
m_program->SetUniform("material.specular", 1);
m_program->SetUniform("material.shininess", m_material.shininess);

glActiveTexture(GL_TEXTURE0);
m_material.diffuse->Bind();
glActiveTexture(GL_TEXTURE1);
m_material.specular->Bind();
```
