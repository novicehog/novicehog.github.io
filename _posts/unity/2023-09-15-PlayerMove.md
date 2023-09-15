---
layout: single
title:  "유니티 다크소울 따라만들기 ch_1 플레이어 이동"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-15
last_modified_at: 2023-09-15

---
이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}

# 초기 설정

## InputSystem 설정 

먼저 인풋 시스템을 패키지에서 설치

![image](https://github.com/novicehog/comments/assets/131991619/4d760224-515d-448b-bcb0-8fd7e031991b)

<br>

그 후 유니티를 재시작하고<br>
Create -> InputAction 을 `PlayerControls`이름으로 생성 <br>
그 뒤 아래 사진 처럼 C# 스크립트 생성을 Apply함
![image](https://github.com/novicehog/comments/assets/131991619/17e2797f-67b4-4587-be48-7efd74500ec7)

<br>
키보드 전용 설정값들을 등록해줌
![image](https://github.com/novicehog/comments/assets/131991619/f836a962-6aea-4b20-b193-0cce0bfa6f30)lp
<br>

카메라 설정값도 추가해줌
![image](https://github.com/novicehog/comments/assets/131991619/3f8c8156-82b7-4690-af33-3008ca4e04f8)

![image](https://github.com/novicehog/comments/assets/131991619/48050f04-90bc-460d-aadc-1ccc021f1619)

![image](https://github.com/novicehog/comments/assets/131991619/fcdd1b12-b068-44e6-bd41-30c719068108)
<br>

edit -> project Setting -> Input System Package에서 다음과 같이 해줌
![image](https://github.com/novicehog/comments/assets/131991619/346017b8-1e55-4752-9e93-7ae5a7918d3d)

## Scripts
<br>

### InputHandler
<details>
<summary>InputHandler 스크립트</summary>
<div markdown="1">       
InputHandler의 스크립트는 다음과 같다.

```c#

public class InputHandler : MonoBehaviour
{
    public float horizontal;
    public float vertical;
    public float moveAmount;
    public float mouseX;
    public float mouseY;

    PlayerControls inputActions;

    Vector2 movementInput;      // 이동 관련 입력 값을 받는 변수
    Vector2 cameraInput;        // 카메라 관련 입력 값을 받는 변수

    public void OnEnable()
    {
        if (inputActions == null)
        {
            inputActions = new PlayerControls();
            inputActions.PlayerMovement.Movement.performed +=
                inputActions => movementInput = inputActions.ReadValue<Vector2>();
            inputActions.PlayerMovement.Camera.performed += i => cameraInput = i.ReadValue<Vector2>();
        }

        // 인풋 액션 활성화
        inputActions.Enable();
    }

    private void OnDisable()
    {
        inputActions.Disable();
    }

    public void TickInput(float delta)
    {
        MoveInput(delta);
    }

    private void MoveInput(float delta)
    {
        horizontal = movementInput.x;
        vertical = movementInput.y;
        moveAmount = Mathf.Clamp01(Mathf.Abs(horizontal) + Mathf.Abs(vertical));
        mouseX = cameraInput.x;
        mouseY = cameraInput.y;
    }
}
```

</div>
</details> 


#### inputAction 관련 부분

inputAction 관련해서 보면 다음과 같다.

```c#
PlayerControls inputActions; // 만들어 뒀던 InputAction

Vector2 movementInput;
Vector2 cameraInput;

public void OnEnable()
{
    if (inputActions == null)
    {
        inputActions = new PlayerControls();  // 1

        // 
        inputActions.PlayerMovement.Movement.performed +=  
            inputActions => movementInput = inputActions.ReadValue<Vector2>();
        inputActions.PlayerMovement.Camera.performed += 
            i => cameraInput = i.ReadValue<Vector2>();
    }

    // 인풋 액션 활성화
    inputActions.Enable();
}
```
<br>

**<span style="color:skyblue">1.</span>** 객체가 활성화 될 때 inputActions이 비어있다면 새롭게 만든다.<br>

**<span style="color:skyblue">2.</span>** PlayerControls에서 만들어 두었던 inputAction에 performed(입력이 되고 있다면 계속 호출)액션에 람다식을 통해
    이동 관련의 함수를 넣어준다.
![인풋액션경로](https://github.com/novicehog/comments/assets/131991619/3f07f88c-76e0-48f2-b916-28e3a81909bd)
<br>
람다식은 함수를 따로 선언하지 않고 익명의 함수를 만드는 방법으로 여기서 사용된 람다식을 
굳이 함수로 선언한다고 하면 다음과 같다고 볼 수 있다.

```c#
//inputActions.PlayerMovement.Movement.performed +=  
//inputActions => movementInput = inputActions.ReadValue<Vector2>();

// 위의 내용은 메소드를 선언하지 않는다는 것 외에는 아래와 같은 기능을 한다.
inputActions.PlayerMovement.Movement.performed += playerMove;
void playerMove(InputAction.CallbackContext inputActions)
{
    movementInput = inputActions.ReadValue<Vector2>();
}
```
<br>

**<span style="color:skyblue">3.</span>** inputActions.ReadValue<Vector2>();를 통해 입력 값을 받아온다.
이때 여기서 받아오는 값은 사전에 설정한 InputAction의 값과 일치해야한다.

![vector2](https://github.com/novicehog/comments/assets/131991619/9d676698-3664-4226-b17e-d9ad27f44f5c)

**<span style="color:skyblue">4.</span>** inputActions.Enable()으로 `inputActions을 활성화` 해준다.


#### 이동 관련 부분
inputAction을 통해 받은 입력 값을 토대로 이동 관련 로직이 구현된다.
`중요한 것`은 **<span style="color:#44C9A1">InputHandler</span>** 스크립트는 기능을 구현만을 담당하며
직접 이것들을 사용하는 것은 **<span style="color:#44C9A1">PlayerLocomotion</span>** 스크립트가 담당한다.
