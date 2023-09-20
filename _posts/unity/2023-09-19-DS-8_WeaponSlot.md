---
layout: single
title:  "유니티 다크소울 따라만들기 ch_8 무기 슬롯"
categories: 
    - Unity
tag: [Unity, DarkSouls]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-09-19
last_modified_at: 2023-09-20

---

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


이 글은 유튜브 **Sebastian Graves Create Dark Souls**를 보고 따라 만들면서 헷갈리는 부분을 정리한 글입니다.
{: .notice--warning}


# 구현 개요
무기를 손에 들고 있는 것을 만든다.

## 구현 내용
대략적인 구현 내용은 다음과 같다.

1. Item 클래스와 그것을 상속받는 WeaponItem 클래스를 만든다
2. 양 손에 무기를 생성하여 자식으로 넣거나 장착 중인 무기를 파괴하여 안보이게하는<br>
   기능이 구현된 WeaponHolderSlot 스크립트를 만들어 넣는다.
3. 양 손에 장착된 WeaponHolderSlot스크립트를 관리하는 WeaponSlotManager 스크립트를 만든다.
4. PlyaerInventory 스크립트를 하나 더 만들어 WeaponSlotManager 스크립트와 상호작용한다.

