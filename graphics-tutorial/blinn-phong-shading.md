---
description: Blinn-Phong shading
---

# ⚡ Blinn-Phong shading

이전의 lighting에서, phong shading 방식을 사용해 빛을 조절하는 방식을 사용했는데, 이 방식을 쓸 때, specular shininess 값이 작을 경우, highlight 부분이 잘리는 듯한 현상이 생기는 문제점이 있다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 11.04.39.png" alt="" width="375"><figcaption></figcaption></figure>

해당 원인은, specular 계산시, view와 reflection간의 각도가 90도 보다 커지는 경우 dot product 값이 0보다 작아져 cutoff가 발생한다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 11.07.58.png" alt="" width="563"><figcaption></figcaption></figure>

### Blinn-Phong shading

이를 보완하기 위한 방식이 Blinn이 제안한 blinn-phong shading 방식이다.

해당 방식은, view와 light를 이등분 하는 벡터(halfway)와 normal vector간의 사잇각으로 계산하는 방식으로, view가 reflection과 일치할때는 halfway 벡터는 normal 벡터와 일치하게 된다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 12.02.38.png" alt="" width="375"><figcaption></figcaption></figure>

아래와 같이, 좀 더 cutoff 현상은 적어지나, highlight 부분이 좀 더 커지는 경향이 있어, material에 대해 값을 설정할때, shininess 값을 조정할 필요는 있다. 다만, phong shading보단 부작용이 적어 사용하게 된다.

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 12.04.18.png" alt="" width="563"><figcaption><p>shininess 값이 적은 경우</p></figcaption></figure>

<figure><img src="../.gitbook/assets/스크린샷 2023-09-20 12.06.54.png" alt="" width="563"><figcaption><p>shininess 값이 큰 경우</p></figcaption></figure>

#### Blinn-Phong 구현

상황에 따라 phong shading을 사용하는 경우도 있기에, 아래와 같이 조건을 나눠 phong shading을 사용 하는 경우와, blinn-phong shading을 사용하는 경우로 나눠서 처리한다.

Blinn-Phong 구조는 view와 light를 이등분하는 halfway 벡터를 사용하므로, 이를 구현해서 해당 케이스에 맞춰 halfDir 벡터를 사용한다.

```glsl
vec3 specColor = texture2D(material.specular, texCoord).xyz;
float spec = 0.0;
if (blinn == 0) {
  vec3 viewDir = normalize(viewPos - position);
  vec3 reflectDir = reflect(-lightDir, pixelNorm);
  spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
}
else {
  vec3 viewDir = normalize(viewPos - position);
  vec3 halfDir = normalize(lightDir + viewDir);
  spec = pow(max(dot(halfDir, pixelNorm), 0.0), material.shininess);
}
vec3 specular = spec * specColor * light.specular;
```







