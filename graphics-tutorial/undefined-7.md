---
description: light caster
---

# 🥑 라이트 캐스터

물체에 빛을 쐬는 광원의 종류를 다양화 하는데, 흔히 사용되는 light caster는 3가지 종류가 있다.

* Directional light
* Point light(omni light0
* Spot light



### Directional Light

광원이 너무 멀리 떨어져서 모든 지점에 동일한 방향의 광선이 평행하게 발사되는 광원(ex. 태양광). Directional light는 방향밖에 없기에, 방향만 지정해주고 모든 곳에 동일한 크기의 빛을 적용시킨다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-15 at 3.46.14 PM.png" alt="" width="375"><figcaption><p>directional light</p></figcaption></figure>

position을 direction으로 변경 후, lightDir를 direction의 음수로 normalize를 진행한다.

<pre class="language-glsl"><code class="lang-glsl"><a data-footnote-ref href="#user-content-fn-1">struct</a> Light {
  vec3 direction;
  vec3 ambient;
  vec3 diffuse;
  vec3 specular;
};

vec3 lightDir = normalize(-light.direction);
</code></pre>

이후 아래와 같이 uniform setting을 하면 끝이다.

```cpp
m_program->SetUniform("light.direction", m_light.direction);
```

### Point light

한 점에서부터 모든 방향으로 광원이 발사된다. Omni direction이라고도 하며, 이전 과정에서 진행했던 light를 통틀어서 Point light로 볼 수 있다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-15 at 3.54.49 PM.png" alt="" width="375"><figcaption><p>point light</p></figcaption></figure>

#### Attenuation(감쇠)

물리 법칙상 거리의 제곱에 반비레하게 빛의 감쇠가 일어나나, local illumination model(광원이 물체에 직접 닿는 빛만 적용되며, 물체에 반사되서 나오는 빛에 대한 고려가 없음), 즉 주변의 반사광을 고려하지 않는 경우 거리의 제곱에 직접 반비례하게 만들면 급격히 어두워진다.&#x20;

이에 따라, Attenuation model을 사용하게 된다. 수식은 아래와 같다.

## $$F_{att} = \frac{1.0}{K_e+K_1*d+K_q*d^2}$$

* d: 광원과 물체 표면 간의 거리
* Kc, Kl, Kq 세 개의 파라미터(c는 상수(constant), l은 1차원 선형(linear), q는 2차원(quadratic)을 나타낸다)
* 광원이 어느정도 거리까지 영향을 주게할 것인지에 따라 파라미터를 조절

Ogre3D 엔진의 경우, 아래와 같은 광원 최대 거리에 따른 파라미터를 갖는다. Distance 사이의 값은 선형 보간법을 통해 구하게 된다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-15 at 4.09.02 PM.png" alt="" width="563"><figcaption></figcaption></figure>

이를 대략적으로 보간한 함수는 다음과 같다.

```cpp
glm::vec3 GetAttenuationCoeff(float distance) {
    const auto linear_coeff = glm::vec4(
        8.4523112e-05, 4.4712582e+00, -1.8516388e+00, 3.3955811e+01
    );
    const auto quad_coeff = glm::vec4(
        -7.6103583e-04, 9.0120201e+00, -1.1618500e+01, 1.0000464e+02
    );

    float kc = 1.0f;
    float d = 1.0f / distance;
    auto dvec = glm::vec4(1.0f, d, d*d, d*d*d);
    float kl = glm::dot(linear_coeff, dvec);
    float kq = glm::dot(quad_coeff, dvec);
    
    return glm::vec3(kc, glm::max(kl, 0.0f), glm::max(kq*kq, 0.0f));
}
```

이후 fragment shader를 아래와 같이 수정한다. 위의 수식을 코드로 나타낸 것과 동일하다.

```glsl
float dist = length(light.position - position);
vec3 distPoly = vec3(1.0, dist, dist * dist);
float attenuation = 1.0 / dot(distPoly, light.attenuation);
vec3 lightDir = (light.position - position) / dist;

vec3 result = (ambient + diffuse + specular) * attenuation;
```

이후 해당 program에 세팅한다.

```cpp
m_program->SetUniform("light.position", m_light.position);
m_program->SetUniform("light.attenuation", GetAttenuationCoeff(m_light.distance));
```



### Spot Light

스포트라이트는 광원의 위치와 방향을 모두 가지며, 광원 방향으로부터 일정 각도만큼만 광선을 발사한다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-15 at 5.58.12 PM.png" alt="" width="375"><figcaption><p>spot light</p></figcaption></figure>

&#x20;Fragment shader의 light 구조체에 direction과 cutoff를 추가.

```glsl
struct Light {
    vec3 position;
    vec3 direction;
    float cutoff;
    vec3 attenuation;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
```

이후, main문을 아래와 같이 수정한다. theta를 lightDir과 light의 direction을 정규화한 값의 내적을 했기에, cos값으로 나오게 된다. 따라서, 이보다 cutoff값이 클 때를 구해야 theta값이 작을 때로 볼 수 있다.

```glsl
float theta = dot(lightDir, normalize(-light.direction));
vec3 result = ambient;

// theta값은 광원의 방향과 lightDir과 실제 광원의 direction을 내적한 값, 즉 cos 값을 계산한다.
// 해당 cos값이 지정한 cos 값보다 크단 말은 지정한 각도보다 작다는 말이며, 이 경우에만 spotlight에 대한 조명을 계산한다.
if (theta > light.cutoff) {
    vec3 pixelNorm = normalize(normal);
    // 빛의 크기를 구하되, pixelNorm과 lightDir의 내적 값을 계산하는데, 이 수치가 음수일때는 0으로 둠.
    // 이후 material과 light의 diffuse와 곱해서 값을 구함.
    float diff = max(dot(pixelNorm, lightDir), 0.0);
    vec3 diffuse = diff * texColor * light.diffuse;

    vec3 specColor = texture(material.specular, texCoord).xyz;
    // light direction을 구한 방식과 동일.
    // 현재 카메라의 world space 상의 좌표와 픽셀 좌표 간의 차를 통해 시선 벡터 viewDir 계산.
    vec3 viewDir = normalize(viewPos - position);
    // light 벡터 방향의 광선이 normal 벡터 방향의 표면에 부딪혔을 때 반사되는 벡터를 출력하는 내장함수인 reflect 함수를 이용.
    vec3 reflectDir = reflect(-lightDir, pixelNorm);
    // reflectDir과 viewDir 간의 내적을 통해 반사광을 많이 보는 정도를 계산
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = spec * specColor * light.specular;

    result += diffuse + specular;
}
```

&#x20;허나, 이렇게 하면 경계선이 너무 명확하게 그려지기에, 아래와 같은 방법으로 경계선상을 부드럽게 한다.

* 광원의 방향을 기준으로 바깥쪽 각도 / 안쪽 각도를 정한다
* 안쪽 각도 내의 빛은 조명을 100% 적용한다.
* 안쪽에서 바깥쪽 각도 사이의 빛은 조명을 100%에서 0% 사이에 오게끔 조절한다.

## $$I =\frac{cos(x) - cos(outer)}{ cos(inner) - cos(outer)}$$

cutoff를 float에서 vec2로 변경한다.

```glsl
vec2 cutoff;
```

glsl에서 아래와 같이 위 식을 적용시키고, 0과 1 사이의 값으로 지정한다. 이후 result에 intensity를 곱한다.

```glsl
float intensity = clamp((theta - light.cutoff[1]) / (light.cutoff[0] - light.cutoff[1]), 0.0, 1.0);
    
result += (diffuse + specular) * intensity;
```

그에 맞춰 cutoff를 수정한다.

```cpp
glm::vec2 cutoff { glm::vec2(20.0f, 5.0f) };
```

```cpp
m_program->SetUniform("light.cutoff", glm::vec2(cosf(glm::radians(m_light.cutoff[0])),
    cosf(glm::radians(m_light.cutoff[0] + m_light.cutoff[1]))));
```



### 복수의 광원 처리

아래의 방식으로 복수 광원 처리가 가능한데, 방식은 복수의 파라미터를 fragment shader에 전달시키는 방식이다.\
다만, 조명 수가 많아지는 만큼 그 과정에서 부하가 커진다는 문제점이 있다

```glsl
out vec4 FragColor;
void main() {
    // 결과 색상 초기화
    vec3 output = vec3(0.0);
    // directional light에 의한 결과 누적
    output += someFunctionToCalculateDirectionalLight();
    // 모든 point light 들에 대해 결과 누적
    for(int i = 0; i < nr_of_point_lights; i++)
      output += someFunctionToCalculatePointLight();
    // spot light에 대한 결과 누적
    output += someFunctionToCalculateSpotLight();
    FragColor = vec4(output, 1.0);
}
```



[^1]: 
