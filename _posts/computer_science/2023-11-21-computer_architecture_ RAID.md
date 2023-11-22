---
layout: single
title:  "[컴퓨터 구조] 보조 기억 장치 RAID"
categories: 
    - cs
tag: [cs, computer_architecture]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-11-23
last_modified_at: 2023-11-23
---

이 글은 패스트캠퍼스의 **현실 세상의 컴퓨터공학 지식 with 30가지 실무 시나리오 초격차 패키지 Online.**를 보고 공부한 내용을 정리한 글입니다.
{: .notice--warning}

# RAID란
보조 기억 장치를 `더 높은 성능`과 `더 높은 안전성`으로 관리하는 방법이다.

## RAID의 단계
이러한 보조 기억 장치 관리 방법인 RAID에는 여러가지 단계가 존재한다.

RAID 0, RAID 1 , RAID 2...

### RAID 0

![image](https://github.com/novicehog/comments/assets/131991619/e37c1037-f8db-4f50-b463-787dd18ab514)

- 데이터를 단순히 보조 기억 장치에 나누어 저장하는 구성 방식 : 성능 항샹 / 신뢰성 감소
- 마치 줄무늬처럼 저장된 데이터 : 스트라입
- 분산하여 저장하는 것 : 스트라이핑
- 데이터를 여러 하드디스크에 나눠서 저장하므로 데이터가 필요할 때 <br>
    `여러 하드디스크를 동시에 접근`하여 데이터를 가져올 수 있으므로 `성능은 올라`가나<br>
    `하드디스크 중 하나라도 고장`날 경우 `나머지 데이터도 쓸모없어`지기에 `신뢰성이 감소`함


즉 RAID는 `성능을 위해 신뢰성을 포기`한 케이스이다.


### RAID 1

![image](https://github.com/novicehog/comments/assets/131991619/e47bd340-f6f4-4a04-966d-5345c717a03c)

- 복사본을 만드는 방식 (미러링)
- 쓰기 성능의 감소, 저장 공간 감소, 신뢰성 증가(복구 용이)
- 신뢰성 확보를 위하여 복사본을 만들어 저장하므로 <br>
    데이터를 저장할 때 복사본 까지 `두 번 저장`하므로 `쓰기 성능이 감소`하며 `저장공간이 감소`하지만<br>
    한 `디스크가 고장나더라도 복사본이 있으`므로 `신뢰성이 증가`한다.
    

RAID 1은 `신뢰성을 위해 성능을 포기`한 케이스이다.


### RAID 4

![image](https://github.com/novicehog/comments/assets/131991619/a81e5e26-67a5-482a-bcd6-0ccae1dcaf1d)

- `패리티 비트(parity bit)`라는 오류 검출용 비트를 저장하는 장치르 따로 두는 방식
- RAID 1에 비해 `적은 하드 디스크로도 신뢰성 증가` 가능
- 단, 패리티 비트를 저장한 디스크에 `병목 현상`이 증가(오류 검출을 위해 접근해야 하는 디스크는 한개로 고정되므로)


#### 패리티 비트란
> 오류를 검출할 수 있는 특별한 정보
 
- 홀수 패리티 : 전체 1의 개수가 홀수가 되도록 패리티 비트를 정하는 방식
- 짝수 패리티 : 전체 1의 개수가 짝수가 되도록 패리티 비트를 정하는 방식
- `두개 이상의 비트에 문제가 생길 경우` `오류 검출 불가능`(문제가 두 개라면 홀짝의 판정이 달라지지 않음으로)
- 본래 패리티 비트는 오류 검출용 비트일 뿐, 복구능 불가능하지만 `RAID`에서는 `어느 정도의 복구가 가능`

![image](https://github.com/novicehog/comments/assets/131991619/da0ab6d2-0797-4f45-a42d-851404c8bf93)

### RAID 5

![image](https://github.com/novicehog/comments/assets/131991619/6538d456-1a37-4ba4-88fa-539e025ba980)

- `패리티 비트를 분산하여 저장`하여 `RAID 4의 병목 현상을 해소`

### RAID 6

![image](https://github.com/novicehog/comments/assets/131991619/c70f6cf8-f773-44c5-aa17-ff584657a0f4)

- 패리티를 두 개를 두는 방식
- `RAID 4나 5보다 더욱 신뢰성이 높아`진 방식(단, 패리티를 두개 사용하므로 `쓰기 성능은 감소`)

### nested RAID
여러가지 RAID는 혼합하여 사용하는 것.
