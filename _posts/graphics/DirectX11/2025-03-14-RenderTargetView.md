---
layout: single
title:  "[DirectX11] RenderTargetView"
categories: 
    - Graphics
tag: [DirectX11]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2025-03-14
last_modified_at: 2025-03-14

---
이 글은 인프런 강의 **[게임 프로그래머 도약반] DirectX11 입문**를 보고 따라 만들면서 헷갈리는 부분을 복습하며 정리한 글입니다.
{: .notice--warning}

# RenderTargetView
DirectX11에서 `RenderTargetView`란 쉽게 말해서 그림을 그릴 `도화지`를 `가리키는 것`이다.<br>
여기서 말하는 `도화지`는 swapChain의 `backBuffer(다음 프레임에 보여질 텍스쳐)`이고 <br>
`RenderTargetView`는 이 backBuffer를 `가리키고 있는 것`이라고 생각하면 된다.


RenderTargetView를 사용하기 위해선 크게 다음 3단계를 따라가면 된다.<br>
1. RenderTargetView를가 BackBuffer를 가리키게함
2. RenderTargetView를에다가 그리라고 파이프라인에서 지정함
3. BackBuffer를 이용해서 화면에 출력해라.


## 1. RenderTargetView에 BackBuffer지정
우선 RendertagetView를 변수로 만들어야하니 Comptr로 생성한다.

```cpp
ComPtr<ID3D11RenderTargetView> _renderTargetView;
```


이제 RendertagetView를 deivce를 통해 만드는데 만들어둔 swapChain의 backbuffer를 가리키게 만든다.

```cpp

HRESULT hr;

// backBuffer를 저장할 변수
ComPtr<ID3D11Texture2D> backBuffer = nullptr;

// backBuffer변수에 swapChain의 backbuffer 저장
hr = _swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (void**)backBuffer.GetAddressOf());
CHECK(hr);

// backBuffer를 가리키는 RenderTargetView를 생성
_device->CreateRenderTargetView(backBuffer.Get(), nullptr, _renderTargetView.GetAddressOf());
CHECK(hr);

```

## 2. RenderTargetView를 도화지로 지정

```cpp
// 도화지 지정
_deviceContext->OMSetRenderTargets(1, _renderTargetView.GetAddressOf(), nullptr);
// 도화지를 특정 색으로 싹 밀어버림
// color는 0.5,0.5,0.5,0 으로 하였음(회색)
_deviceContext->ClearRenderTargetView(_renderTargetView.Get(), _clearColor);
// 도화지에서 실제로 그림을 그릴 부분을 정함
_deviceContext->RSSetViewports(1, &_viewport);
```

<br>
이 작업은 매 프레임마다 렌더링 파이프라인에서 맨 처음으로 해줘야한다.

## 3. 화면에 출력
모든 파이프라인 과정이 끝나면 SawpChain의 버퍼를 갈까끼움으로 새 화면을 띄운다.

```cpp
// 버퍼 교체
HRESULT hr = _swapChain->Present(1, 0);
CHECK(hr);
```

## 결과
화면을 띄우면 다음과 같이 회색으로 칠해진 화면이 나온다.
![렌더링결과](https://github.com/user-attachments/assets/67f606b2-8856-4fc3-867c-fd6d49ee3b01)

앞으로 그려질 렌더링들은 이 회색 화면 위에 덧 씌워서 그려질 것이다.


