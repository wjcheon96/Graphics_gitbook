---
description: meterial
---

# 🩸 재질

최종적인 색상은 빛 색상 x 재질의 색상으로 이루어진다. 그렇기에 재질 또한 중요한데, 재질을 ambient / diffuse / specular로 나누어 표현하게 된다.

* 금속 재질은 반사광이 강하다.
* 천 재질은 반사광이 약하다.

이를 표현하기 위해, 이전의 fragment shader를 구조체에 담아, 수정한다.

vertual Shader는 변경사항이 없으나, fragment shader에서 light와 meterial이 각각 ambient, diffuse, specular를 지니기에 변수가 너무 많아지게 되고, 이를 해결하기 위해 구조체를 활용해 light와 meterial에 ambient, diffuse, specular를 담는다. 각각 light와 meterial은 position과 shiness를 추가적으로 갖고있는데, 이는 실제 조명의 위치와 meterial별 반사광의 차이를 지정하기 때문이다.

```glsl
struct Light {
    vec3 position;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
uniform Light light;

struct Material {
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float shininess;
};
uniform Material material;
```

이후 main문을 아래와 같이 수정한다. 큰 변경사항은 없고, 구조체에 담아 light와 meterial을 조명을 담았던 방식과 동일하게 담는다. 위에서 언급했듯, 최종적인 색상은 조명 색상 x 재질의 색상이므로 각각 이를 곱해 계산을 진행한다.

```glsl
void main() {
    vec3 ambient = material.ambient * light.ambient;

    vec3 lightDir = normalize(light.position - position);
    vec3 pixelNorm = normalize(normal);
    float diff = max(dot(pixelNorm, lightDir), 0.0);
    vec3 diffuse = diff * material.diffuse * light.diffuse;

    vec3 viewDir = normalize(viewPos - position);
    vec3 reflectDir = reflect(-lightDir, pixelNorm);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = spec * material.specular * light.specular;

    vec3 result = ambient + diffuse + specular;
    fragColor = vec4(result, 1.0);
}
```

그에 맞춰 program의 uniform 값을 아래와 같이 세팅한다.

```cpp
m_program->Use();
m_program->SetUniform("light.position", m_light.position);
m_program->SetUniform("light.ambient", m_light.diffuse);
m_program->SetUniform("material.ambient", m_light.diffuse);
m_program->SetUniform("transform", projection * view * lightModelTransform);
m_program->SetUniform("modelTransform", lightModelTransform);
glDrawElements(GL_TRIANGLES, 36, GL_UNSIGNED_INT, 0);

m_program->Use();
m_program->SetUniform("viewPos", m_cameraPos);
m_program->SetUniform("light.position", m_light.position);
m_program->SetUniform("light.ambient", m_light.ambient);
m_program->SetUniform("light.diffuse", m_light.diffuse);
m_program->SetUniform("light.specular", m_light.specular);
m_program->SetUniform("material.ambient", m_material.ambient);
m_program->SetUniform("material.diffuse", m_material.diffuse);
m_program->SetUniform("material.specular", m_material.specular);
m_program->SetUniform("material.shininess", m_material.shininess);
```

