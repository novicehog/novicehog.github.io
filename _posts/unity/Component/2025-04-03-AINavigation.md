---
layout: single
title:  "Unity AI Navigation"
categories: 
    - Unity
tag: [Unity]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2025-04-03
last_modified_at: 2023-04-04

---

# AI Navigation
Unity의 `AI Navigation` 시스템은 게임 오브젝트가 씬 내에서 `지형과 장애물을 인식`하고<br>
`스스로 경로를 찾아 이동`할 수 있도록 해주는 강력한 기능이다.

기본적으로 내장되지 않은 경우 Package Manager -> Unity Registry 에서 설치가 가능하다.

## 사용법
AI Navigation은 크게 세 가지로 이루어져 있다.

- NavMesh Surface: `씬에서 이동 가능한 영역을 정의`하는데 사용.
- NavMesh Agent: `경로를 따라 이동하는 캐릭터에 부착`되는 컴포넌트로, 목표 위치를 지정하면 스스로 최적의 경로를 계산해 이동.
- NavMesh Obstacle: AI의 이동을 막는 장애물로, 동적 장애물도 처리할 수 있음.


### NavMesh Surface
여러가지 설정값들을 고려하여 Agent가 움직일 수 있는 영역을 정의하는데 사용된다.<br>

![surface Component](https://github.com/user-attachments/assets/7cc6eca7-c771-4e80-98f3-6d6c90029dc1)


<details>
<summary>AgentType</summary>
<div markdown="1"> 

영역을 지나다닐 Agent의 Type을 지정한다.<br>
AgentType은 Window -> AI -> Navigation 창에서 새로 만들 수 있으며
Agent의 여러 설정값들을 조절한다.

![AgentType](https://github.com/user-attachments/assets/720ab2c0-de93-4ad5-ba47-fda045ac7f0c)

</div>
</details>


Default Area : 기본으로 설정할 영역을 지정

Use Geometry : Nav Mesh를 생성할 기준을 정함.
    Render Meshes -> Mesh를 기준으로 생성
    Physics Collider -> Collider를 기준으로 생성