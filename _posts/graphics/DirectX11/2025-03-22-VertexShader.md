---
layout: single
title:  "[DirectX11] Vertex Shader"
categories: 
    - Graphics
tag: [DirectX11]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2025-03-22
last_modified_at: 2025-03-23

---
이 글은 인프런 강의 **[게임 프로그래머 도약반] DirectX11 입문**를 보고 따라 만들면서 헷갈리는 부분을 복습하며 정리한 글입니다.
{: .notice--warning}

# Vertex Shader
`VertexShader단계`는 `IA단계에서 넘겨 받은 데이터`를 통해서 `GPU에서 조작하는 과정`이다.

이떄 넘겨받은 `모든 정점을 대상`으로 `정점별 연산`을 수행한다. <br>
보통 이 단계에서 월드 변환 -> View 변환 -> Projection 변환 행렬들을 차례차례 곱해서 <br>
변환된 좌표를 return한다.

## 사용법
### 파이프 라인에 끼워넣기
vertexShader는 ID3D11VertexShader라는 클래스로 저장되며<br>
VS 단계에서 조작하려면 `.hlsl` 확장자 같은 파일에 따로 `셰이더 코드`를 작성해야한다.<br>
이때 이 셰이더 코드가 담긴 파일을 GPU에게 알려주기 위해서 `ID3DBlob` 이라는 형태로 저장하는데<br>
이는 [InputLayout](https://novicehog.github.io/graphics/IA/#inputlayout)단계에서 이루어진다.


`VertexShader`를 생성하는 코드는 다음과 같다.
```cpp
// 파일명, entryPoint 함수 이름 , 셰이더 버전, 저장할 ID3DBlob변수
LoadShaderFromFile(L"Default.hlsl", "VS", "vs_5_0", _vsBlob);
HRESULT hr = _device->CreateVertexShader(_vsBlob->GetBufferPointer(),_vsBlob->GetBufferSize(), nullptr, _vertexShader.GetAddressOf());
```


`VectexShader`도 `DeviceContext`를 통해 파이프라인에 등록해주면 사용할 준비가 끝난다.
```cpp
// VS
_deviceContext->VSSetShader(_vertexShader.Get(), nullptr, 0);
```


### 셰이더 파일에서의 사용
셰이더 파일 내에서 사용할 때 주의해야할 점은 InputLayout에서 지정한 형태와 맞춰야한다.<br>
InputLayout을 통해 VertexBuffer에 POSITION과 COLOR의 데이터가 들어가 있다고 묘사했다면

```cpp
D3D11_INPUT_ELEMENT_DESC layout[] =
{
    {"POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_PER_VERTEX_DATA,0},
    {"COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D11_INPUT_PER_VERTEX_DATA,0},
};
```

<br>


다음과 같이 셰이더 코드에서는 `POSITION`과 `COLOR`를 받아주면 된다.
이 `POSITION과 COLOR를 제대로 맞춰`줘야 `VertexBuffer의 데이터를 제대로 가져올 수 있다.`

```hlsl
struct VS_INPUT
{
    float4 position : POSITION;  
    float4 color : COLOR;
};
```


이 INPUT 데이터를 통해 VS에서 가공해서 내보낼 OUTPUT데이터도 struct로 정의해주면 된다.
OUTPUT 데이터에서 중요한 점은 데이터 정의는 자유롭지만 SV_POSITION의 변수를 무조건 선언해주어야 한다.

```hlsl
struct VS_OUTPUT
{
    float4 position : SV_POSITION;  // 반드시 필요함
    float4 color : COLOR;
};
```

<br>

모든 준비가 끝났으면 VS함수를 조작하면 된다.<br>
지금은 따로 할수있는게 없어서 그대로 넘겨주지만 <br>
나중에 Constant Buffer(상수 버퍼)를 이용해서 여러 변환행렬을 넘겨주어 이곳에서 계산이 가능하다.


```hlsl
VS_OUTPUT VS(VS_INPUT input)
{
    VS_OUTPUT output;
    output.position = input.position;
    output.color = input.color;
    
    return output;
}
```


### 결과
다른 파이프라인 과정도 추가로 필요하지만 VS가 영향을 준 부분만 보자면 다음과 같다.<br>
먼저 다음과 같이 VertexBuffer를 넘겨주고

```cpp
// Vec3와 Color는 using을 통해 이름만 바꾼 DirectX::XMFLOAT3와 DirectX::XMFLOAT4임
_vertices[0].position = Vec3(-0.5f, -0.5f, 0);
_vertices[0].color = Color(1.0f, 0.f, 0.f, 1.f);

_vertices[1].position = Vec3(0, 0.5f, 0);
_vertices[1].color = Color(1.0f, 0.f, 0.f, 1.f);

_vertices[2].position = Vec3(0.5f, -0.5f, 0);
_vertices[2].color = Color(1.0f, 0.f, 0.f, 1.f);
```
<br>

PS단계에서는 color만 바로 반환한다면

```hlsl

float4 PS(VS_OUTPUT input) : SV_Target
{
    return input.color;
}

```

아래와 같은 삼각형이 결과로 출력된것을 볼 수 있다.

![삼각형](https://github.com/user-attachments/assets/b6f49a9f-8a04-466c-b99a-45e73b974ab1)

VS코드를 다음과 같이 조금 수정해서 position의 값을 수정하면

```hlsl
VS_OUTPUT VS(VS_INPUT input)
{
    VS_OUTPUT output;
    output.position = input.position + float4(0.3f, 0.3f, 0.0f, 0.0f); // 포지션값 변경
    output.color = input.color;
    
    return output;
}
```

삼각형이 이동한것을 볼 수 있다.

![삼각형 이동](https://github.com/user-attachments/assets/d36d6aa2-c641-4787-bbec-85b4d31f07e9)



## 결론
VertexShader는 VertexBuffer로 받은 VertexData들을 프로그래머가 조작할 수 있는곳이며, <br>
`모든 정점`을 대상으로 `함수가 실행`된다.

