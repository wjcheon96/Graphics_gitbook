---
description: Object Loader
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 🥥 오브젝트 로더

실제 물체를 렌더링하는 방법은 3D modeling tool을 이용해 3D 모델 디자인한 이후, 이를 3D 모델 파일 포멧(ex: FBX, Collada(dae), GLTF, OBJ, PLY 등)으로 저장한다. 이후 해당 파일을 읽어들이는 object loader를 구현한다.

이 중, 가장 범용적으로 쓰이는 Object file인 .obj파일을 기준으로 기술한다. obj파일과 mtl파일의 형태를 약식으로 표현하자면 아래와 같다.

v는 vertex를, vt는 texture coordinate, vn은 vertex normal이며, f에서 나타내고자 하는 면을 v, vt, vn에 따라서 점의 좌표를 지정해준다.

```
# Blender v2.80 (sub 44) OBJ File: ''
# www.blender.org
mtllib backpack.mtl
o Cube.028__0
v -0.556751 -0.023763 -0.141540
v -0.556645 0.003057 -0.140628
vt 0.283512 0.401314
vt 0.283512 0.394454
vn 0.0001 0.9989 0.0473
vn 0.0000 0.9989 0.0464
usemtl Scene_-_Root
s off
f 55/1/1 57/2/2 58/3/2
f 57/2/2 61/4/2 62/5/2
```



### assimp(open asset import library)

다양한 언어에서 사용 가능하며, 매우 다양한 3D 모델 파일을 지원해 3D파일 로딩을 해당 라이브러리를 통해 할 수 있다.

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-17 at 4.09.57 PM.png" alt=""><figcaption><p>Assimp 라이브러리의 클래스 구조</p></figcaption></figure>



### Scene Tree

3D 장면을 트리 형식으로 관리하는 방식. 대부분의 3D 모델링 툴들이 이러한 방식으로 장면을 관리한다.

* 부모/자식 관계로 되어있다.
* 자식의 transform 정보(position / rotation / scale)은 부모의 local coordinates를 기준으로 기술된다.



### Mesh 데이터를 로딩하고 그리는 법

기본 골자는 이전의 객체들과 비슷한데, 아래와 같이 model.hpp를 만든다.

```cpp
#include <assimp/Importer.hpp>
#include <assimp/scene.h>
#include <assimp/postprocess.h>

CLASS_PTR(Model);
class Model {
public:
    static ModelUPtr Load(const std::string& filename);

    int GetMeshCount() const { return (int)m_meshes.size(); }
    MeshPtr GetMesh(int index) const { return m_meshes[index]; }
    void Draw() const;

private:
    Model() {}
    bool LoadByAssimp(const std::string& filename);
    void ProcessMesh(aiMesh* mesh, const aiScene* scene);
    void ProcessNode(aiNode* node, const aiScene* scene);
        
    std::vector<MeshPtr> m_meshes;
};

#endif
```

Assimp::Importer클래스의 readFile 함수를 이용하는데, scene->mRootNode부터 재귀적으로 처리한다(ProcessNode 함수). 이는 scene tree상 이미지가 부모-자식 관계로 되어있기 때문이다.  ProcessNode에서 mesh가 있는지 확인하고, 있으면 해당 mesh를 처리한다.

```cpp
bool Model::LoadByAssimp(const std::string& filename) {
    Assimp::Importer importer;
    auto scene = importer.ReadFile(filename, aiProcess_Triangulate | aiProcess_FlipUVs);

    if (!scene || scene->mFlags & AI_SCENE_FLAGS_INCOMPLETE || !scene->mRootNode) {
        SPDLOG_ERROR("failed to load model: {}", filename);
        return false;
    }

    auto dirname = filename.substr(0, filename.find_last_of("/"));
    auto LoadTexture = [&](aiMaterial* material, aiTextureType type) -> TexturePtr {
        if (material->GetTextureCount(type) <= 0)
            return nullptr;
        aiString filepath;
        material->GetTexture(type, 0, &filepath);
        auto image = Image::Load(fmt::format("{}/{}", dirname, filepath.C_Str()));
        if (!image)
            return nullptr;
        return Texture::CreateFromImage(image.get());
    };

    for (uint32_t i = 0; i < scene->mNumMaterials; i++) {
        auto material = scene->mMaterials[i];
        auto glMaterial = Material::Create();
        glMaterial->diffuse = LoadTexture(material, aiTextureType_DIFFUSE);
        glMaterial->specular = LoadTexture(material, aiTextureType_SPECULAR);
        m_materials.push_back(std::move(glMaterial));
    }

    ProcessNode(scene->mRootNode, scene);
    return true;
}

void Model::ProcessNode(aiNode* node, const aiScene* scene) {
    for (uint32_t i = 0; i < node->mNumMeshes; i++) {
        auto meshIndex = node->mMeshes[i];
        auto mesh = scene->mMeshes[meshIndex];
        ProcessMesh(mesh, scene);
    }

    for (uint32_t i = 0; i < node->mNumChildren; i++) {
        ProcessNode(node->mChildren[i], scene);
    }
}
```

#### Ramda closer

내부의 `auto LoadTexture = [&](aiMaterila* material, aiTextureType type -> TexturePtr{}`부분을 보면, 람다 클로저가 적용된 걸 알 수 있다. `[capture, ...] = (param_type parameters, ...) -> return_type`의 형태로 이루어져 있는데,  파라미터 앞에 &가 붙으면 외부의 값을 capture해서 외부의 값들도 사용하게 할 수 있다.

