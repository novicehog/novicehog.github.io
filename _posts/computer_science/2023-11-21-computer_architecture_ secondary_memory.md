---
layout: single
title:  "[컴퓨터 구조] 저장장치 보조 기억 장치"
categories: 
    - cs
tag: [cs, computer_architecture]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-11-21
last_modified_at: 2023-11-21
---

이 글은 패스트캠퍼스의 **현실 세상의 컴퓨터공학 지식 with 30가지 실무 시나리오 초격차 패키지 Online.**를 보고 공부한 내용을 정리한 글입니다.
{: .notice--warning}

# 저장장치
저장장치의 특성은 간단하다 CPU와 멀어질 수록 `속도 ↓` `비용 ↓` `용량 ↑` 의 특징을 가진다.

그렇기 때문에 CPU와 가장 가까운 저장장치인 레지스터의 경우 매우 빠른 속도를 가지지만 용량대비 비싼 비용을 가진다.


## 보조 기억 장치
보조 기억장치는 크게 두 가지가 있다.

- 하드 디스크 드라이브(HDD)
- 플래시 메모리 기반 (SSD, USB, SD카드)

### 하드 디스크
#### 하드 디스크의 부품
하드 디스크를 이루는 부품을 먼저 살펴보면

![image](https://github.com/novicehog/comments/assets/131991619/548ef365-33ec-4e36-b9bc-122cf8e02793)

플레터 : 하드 디스크 상에서 `실질적으로 데이터가 저장`되는 부분
스핀들 : 플레터를 회전시키는 부분

<br>

![image](https://github.com/novicehog/comments/assets/131991619/78795712-a260-4f89-a556-629e399a9c08)

헤드 : 플레터의 `데이터를 읽고 쓰는 부분`
디스크 암 : 헤더를 옮기는 부분


#### 하드디스크의 데이터 단위
![image](https://github.com/novicehog/comments/assets/131991619/94f6322f-596f-4c1b-8131-ddbeaa20fefb)

트랙 : 플래터 상의 `동심원`
섹터 : `트랙을 나눈 단위` (가장 작은 단위)
실린더 : `여러 개의 트랙`을 모은 단위
블록 : `실제 입출력이 수행되는 단위`

#### 하드 디스크 지연시간
![image](https://github.com/novicehog/comments/assets/131991619/8ad2b077-2fba-4beb-9ca4-fd6077b15534)

탐색 시간 : `헤드`를 `원하는 섹터까지 이동`시키는 시간
회전 지연 : 원하는 `섹터`를 `헤더까지 회전`시키는 시간
전송 시간 : 데이터를 송수신하는 시간



## 플래시 메모리

![image](https://github.com/novicehog/comments/assets/131991619/5f792fb9-2f3f-4505-9983-2c7d81ab7144)

- `반도체 기반`의 저장 장치
- `범용성이 매우 높은` 저장장치로 플래시 메모리는 `보조 기억 장치로 쓰이기도 한다.`

### 플레시 메모리 저장 단위
- 셀(cell) : 플래시 메모리의 가장 작은 저장 단위
- `한 셀에 몇 비트까지 저장이 가능한지`에 따라 플래시 메모리의 `종류가 나뉨`
    - SLC 타입 : 한 셀에 한 비트 저장 가능 
    - MLC 타입 : 한 셀에 두 비트 저장 가능
    - TLC 타입 : 한 셀에 세 비트 저장 가능
    - ...


SLC, MLC, TLC를 비교해보면 `한 셀에 적은 비트를 저장할 수록` `비싸고 성능이 좋다고` 생각하면 된다.

![image](https://github.com/novicehog/comments/assets/131991619/d581635e-a9dc-418a-851d-4c12d895ac3c)

***

플래시 메모리에서는 읽고 쓰기 단위와 삭제 단위가 다르다.
- `읽기 쓰기`는 `페이지` 단위
- `삭제`는 `블록` 단위

![image](https://github.com/novicehog/comments/assets/131991619/828330ae-f64b-41dc-94b8-fc35e60769cf)

