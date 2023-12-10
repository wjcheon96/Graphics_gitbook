---
description: Lighting
---

# 💡 조명

물체의 색상을 결정한다 라고 함은, 광원과 재질, 이 외에도 매우 복잡한 물리적 현상들이 겹쳐서 일어난다.



### Illumination Model <a href="#illumination-model" id="illumination-model"></a>

빛이 어떤식으로 눈에 들어오는지를 결정하는가 를 수식으로 모델링 하는 것.

* 반사광: 광원이 물체에 반사되어 발생한 빛
* Local illumination model: 반사광을 고려한 모델
* Global illumination model: 반사광을 고려하여 모델링. 더 사실적으로 보이나, 비용이 그만큼 많이 든다. Ray tracing으로 보완하는 방법이 있다.

해당 파트에서는, local illumination model을 기준으로 진행한다.



### Phong's illumination model

가장 일반적인 local illumination model로, 빛과 물체 간의 색상 결정을 3가지로 나눠서 표현한다. 각각, ambient(주변광), diffuse(분산광), specular(반사광)으로 불리며, 위 세개의 빛을 더해서 최종 색상을 결정하는 방식을 쓴다.

#### Ambient(주변광)

빛의 방향, 물체 표면의 방향, 시선 방향과 아무 상관 없이 물체가 기본적으로 받는 빛을 나타내며, 상수값으로 표현한다.

#### Defuse(분산광)

빛이 물체 표면에 부딛혔을때, 모든 방향으로 고르게 퍼지는 빛을 말하며, 시선의 방향과는 상관 없이 빛의 방향과 물체 표면의 방향에 따라 결정된다.

`diffuse = dot(light, normal)`

<figure><img src="../.gitbook/assets/스크린샷 2023-09-13 11.48.45.png" alt="" width="182"><figcaption></figcaption></figure>

#### Specular(반사광)

빛이 물체 표면에 부딪혀 반사되는 광원으로, 시선의 방향이 이 반사광의 방향과 동일할 때 가장 강한 빛이 나온다.

* `reflect = 2 * dot(light, normal) - light`
* `specular = dot(view, reflect)`



<figure><img src="../.gitbook/assets/스크린샷 2023-09-13 11.50.56.png" alt="" width="185"><figcaption></figcaption></figure>



### 주변광 구현

주변광(Ambient light) 구현을 위해 쉐이더를 만들어준다.

#### lighting.vs

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
layout (location = 2) in vec2 aTexCoord;

uniform mat4 transform;

out vec3 normal;
out vec2 texCoord;

void main() {
  gl_Position = transform * vec4(aPos, 1.0);
  normal = aNormal;
  texCoord = aTexCoord;
}
```

#### lighting.fs

lightingColor는 광원의 색상을, objectColor는 물체의 색상을 나타내며, lightingColor와 ambientStrength(밝기 조절 계수)를 곱함으로서 실제 광원의 색상을 만들어낸다.

```glsl
#version 330 core
in vec3 normal;
in vec2 texCoord;
out vec4 fragColor;

uniform vec3 lightColor;
uniform vec3 objectColor;
uniform float ambientStrength;

void main() {
  vec3 ambient = ambientStrength * lightColor;
  vec3 result = ambient * objectColor;
  fragColor = vec4(result, 1.0);
}
```

해당 쉐이더를 읽어오도록 연결시킨 이후, 위의 uniform을 세팅할 수 있게끔 해당 함수를 program에 구현한다. 방식은 쉐이더에 전달만 시켜주며, 이전에 썼던 방식과 동일하다.

```cpp
void Program::SetUniform(const std::string& name, float value) const {
    auto loc = glGetUniformLocation(m_program, name.c_str());
    glUniform1f(loc, value);
}

