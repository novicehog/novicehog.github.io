---
layout: single
title:  "[DirectX11] Pixel Shader"
categories: 
    - Graphics
tag: [DirectX11]
published: true

author_profile: true #

search: true 

date: 2025-04-01
last_modified_at: 2025-04-01

---
이 글은 인프런 강의 **[게임 프로그래머 도약반] DirectX11 입문**를 보고 따라 만들면서 헷갈리는 부분을 복습하며 정리한 글입니다.
{: .notice--warning}

# Pixel Shader
Pixel Shader는 `Rasterizer 단계`에서 생성된 `픽셀들을 처리`하는 단계이다.<br>
`Rasterzier단계`는 간단하게 말해서 `VS에서 받은 정점 데이터를 픽셀로 변환`하는 과정이다.

PS단계를 통해 `최종 화면에 출력될 색상을 결정`한다.


## 사용법
### 파이프 라인에 끼워넣기
pixel Shader도 vertex Shader와 유사하게 `ID3DBlob`이라는 형태로 저장하여 사용한다.
```cpp
// 파일명, entryPoint 함수 이름 , 셰이더 버전, 저장할 ID3DBlob변수
LoadShaderFromFile(L"Default.hlsl", "PS", "ps_5_0", _psBlob);
HRESULT hr = _device->CreatePixelShader(_psBlob->GetBufferPointer(), _psBlob->GetBufferSize(), nullptr, _pixelShader.GetAddressOf());
CHECK(hr);
```

Device Context를 통해 파이프라인에 등록하여 사용한다.
```cpp
// PS
_deviceContext->PSSetShader(_pixelShader.Get(), nullptr, 0);
```

### 셰이더 파일에서의 사용
Rasterizer단계에서 생성된 모든 픽셀들은 Vertex와 관련하여 자동으로 보간된 값이 들어온다.<br>

VertexData를 다음과 같이 하여 각각 빨,초,파의 정점을 가진 삼각형을 만들어주고
```cpp
// VertexData
{
    _vertices.resize(3);

    _vertices[0].position = Vec3(-0.5f, -0.5f, 0);
    _vertices[0].color = Color(1.0f, 0.f, 0.f, 1.f); // 빨간색

    _vertices[1].position = Vec3(0, 0.5f, 0);
    _vertices[1].color = Color(0.f, 1.0f, 0.f, 1.f); // 초록색

    _vertices[2].position = Vec3(0.5f, -0.5f, 0);
    _vertices[2].color = Color(0.f, 0.f, 1.f, 1.f);  // 파란색
}
```

PS 단계에서 그대로 반환해주면 

```hlsl
struct VS_INPUT
{
    float4 position : POSITION;  
    float4 color : COLOR;
};

struct VS_OUTPUT
{
    float4 position : SV_POSITION;
    float4 color : COLOR;
};


VS_OUTPUT VS(VS_INPUT input)
{
    VS_OUTPUT output;
    output.position = input.position;
    output.color = input.color;
    
    return output;
}

float4 PS(VS_OUTPUT input) : SV_Target
{
    return input.color; // 값을 그대로 리턴
}
```


![삼각형](https://github.com/user-attachments/assets/a9f98b75-6d62-493d-8b27-8948c017e1c7)

`픽셀이 알아서 보간된 값(color)을 가지는 것`을 볼 수 있다.

