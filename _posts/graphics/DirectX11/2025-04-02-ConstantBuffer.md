---
layout: single
title:  "[DirectX11] Constant Buffer"
categories: 
    - Graphics
tag: [DirectX11]
published: true

author_profile: true #

search: true 

date: 2025-04-02
last_modified_at: 2025-04-03

---
이 글은 인프런 강의 **[게임 프로그래머 도약반] DirectX11 입문**를 보고 따라 만들면서 헷갈리는 부분을 복습하며 정리한 글입니다.
{: .notice--warning}

# Constant Buffer 상수버퍼
Constant Buffer는 CPU에서 GPU로 데이터를 전달하기 위해 사용하는 도구이다.<br>
쉽게 말하자면 `외부(CPU)에서 값을 조정`하여 `셰이더(GPU) 파일 내`에서 `변수처럼 사용`할 수 있게 해주는 것이다.

예를 들어 Constant Buffer를 이용하여 `여러 변환 행렬을 넘김`으로써 셰이더 파일 안에서 조작할 수 있다.

## 코드에서 사용 (데이터 기입)
ConstantBuffer도 일단은 Buffer이기 때문에 `ID3D11Buffer`라는 형태로 선언한다.
### 선언
```cpp
ComPtr<ID3D11Buffer> _constantBuffer;
```

### 객체 생성
`buffer를 사용할 용도`를 `D3D11_BUFFER_DESC 구조체에 기입`하여 device를 통해 생성한다.

```cpp
void Game::CreateConstantBuffer()
{
	D3D11_BUFFER_DESC desc;
	ZeroMemory(&desc, sizeof(desc));
	desc.Usage = D3D11_USAGE_DYNAMIC; // map과 unmap을 통해 데이터 기입해야함
	desc.BindFlags = D3D11_BIND_CONSTANT_BUFFER; // Contant Buffer임을 명시
	desc.ByteWidth = sizeof(TransformData);
	desc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;

	HRESULT hr = _device->CreateBuffer(&desc, nullptr, _constantBuffer.GetAddressOf());
	CHECK(hr);
}
```


### 데이터 기입
먼저 구조체로 `어떤 형태의 데이터를 넘겨줄지 정의`한다. <br>
여기서는 간단하게 DirectX::XMFLOAT3타입의 offset만 정의했다.

```cpp
struct ConstantBufferData
{
	Vec3 offset; // DirectX::XMFLOAT3
	float dummy; // 버퍼를 사이즈를 맞춰주기 위한 더미 데이터
};
```


다음 처럼 매 프레임 0.003씩 값이 증가하게 하고 `Map, UnMap을 통해 Buffer에 값을 채워넣었다.`<br>

```cpp
void Game::Update()
{
	// 매 프레임 offset값을 조금씩 늘려줌
	_constantBufferData.offset.x += 0.001f;
	_constantBufferData.offset.y += 0.001f;

	D3D11_MAPPED_SUBRESOURCE subResource;
	ZeroMemory(&subResource, sizeof(subResource));
	
	_deviceContext->Map(_constantBuffer.Get(), 0, D3D11_MAP_WRITE_DISCARD, 0, &subResource);
	::memcpy(subResource.pData, &_constantBufferData, sizeof(_constantBufferData));
	_deviceContext->Unmap(_constantBuffer.Get(), 0);
}
```


### 파이프 라인에 끼워넣기
`VSSetConstantBuffers함수`를 이용해서 파이프라인에 상수 버퍼 값을 끼워넣으면 된다.<br>
여기서 중요한 것은 `0번째 슬롯`에 넣어줬단 것을 기억해야한다.
```cpp
_deviceContext->VSSetConstantBuffers(0, 1, _constantBuffer.GetAddressOf());
```


## 셰이더 파일에서의 사용
셰이더 파일에서는 다음과 같이 ConstantBuffer를 변수로 선언하여 사용한다.
```hlsl
cbuffer myConstData : register(b0) // 0번 슬롯
{
	float4 offset;
}
```

`VertexShader` 영역에서 단순히 `ConstantBuffer의 offset값을 더해주고`,
`PixelShader` 영역에서는 색을 그대로 출력하여 출력하면

```hlsl
VS_OUTPUT VS(VS_INPUT input)
{
	VS_OUTPUT output;
	output.position = input.position + offset;
	output.color = input.color;

	return output;
}

float4 PS(VS_OUTPUT input) : SV_Target
{
    float4 color = input.color;
	return color;
}
```


다음과 같이 constBuffer로 넘겨준 offset이 매 프레임 증가함에 따라<br>
VS 영역에서 output.position의 값도 점점 증가하여 이동하는 것을 볼 수 있다.
![결과](https://github.com/user-attachments/assets/5068dc08-1d73-4700-91b8-1069eb80be54)

![렌더링결과](https://github.com/user-attachments/assets/67f606b2-8856-4fc3-867c-fd6d49ee3b01)

# 결론
- `Constant Buffer(상수 버퍼)`를 이용하여 `GPU밖(CPU)에서 셰이더 파일 내에서 사용할 값을 넘겨줄 수 있다.`