void Program::SetUniform(const std::string& name, const glm::vec3& value) const {
    auto loc = glGetUniformLocation(m_program, name.c_str());
    glUniform3fv(loc, 1, glm::value_ptr(value));
}
```

해당 색상을 저장할 변수을 설정한다.

```cpp
glm::vec3 m_lightColor { glm::vec3(1.0f, 1.0f, 1.0f) };
glm::vec3 m_objectColor { glm::vec3(1.0f, 0.5f, 0.0f) }; 
float m_ambientStrength { 0.1f };
```

명암을 정해진 수치로만 설정한 값이며, ImGui를 통해서 이를 조절 할 수있다.

```cpp
if (ImGui::CollapsingHeader("light")) {
    ImGui::ColorEdit3("light color", glm::value_ptr(m_lightColor));
    ImGui::ColorEdit3("object color", glm::value_ptr(m_objectColor));
    ImGui::SliderFloat("ambient strength", &m_ambientStrength, 0.0f, 1.0f);
}
```

이후 uniform 값을 Render에서 세팅해주면 된다.

```cpp
m_program->SetUniform("lightColor", m_lightColor);
m_program->SetUniform("objectColor", m_objectColor);
m_program->SetUniform("ambientStrength", m_ambientStrength);
```



### 분산광 구현

분산광(defuse light)의 밝기는 빛의 방향과 밝기, 표현의 normal 방향이 필요하다.

#### lighting.vs / lighting.fs 수정

lighting.vs에 uniform matrix를 추가한다. modelTransform은 local coordinate를 world coordinate으로 바꾸는데 사용된다.

```glsl
uniform mat4 modelTransform;
```

normal 벡터가 점이 아닌 벡터이기에, inverse transpose를 해야한다. 모든 점에서 동일하게 작용시키기 위함.

현재는 쉐이더 프로그램 내부에서 실행시켰기에, modelTranform uniform이 매번 계산되나, 일반적으로 cpu에서 계산한 값을 넘겨줘서 연산 횟수를 줄임.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-14 at 2.30.21 PM.png" alt="" width="375"><figcaption></figcaption></figure>

```glsl
normal = (transpose(inverse(modelTransform)) * vec4(aNormal, 0.0)).xyz;
```

이후, position을 fragment shader에 넘겨준다.

```glsl
position = (modelTransform * vec4(aPos, 1.0)).xyz;
```

position을 받아와서, lightDir을 계산하는데, 이때 빛의 방향 = 광원 위치 - 각 픽셀의 3D위치를 나타내며, normal vector가 보간되는 과정(vertax shader에서 계산된 normal이 rasterization되는 과정)에서 크기가 1이 아닐 수 있기에 이를 보간하는 과정을 가진다.

이후 pixelNorm과 lightDir을 내적(투영)하고, 빛의 색상을 곱해 최종적인 결과값을 만든다.

```glsl
vec3 lightDir = normalize(lightPos - position);
// 새로 입력된 vertex 의 normal vector
vec3 pixelNorm = normalize(normal);
// 빛의 크기를 구하되, pixelNorm과 lightDir의 내적 값을 계산하는데, 이 수치가 음수일때는 0으로 둠.
vec3 diffuse = max(dot(pixelNorm, lightDir), 0.0) * lightColor;
// 최종적으로 분산광과 반사광의 합을 통해 실제 색상을 결정한다.
vec3 result = (ambient + diffuse) * objectColor;
```

아래와 같이 설정을 해서, element를 그리고, 아래에 다시 세팅을 함으로서 imGui에서 세팅한 값을 가지고 그림을 그리게끔 한다.

```cpp
m_program->Use();
m_program->SetUniform("lightPos", m_lightPos);
m_program->SetUniform("lightColor", glm::vec3(1.0f, 1.0f, 1.0f));
m_program->SetUniform("objectColor", glm::vec3(1.0f, 1.0f, 1.0f));
m_program->SetUniform("ambientStrength", 1.0f);
m_program->SetUniform("transform", projection * view * lightModelTransform);
m_program->SetUniform("modelTransform", lightModelTransform);
glDrawElements(GL_TRIANGLES, 36, GL_UNSIGNED_INT, 0);

m_program->Use();
m_program->SetUniform("lightPos", m_lightPos);
m_program->SetUniform("lightColor", m_lightColor);
m_program->SetUniform("objectColor", m_objectColor);
m_program->SetUniform("ambientStrength", m_ambientStrength);
```



### 반사광(Specular light)

반사광의 밝기를 결정하는 요소는 빛의 방향과 밝기, 표면의 normal 방향, 시선의 방향이 있다.

#### lighting.fs 수정

lighting.vs를 수정하지 않는 이유는, 이미 분산광을 계산하면서 vertex에서 필요한 계산이 다 이뤄졌기 때문이다.

```glsl
uniform float specularStrength;
uniform float specularShininess;
uniform vec3 viewPos;
```

specularStrength로 반사광의 정도를 조절하며, specularShininess로 반사광의 면적을 조절한다.

View dir은 light dir을 구한것과 비슷하게, 카메라의 world space 상의 coordinate와 픽셀 coordinate 간의차로 구현한다.

reflectDir을 reflect 함수를 통해 계산하며, specular도 위의 diffuse(분산광)을 구한 방식과 비슷하게 구한다.

specular를 구할때, spec 변수가 제곱(pow)가 있는 까닭은 반사광의 면적을 구하기 위함이다.

```glsl
uniform float specularStrength;
uniform float specularShininess;

vec3 viewDir = normalize(viewPos - position);
vec3 reflectDir = reflect(-lightDir, pixelNorm);
float spec = pow(max(dot(viewDir, reflectDir), 0.0), specularShininess);
vec3 specular = specularStrength * spec * lightColor;

vec3 result = (ambient + diffuse + specular) * objectColor;
```

`reflect(light, normal)`: `light` 벡터 방향의 광선이 `normal` 벡터 방향의 표면에 부딪혔을 때 반사되는 벡터를 출력하는 내장함수.

이후 관련된 uniform을 세팅해준다.

```cpp
m_program->SetUniform("viewPos", m_cameraPos);
m_program->SetUniform("specularStrength", m_specularStrength);
m_program->SetUniform("specularShininess", m_specularShininess);
```

