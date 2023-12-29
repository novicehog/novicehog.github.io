---
layout: single
title:  "[운영체제] 프로세스 상태와 생성"
categories: 
    - cs
tag: [cs, OS]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-12-27
last_modified_at: 2023-12-27
---


이 글은 패스트캠퍼스의 **현실 세상의 컴퓨터공학 지식 with 30가지 실무 시나리오 초격차 패키지 Online.**를 보고 공부한 내용을 정리한 글입니다.
이해한 내용을 바탕으로 작성했기에 틀린 내용이 있을 수 있습니다.
{: .notice--warning}

# 프로세스 상태와 생성

## 프로세스 상태
대표적인 프로세스의 상태는 다음 5가지가 있다

- 생성 상태 (new) : 프로세스가 `생성`되고, 운영체제에 등록되는 단계, PCB가 만들어짐
- 준비 상태 (ready) : 프로세스가 `CPU를 할당받기를 기다리는 단계.` 이 상태의 프로세스는 실행을 위한 `모든 준비를 마친 상태`
- 실행 상태 (running) : 프로세스가 `CPU를 할당`받아 명령어를 실행 중인 상태
- 대기 상태 (blocked) : 프로세스가 `어떤 이벤트(예를 들어, I/O 작업의 완료)`를 기다리는 상태
- 종료 상태 (terminated) : 프로세스가 `종료`되는 상태, PCB가 삭제됨

![프로세스 상태](https://github.com/novicehog/comments/assets/131991619/a5b5c233-b1b0-4be4-aa4c-69fdfaab8558)


## 프로세스 생성
### 프로세스 계층적 구조
프로세스는 최초의 프로세스를 부모로 갖고 거기서 부터 파생되어 생성되는 계층적 구조를 가지고 있다.

![프로세스 계층적 구조](https://github.com/novicehog/comments/assets/131991619/73b86372-43cd-4f67-9c55-7040ea7a8d5b)

리눅스의 터미널에서는 pstree라는 명령어를 통해 프로세스의 계층적 구조를 살펴볼 수 있다.

<img width="383" alt="image" src="https://github.com/novicehog/comments/assets/131991619/6ca99c31-1f54-4be2-816e-75507ee0276c">


### 계층적으로 프로세스가 생성되는 원리
프로세스가 계층적으로 생성되는 원리 중 하나는 `fork-exec 시스템 콜`이다.

`fork` 는 자신의 복제본을 만드는 시스템 콜이다.

![fork](https://github.com/novicehog/comments/assets/131991619/ddd13c09-dd59-47f1-9042-4f34d286e2ea)

자신을 토대로 복제본을 만들기에 `사용자 영역의 내용은 같지만` 커널영역에 저장되는 `PCB의 PID는 다르게 저장`되며 
생성되는 프로세스의 PPID는 `원본을 부모로 가르킨다.`

***

`exec` 는 새로운 코드를 덮어씌우는 시스템 콜이다.

![exec](https://github.com/novicehog/comments/assets/131991619/a8f8e618-ca72-4b60-b2ca-7f28a070f5c9)

exec가 실행되므로 복제본이 아닌 새로운 프로세스가 만들어지며 이렇게 `fork-exec가 한 묶음으로 실행`되어 `계층적으로 프로세스를 생성`할 수 있게 된다.

흔히 누구나 즐기는 게임도 이러한 과정을 거쳐 fork-exec를 통해 어떠한 부모 프로세스로 부터 생성된다.


## 정리
프로세스는 크게 5가지의 상태를 가지고 있다. 생성, 준비, 실행, 대기, 종료 <br>
프로세스는 fork-exec 시스템 콜을 통해 `계층적으로 생성`되는 원리를 가지고 있다.

