---
layout: single
title:  "DirectX11 Device, DeviceContext, SwapChain"
categories: 
    - Graphics
tag: [DirectX11]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2025-03-03
last_modified_at: 2025-03-03

---
이 글은 인프런 강의 **[게임 프로그래머 도약반] DirectX11 입문**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}

# Device 와 DeviceContext 그리고 SwapChain
## Device란
`Device`란 `GPU가 사용할 그래픽 리소스를 생성 및 관리`하는 클래스이다.<br>
즉, GPU에게 그래픽 처리를 맡기기 위해서 `그래픽 관련 데이터`들을 `device를 통해 가공`한다고 생각하면된다.<br>

## DeviceContext란
`DeviceContext`는 Device를 통해 생성된 리소스들을 직접 사용하는 클래스이다.<br>
그래서 우리는 Device로 그래픽 리소스를 가공하고 DeviceContext에게 이 리소스를 사용하라고 명령을 내려서<br>
어떠한 무언가를 그릴 수 있게된다.

## SwapChain이란
화면은 지속적으로 갱신되며 새로운 화면을 보여줘야한다. 이때 단 한개의 버퍼만 이용한다면 새로운 화면을 띄울 때<br>
기존 화면을 지우고 새로운 화면을 작성하다보니 깜빡임 현상이 존재한다.<br>
이를 해결하기 위해 DirectX11에서 화면에 보이는 이미지를 `Front Buffer`와 `Back Buffer` 두 개의 버퍼로 나누어서 다루어<br>
`Front Buffer`는 직접 화면을 보여주며 `Back Buffer`는 다음에 그려질 화면을 미리 그려둔 뒤,<br>
화면이 갱신되어야 할 때 `두 버퍼를 빠르게 스왑`하여 `부드러운 화면전환`을 만든다.

`swapChain`이란 이 두 버퍼를 관리하는 역할을 한다.<br>



### 사용법
DirectX11에서 제공하는  `D3D11CreateDeviceAndSwapChain`함수를 이용하면 Devcie, DeviceContext, SwapChaine을 한 번에 만들 수 있다.<br>
함수를 사용하기 위해선 `SwapChain을 어떻게 만들지의 정보`를 담고 있는 `DXGI_SWAP_CHAIN_DESC`가 필요하다.<br>
그 뒤 Device와 DeviceContext에 대한 정보는 함수 안에서 처리한다.


```cpp
#include <d3d11.h>

DXGI_SWAP_CHAIN_DESC desc;
// 일단 desc를 싹 비움
ZeroMemory(&desc, sizeof(desc));

// desc값을 하나하나 넣어줌
{
	desc.BufferDesc.Width = _width;     // 너비
	desc.BufferDesc.Height = _height;   // 높이 1920x1080을 만들고 싶다면 width 1920 height 1080
	desc.BufferDesc.RefreshRate.Numerator = 60; // 화면 주사율의 분자 60번 갱신
	desc.BufferDesc.RefreshRate.Denominator = 1;// 화면 주사율의 분모 Numerator/Denominator로 60hz를 의미
	desc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM; // 버퍼의 색상 포맷을 설정, 32비트 색상 포맷 설정
	desc.BufferDesc.ScanlineOrdering = DXGI_MODE_SCANLINE_ORDER_UNSPECIFIED; // 스캔 라인 방식 설정, 스캔 라인 순서를 지정하지 않음
	desc.BufferDesc.Scaling = DXGI_MODE_SCALING_UNSPECIFIED; // 화면 크기 조정 방식, 크기조정방식에 제한을 두지 않겠다 설정
	desc.SampleDesc.Count = 1;		// 멀티샘플링 설정, 멀티 샘플링 사용안함 -> 계단현상 딱히 신경안씀
	desc.SampleDesc.Quality = 0;    // 샘플링 품질 설정, 최저 품질(멀티샘플링을 사용안해서 딱히 의미없음)
	desc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT; // 버퍼의 용도를 설정, 버퍼를 최종결과 화면을 그리는 용도로 사용
	desc.BufferCount = 2;   // 스왑체인에 사용할 버퍼의 개수, 2개사용
	desc.OutputWindow = _hwnd; // 렌더링 결과를 출력할 창, 윈도우 창에 렌더링
	desc.Windowed = true;   // 창모드 설정
	desc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD; // 스왑 기타 효과설정, 여기선 버퍼를 교체할 때 기존 프론트 버퍼를 저장하지 않고 버림
}


// 함수를 실행하면 _swapChain, _device, _deviceContext 변수가 채워짐
HRESULT hr = ::D3D11CreateDeviceAndSwapChain(
	nullptr,                 // IDXGIAdapter*, 그래픽 어댑터 지정, nullptr이면 기본 GPU로 설정
	D3D_DRIVER_TYPE_HARDWARE,// D3D_DRIVER_TYPE, 그래픽 드라이버 설정, 하드웨어 기반 드라이버 사용
	nullptr,                 // HMODULE, 소프트웨어 렌더링 설정, 기본설정이용
	0,                       // UINT, 플래그 , 따로 사용x , 디버그용으로 따로 설정가능
	nullptr,                 // const D3D_FEATURE_LEVEL* 기능 수준 명시, 직접 DirectX의 Feature_Level을 명시할 수 있고 nullptr시 자동으로 설정
	0,                       // UINT, 기능 수준의 개수 명시
	D3D11_SDK_VERSION,       // UINT, DirectX 11 SDK 버전 참조
	&desc,                   // const DXGI_SWAP_CHAIN_DESC*, SwapChain인자
	_swapChain.GetAddressOf(),// IDXGISwapChain**  swapchaine을 담을 더블 포인더 변수
	_device.GetAddressOf(),   // ID3D11Device** device를 담을 더블 포인터 변수
	nullptr,                  // D3D_FEATURE_LEVEL* 기능 수준 포인터 변수, 사용x
	_deviceContext.GetAddressOf() // ID3D11DeviceContext** deviceContext를 담을 더블 포인텨 변수
);

```