이후, ProcessMesh에서 aiMesh, asScene이라는 assimp 내부의 클래스를 입력으로 받아, mesh의 data를 로딩한다. std::vector로 vertex array와 index array를 만들고, 각각의 vertex, index에 대한 정보를 mesh에 집어넣는다. 값을 집어넣을때, push\_back으로 동적할당을 진행할 수도 있으나, 크기가 정해져있기에 크기에 맞춰 집어넣음으로서 메모리 공간을 아낀다.

```cpp
void Model::ProcessMesh(aiMesh* mesh, const aiScene* scene) {
    SPDLOG_INFO("process mesh: {}, #vert: {}, #face: {}",
        mesh->mName.C_Str(), mesh->mNumVertices, mesh->mNumFaces);

    std::vector<Vertex> vertices;
    vertices.resize(mesh->mNumVertices);
    for (uint32_t i = 0; i < mesh->mNumVertices; i++) {
        auto& v = vertices[i];
        v.position = glm::vec3(mesh->mVertices[i].x, mesh->mVertices[i].y, mesh->mVertices[i].z);
        v.normal = glm::vec3(mesh->mNormals[i].x, mesh->mNormals[i].y, mesh->mNormals[i].z);
        v.texCoord = glm::vec2(mesh->mTextureCoords[0][i].x, mesh->mTextureCoords[0][i].y);
    }

    std::vector<uint32_t> indices;
    indices.resize(mesh->mNumFaces * 3);
    for (uint32_t i = 0; i < mesh->mNumFaces; i++) {
        indices[3 * i] = mesh->mFaces[i].mIndices[0];
        indices[3*i+1] = mesh->mFaces[i].mIndices[1];
        indices[3*i+2] = mesh->mFaces[i].mIndices[2];
    }

    auto glMesh = Mesh::Create(vertices, indices, GL_TRIANGLES);
    if (mesh->mMaterialIndex >= 0)
        glMesh->SetMaterial(m_materials[mesh->mMaterialIndex]);
    
    m_meshes.push_back(std::move(glMesh));
}
```

Mesh 에서 material 객체를 생성해서, 이를 mesh에서 받아서 쓴다.

```cpp
CLASS_PTR(Material);
class Material {
    public:
        static MaterialUPtr Create() {
            return MaterialUPtr(new Material());
        }
        TexturePtr diffuse;
        TexturePtr specular;
        float shininess { 32.0f };

        void SetToProgram(const Program* program) const;

    private:
        Material() {}
};
```

내부적으로, 아래와 같은 방식으로 program을 세팅한다. diffuse와 specular가 있을 시 해당 경우에 맞춰 uniform을 세팅한다.

```cpp
void Material::SetToProgram(const Program* program) const {
    int textureCount = 0;
    // material의 diffuse가 있으면, 0번 슬롯에 바인딩을 시킨다.
    if (diffuse) {
        glActiveTexture(GL_TEXTURE0 + textureCount);
        program->SetUniform("material.diffuse", textureCount);
        diffuse->Bind();
        textureCount++;
    }
    // specula가 있을시, 동일하게 진행.
    if (specular) {
        glActiveTexture(GL_TEXTURE0 + textureCount);
        program->SetUniform("material.specular", textureCount);
        specular->Bind();
        textureCount++;
    }
    // texture 다시 0번으로 초기화시켜서, shininess 추가.
    glActiveTexture(GL_TEXTURE0);
    program->SetUniform("material.shininess", shininess);
}
```

Mesh의 구현체는 다음과 같다. vertex, index에 대한 부분을 세팅하고, glDrawElements를 호출해 그림을 그리는 역할을 한다.

<pre class="language-cpp"><code class="lang-cpp"><strong>void Mesh::Init(const std::vector&#x3C;Vertex>&#x26; vertices, const std::vector&#x3C;uint32_t>&#x26; indices, uint32_t primitiveType) {
</strong>    m_vertexLayout = VertexLayout::Create();
    m_vertexBuffer = Buffer::CreateWithData(GL_ARRAY_BUFFER, GL_STATIC_DRAW,
        vertices.data(), sizeof(Vertex), vertices.size());
    m_indexBuffer = Buffer::CreateWithData(GL_ELEMENT_ARRAY_BUFFER, GL_STATIC_DRAW,
        indices.data(), sizeof(uint32_t), indices.size());
    // position
    m_vertexLayout->SetAttrib(0, 3, GL_FLOAT, false,
        sizeof(Vertex), 0);
    // normal
    m_vertexLayout->SetAttrib(1, 3, GL_FLOAT, false,
        sizeof(Vertex), offsetof(Vertex, normal));
    // texcoordinate
    // offsetof operator는 structure안의 인자로 들어온 member가 얼마만큼 건너 뛰었는지를 계산해준다.
    m_vertexLayout->SetAttrib(2, 2, GL_FLOAT, false,
        sizeof(Vertex), offsetof(Vertex, texCoord));
}

void Mesh::Draw(const Program* program) const {
    m_vertexLayout->Bind();
    if (m_material) {
        m_material->SetToProgram(program);
    }
    glDrawElements(m_primitiveType, m_indexBuffer->GetCount(), GL_UNSIGNED_INT, 0);
}
</code></pre>



이후, context에서 model을 draw하는 방식으로 변경한다.

```cpp
m_model->Draw(m_program.get());
```
