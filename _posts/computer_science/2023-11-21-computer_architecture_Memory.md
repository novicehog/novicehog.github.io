---
layout: single
title:  "[컴퓨터 구조] 저장장치 메인 메모리"
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

## 메인 메모리

메인 메모리에는 두 가지의 하드웨어가 있다.

- `RAM`(Random Acces Memory)
- `ROM`(Read Only Memory)

메인 메모리는 위처럼 두 가지가 있지만 흔히 메모리라고 칭하는 용어는 RAM을 가르킨다.

### RAM(Random Acces Memory) : 휘발성 저장장치

RAM은 `실행중인 프로그램`이 담겨있는 저장장치로서 전원을 끄면<br>
저장된 데이터가 `모두 날아가는 특징`을 가지고 있다.

RAM은 실행중인 프로그램을 저장하고 있으므로 흔히 책상에 비유되곤한다.

`책상(메모리)이` 크면 클 수록 한 번에 읽어들일 수 있는 `책(데이터)`의 수가 많아진다.

![image](https://github.com/novicehog/comments/assets/131991619/a475f57b-f710-4eb5-8828-4e8a3619fa81)

***
RAM 종류는 4가지가 있다.
- DRAM (Dynamic Ram)
- SRAM (Static RAM)
- SDRAM (Synchromous Dynamic RAM)


#### DRAM과 SRAM
`DRAM(Dynamic Ram)` 
- 시간이 지나면 점차 저장된 데이터가 사라지는 RAM
- 전원이 연결되어 있어도 지속적으로 데이터가 사라지므로 계속 재충전 해줘야함
- 메인 메모리에서 주로 사용되는 RAM


`SRAM(Static RAM)`
- 시간이 지나도 저장된 데이터가 사라지지 않는 RAM
- 캐시 메모리에서 주로 사용되는 RAM


![image](https://github.com/novicehog/comments/assets/131991619/4a3f05e3-178c-4154-9539-9f734c806a67)


#### SDRAM과 DDR SDRAM
`SDRAM(Synchromous Dynamic RAM)`
- `클럭과 동기화`된 DRAM
- 클럭의 타이밍에 맞춰 CPU와 정보를 주고받을 수 있는 RAM
- DRAM보다 성능이 좋음

`DDR SDRAM(Double Data Rate SDRAM)`
- SDRAM에서 `대역폭을 넓혀` 속도를 높인 RAM
- DDR2 SDRAM은 DDR SDRAM보다 2배 더 대역폭이 넓으며 DDR3 SDRAM은 DDR2 SDRAM보다 2배 더 대역폭이 넓다.
- DDR뒤에 숫자가 커질 수록 `성능이 배로 좋아진다`고 생각하면 된다.

### ROM(Read Only Memory) : 읽기 전용 저장장치
- 읽기만 가능한 저장장치이다.
- 수정이 번거롭거나 불가능하다.

ROM의 종류는 다음과 같다
- Mask Rom : 가장 기본적인 ROM , 제조 과정에서 저장할 내용을 미리 기록 ex) 냉장고, 전자레인지, 텔레비전, 게임기
- PROM(Programmble ROM) : 사용자가 데이터를 `한 번` 새길 수 있는 ROM
- EPROM(Erasable PROM) : 지우고 `다시 저장가능`한 PROM
- 플래시 메모리 : 반도체 기반의 저장장치, 메인 메모리에 범주에 속한다기보다는 `범용성이 넓은 저장장치` ex) USB , SD카드, SSD



## 메모리 저장방식
CPU가 메모리에 데이터를 저장하는 방식에는  <br>
리틀엔디안과 빅엔디안 방식이 있다.

### 엔디안
- 연속해서 저장해야 하는 바이트를 저장하는 `순서`
- 나라마다 글을 읽는 순서가 다르듯, 메모리에 바이트를 밀어넣는 순서도 다름.
- 리틀 엔디안과, 빅 엔디안 방식이 있음.


#### 빅 엔디안
- `낮은 번지 주소`부터 `상위 바이트`부터 저장하는 방식
- 여기서 말하는 상위 바이트란 수를 이루는 가장 큰 값(가장 높은자릿수 MSB)
- 1A2B 3C4D를 저장한다하면 낮은 주소 번지부터 1A, 2B, 3C, 4D순으로 저장

