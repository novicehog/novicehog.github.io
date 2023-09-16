---
layout: single
title:  "유니티 다크소울 따라만들기 ch_3 카메라 충돌"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-17
last_modified_at: 2023-09-17

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

# 추가 기능 설명
카메라가 벽 같은 곳을 통과하지 않도록 하는 기능

## Scripts

### CameraHandler에 추가된 부분
***
<details>
<summary>CameraHandler</summary>

```c#

public void FollowTarget(float delta)
{
    // 카메라가 플레이어를 부드럽게 추적함
    Vector3 targetPosition = Vector3.SmoothDamp
        (myTransform.position, targetTransform.position, ref cameraFollowVelocity, delta / followSpeed);
    myTransform.position = targetPosition;

    HandleCameraCollisions(delta);
}

private void HandleCameraCollisions(float delta)
{
    targetPosition = defaultPosition;
    RaycastHit hit;
    Vector3 direction = cameraTransform.position - cameraPivotTransform.position;
    direction.Normalize();

    if (Physics.SphereCast(cameraPivotTransform.position, cameraSphereRadius, direction, out hit, Mathf.Abs(targetPosition)
        , ignoreLayers))
    {
        float dis = Vector3.Distance(cameraPivotTransform.position, hit.point);
        targetPosition = -(dis - cameraCollisionOffset);
    }

    if (Mathf.Abs(targetPosition) < minimumCollisionOffset)
    {
        targetPosition = -minimumCollisionOffset;
    }

    cameraTransformPosition.z = Mathf.Lerp(cameraTransform.localPosition.z, targetPosition, delta / 0.2f);
    cameraTransform.localPosition = cameraTransformPosition;
}
```

<div markdown="1">       
</div>
</details> 



#### 구현 원리 설명
***
큰 틀로보면 카메라 피벗과 카메라 사이에 물체를 검사한다. <br>
그 사이에 물체가 있으면 물체의 위치로 카메라가 이동된다.
![image](https://github.com/novicehog/comments/assets/131991619/96af4d42-7265-46a0-991e-f80fe1b34a44)

좀 더 세부적인 것은 다음과 같다.
- Raycast 가 아닌 SphereCast이기 때문에 감지가 단순히 둘 사이를 <br>
단순한 직선으로 이어서 선과 무엇이 닿는지를 검사하는게 아닌 <br>  
  구 크기의 두께로 검사한다.
  
![image](https://github.com/novicehog/comments/assets/131991619/cf211273-0cd6-44be-b53d-64d43f5d2fa5)

<br>

- 어느정도의 Offset값이 있어서 위치가 어느정도 조정된다. <br>
  벽에 닿을 때 벽과의 Offset과 플레이어와 카메라의 Offset이 있다.


![image](https://github.com/novicehog/comments/assets/131991619/5470f1bf-8c0e-4b5e-9e4f-41414017a335)

<br>

## 배운점
- Raucast외에는 따로 다른 것들이 존재하는지 몰랐으나 이번으로 <br>
 box, Sphere등의 다른 레이캐스트도 존재함을 알았다.
