---
layout: single
title:  "[컴퓨터 구조] 메인 메모리와 캐시메모리"
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

# 메인메모리와 캐시메모리

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