![image](https://github.com/novicehog/comments/assets/131991619/46c817ee-cfa5-4fec-961b-ed99766745dd)

#### 리틀 엔디안
- `낮은 번지 주소`부터 `하위 바이트`부터 저장하는 방식
- 여기서 하위 바위트란 수를 이루는 가장 작은 값(LSB)
- 1A2B 3C4D를 저장한다하면 낮은 주소 번지부터 4D, 3C, 2B, 1A 순으로 저장

![image](https://github.com/novicehog/comments/assets/131991619/0bddd7fd-44f7-4894-8c06-4e09fea89816)

#### 두 엔디안의 장단점
빅 엔디안
- `일상적으로 쓰는 숫자 체계의 순서와 동일`하므로 편리

리틀 엔디안
- 수치 계산(자리 올림 등)이 편리 (뒷자리에서부터 계산하므로)


## CPU가 메모리에 접근하는 방식
CPU가 알맞은 메모리에 접근하여 데이터를 얻어오려면 `올바른 주소`를 알고 있어야 한다.

하지만, 실행중인 프로그램이 가지는 주소는 일정하지 않고 시시때때로 바뀔 수 있다. <br>
즉 같은 프로그램을 두 번 실행하여도 다른 메모리에 적재될 수 있다.

이러한 문제를 가지는 상황에서 CPU는 어떤 주소체계를 사용하여 메모리에 접근하냐 하면<br>
`논리 주소`와 `물리 주소`를 이용할 수 있다.

### 논리 주소와 물리 주소
물리 주소 : 실제 메모리의 하드웨어에서 상의 주소
논리 주소 : CPU 실행 중인 프로그램이 사용하는 주소 (0번지부터 시작)

`모든 프로그램`은 0번지부터 시작하는 `각자의 논리 주소`를 사용한다.

![image](https://github.com/novicehog/comments/assets/131991619/bb7b6e88-ff0b-4e2f-b5aa-baf7fb29984b)

이렇듯 프로그램 각자가 가진 `논리 주소`를 `물리 주소`로 변환해야만 CPU는 메모리에 올바르게 접근할 수 있다.<br>
이때 주소를 변환해주는 장치를 MMU(Memory Management Unit)라고 한다.

![image](https://github.com/novicehog/comments/assets/131991619/8777b49a-dcfa-4a97-a324-3e9fda96bce2)

#### MMU의 기본 동작
- 주소 변환은 `베이스 레지스터`를 활용한다.
- 베이스 레지스터 = 기존 주소(`물리 메모리상 첫 주소`)
- 논리 주소 = 기존 주소로부터 떨어진 거리
- 즉 프로그램마다 `첫 주소를 알아내어` `논리 주소를 더함`으로서 `물리 주소를 알아낸다.`

![image](https://github.com/novicehog/comments/assets/131991619/b69dbd40-2660-4491-bcbf-ea6da608a955)

추가로 `한계 레지스터라`는 것도 있는데 이것은 `프로그램의 크기`를 나타내며
주소 변환시 물리 주소를 알아낼 때 프로그램 주소를 벗어나는것을 방지해준다.



## 캐시 메모리
- `CPU와 메모리 간`의 `속도 차를 극복하기 위해` 탄생
- CPU와 메모리 사이에 위치, 레지스터보다 용량이 크고 메모리보다 빠른 SRAM 기반
- CPU에서 사용할 법한 정보를 `미리 가져와 저장`

![image](https://github.com/novicehog/comments/assets/131991619/022b88d7-ed25-4d0e-a368-25ec2f49dc95)


### 캐시 메모리 단계
캐시 메모리는 `CPU와의 거리로 여러 단계로 나눌 수 있음`

- L1(Level 1) 캐시 메모리 : CPU와 더 가까우며 용량이 작음
- L2(Level 2) 캐시 메모리
- L3(Level 3) 캐시 메모리 : CPU와 멀고 용량이 큼

![image](https://github.com/novicehog/comments/assets/131991619/f09e49d3-f17c-4d5e-902b-bca76b201b65)

### 캐시 메모리에 저장되는 데이터
캐시 메모리는 CPU가 자주 사용할 법한 내용을 `예측하여 저장`한다.

예측을 하는 방법은 두 가지의 `참조 지역성의 원리`를 따른다.

- 시간 지역성 : CPU는 `최근에 접근했던 메모리 공간`에 `다시 접근`하려는 경향이 있음
- 공간 지역성 : CPU는 `접근한 메모리 공간 근처`를 접근하려는 경향이 있다.



