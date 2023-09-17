---
layout: single
title:  "유니티 다크소울 따라만들기 ch_2 카메라 조작"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-16
last_modified_at: 2023-09-16

---
이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}

<!-- 
{: .notice--warning} // 알림 강조
{: .notice--success} // 초록색 강조
{: .notice--danger } // 초록색 강조
{: .notice--info}
{: .notice--primary}
{: .notice}

{: .H1-font}         // 제목 색
<span style="color:Skyblue"> 색 넣기 </span>
<br/> 한줄 내리기

<details>
<summary>VR</summary>
<div markdown="1">       
</div>
</details> 
 -->



# 초기 설정

## 카메라 움직임의 축이되는 빈 오브젝트 생성

다음과 같이 카메라가 Camera Holder의 자식 Camera Pivot의 자식으로 설정함 
![image](https://github.com/novicehog/comments/assets/131991619/65c76acf-18a5-4f1a-8709-2c280331a639)

이 스크립트의 구현 원리는 다음과 같다.
![image](https://github.com/novicehog/comments/assets/131991619/deeb5d3c-e3e5-4b99-ac8c-3bb04e59ff1d)

최상위 부모인 `Camera Holder` 가 Y축의 회전을 담당하여 좌우를 움직이고, <br>
그 다음 `Camera Pivot` 가 X축의 회전을 담당하여 위아래로 회전한다. <br>
그리고 이 둘에게 종속되는 카메라는 자연스럽게 좌우 위아래 회전이 된다.

## Scripts

### CameraHandler
***

<details>
<summary>CameraHandler</summary>
<div markdown="1">     

```c#
public class CameraHandler : MonoBehaviour
{
    public Transform targetTransform;                // 플레이어 캐릭터
    public Transform cameraTransform;                // 카메라 
    public Transform cameraPivotTransform;           // 카메라 피벗
    private Transform myTransform;                   // 카메라 홀더
    private Vector3 cameraTransformPosition;
    private LayerMask ignoreLayers;

    public static CameraHandler singleton;

    public float lookSpeed = 0.1f;
    public float followSpeed = 0.1f;
    public float pivotSpeed = 0.03f;

    private float defaultPosition;
    private float lookAngle;
    private float pivotAngle;
    public float minimumPivot = -35;
    public float maximumPivot = 35;


    private void Awake()
    {
        singleton = this;
        myTransform = transform;
        defaultPosition = cameraTransform.localPosition.z;
        ignoreLayers = ~(1 << 8 | 1 << 9 | 1 << 10);
    }

    public void FollowTarget(float delta)
    {
        Vector3 targetPosition = Vector3.Lerp(myTransform.position, targetTransform.position, delta / followSpeed);
        myTransform.position = targetPosition;
    }

    public void HandleCameraRotation(float delta, float mouseXInput, float mouseYInput)
    {
        lookAngle += (mouseXInput * lookSpeed) / delta;
        pivotAngle -= (mouseYInput * pivotSpeed) / delta;
        pivotAngle = Mathf.Clamp(pivotAngle, minimumPivot, maximumPivot);

        Vector3 rotation = Vector3.zero;
        rotation.y = lookAngle;
        Quaternion targetRotation = Quaternion.Euler(rotation);
        myTransform.rotation = targetRotation;

        rotation = Vector3.zero;
        rotation.x = pivotAngle;

        targetRotation = Quaternion.Euler(rotation);
        cameraPivotTransform.localRotation = targetRotation;
    }
}


```
</div>
</details> 

카메라 핸들러 부분은 크게 어려운 부분이 없으므로 간단히 기록해두겠다.
<br>

#### 싱글턴
***
여기서는 싱글턴이 매우 간단하게 선언되었다.
static 을 통해 스스로를 선언하면서 다른 클래스에서 자유롭게 CameraHandler 객체에 접근이 가능하도록 되어있다. 

```c#
private void Awake()
{
    singleton = this;
}
```
<br>

#### LayerMask
***
레이어는 유니티 내에서 1,2,3,4... 의 숫자로 설정되지만
스크립트 안에서는 1,2,4,8,16... 의 형태로 사용해야한다. <br>

그래서 다음 코드는
8, 9, 10 번 레이어를 제외한(`~`) 다른 모든 레이어들을 의미한다. 
```c#
private void Awake()
{
    ignoreLayers = ~(1 << 8 | 1 << 9 | 1 << 10);
}
```
<br>

#### 카메라 회전
***
카메라 회전은 처음 설명한 것 대로 구현된다.

```c#
public void HandleCameraRotation(float delta, float mouseXInput, float mouseYInput)
{
    
    lookAngle += (mouseXInput * lookSpeed) / delta;
    pivotAngle -= (mouseYInput * pivotSpeed) / delta;

    // 위 아래 회전은 제한을 둔다.
    pivotAngle = Mathf.Clamp(pivotAngle, minimumPivot, maximumPivot);

    // 카메라 홀더의 회전(Y축 좌우)를 담당하는 부분
    Vector3 rotation = Vector3.zero;
    rotation.y = lookAngle;
    Quaternion targetRotation = Quaternion.Euler(rotation);
    myTransform.rotation = targetRotation;


    // 카메라 피벗의 회전(X축 위 아래)를 담당하는 부분
    rotation = Vector3.zero;
    rotation.x = pivotAngle;

    targetRotation = Quaternion.Euler(rotation);
    cameraPivotTransform.localRotation = targetRotation;
}
```
<br>

### InputHandler 에 추가된 부분.
***
```c#
// 카메라 핸들러를 변수 선언
CameraHandler cameraHandler;

private void Awake()
{
    // 카메라 핸들러 객체를 가져옴
    cameraHandler = CameraHandler.singleton;
}
private void FixedUpdate()
{
    // 카메라 회전
    float delta = Time.fixedDeltaTime;

    if(cameraHandler != null)
    {
        cameraHandler.FollowTarget(delta);
        cameraHandler.HandleCameraRotation(delta, mouseX, mouseY);
    }
}
```
<br>

### PlayerLocomotion에 추가된 부분
***
```c#
public void Update()
{
    float delta = Time.deltaTime;

    inputHandler.TickInput(delta);

    moveDirection = cameraObject.forward * inputHandler.vertical;
    moveDirection += cameraObject.right * inputHandler.horizontal;
    moveDirection.Normalize();

    // 이 부분이 추가됨 (카메라가 바라보는 방향을 통해 움직임으로)
    // 위로 솟거나 아래로 박히며 이동하는 것을 막기위해 y축 이동을 없앰
    moveDirection.y = 0; 

    float speed = movementSpeed;
    moveDirection *= speed;



    Vector3 projectedVelocity = Vector3.ProjectOnPlane(moveDirection, normalVector);
    rigidbody.velocity = projectedVelocity;

    animatorHandler.UpdateAnimatorValues(inputHandler.moveAmount, 0);

    if (animatorHandler.canRotate)
    {
        HandleRotation(delta);
    }
}
```
<br>

# 배운점
1. 3인칭 시점의 구현 방법이 어떻게 되는지 감을 잡을 수 있었음.