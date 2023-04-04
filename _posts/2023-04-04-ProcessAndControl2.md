---
title: Process Description and Control 2
author: highcloud
date: 2023-04-04  22:25:00 +0900
categories: [CS, CSE3206_OS]
tags: [os]
render_with_liquid: false
---

# Process Description and Control 2

## 1. Modes of Execution

- 2개의 모드를 지원한다. 
  - mode bit 이용
- 일부 명령어들은 커널에서만 실행가능하다. 
  - 인터럽트 관리
  - I/O 명령
  - 메모리 관리 / page table, TLB load
  - mode bit 변경 

## 2. Mode switch
유저모드와 커널모드간 변경을 이야기한다. 
- 프로세스의 상태 변경 없이 수행한다.
-  context를 저장하고 복구하는데 오버헤드가 있다. 
   -  커널 스택에 저장
- system call, exception, interrupt 가 발생시 커널 모드로 변경된다.  

### Mode of Execution
1. system call이나 exception 실행
   - 커널이 프로세스 대신하여 실행중이다.
   - process context에 존재한다.  
2. interrupt handler 실행
   - 프로세스와 관련이 없다. 즉 방해하는 것이다. 
     - 빠르고 간단하게 수행되어야 한다. 
   - 커널이 interrupt context에 존재한다. 
     - sleep 하면 안된다. [복잡해진다](https://leetcode.com/discuss/interview-question/125101/Why-sleeping-in-ISR-(interrupt-service-routine)-is-not-recommended/).
     - in_interrupt() 함수로 인터럽트 컨텍스트에 있는지 확인가능


정리하면 프로세서는 3가지 중 하나를 하고 있다. 
- 유저 공간에서 유저 코드 실행
- 커널 공간에서 프로세스 문맥 대신 실행
- 커널 공간에서 인터럽트 문맥 실행

## 3. Execution Model of OS

기존에는  모든 프로세스 외부에서 커널을 수행했다. 
하지만 요즘에는 프로세스 문맥에서 운영체제를 실행한다. 

- 장점
  - 현재 프로세스에서 커널 기능이 수행된다. 
  - 프로세스 스위치 없이 모드 스위치하면 된다. 

![](https://i.stack.imgur.com/Kll8K.png)

- Execute all OS function in the context of a current process
  - 프로세스 이미지가 커널의 스택, 데이터, 코드를 포함한다. 
    - data, text는 유저 프로세스간 공유된다. 
  - 커널 스택이 분리되어 존재한다. 

멜트다운과 KPTI에 관해 추가로 알아보자

<img style="text-align: center" src="https://user-images.githubusercontent.com/80192345/229778995-bb1727d1-8ad8-4f55-a09e-5ce780d54229.png" width=300>

## 4. Process Switch

### 언제 발생하냐?

1. 인터럽트
   - 이벤트가 완료되어 ready 상태 프로세스를 실행시키거나 우선순위가 높은 프로세스가 선점할때
   - 타이머 인터럽트로 인해 time slice를 모두 사용했을때
2. 예외
   - 문제가 생겼을 때 종료 시킨다. 
     - exit state로 전환
3. 시스템 콜
   - write나 read 같은 I/O 요청 명령으로 blocked 됬을때

### 어떻게 작동하냐?

모드 스위치보다 많은 노력이 필요하다.

- 현재 프로세서의 문맥을 현재 프로세스 PCB에 저장한다. 
- PCB에 프로세스 상태를 업데이트 한다. ( ready, blocked, exit )
- PCB를 적절한 큐로 이동한다. 
- 다른 프로세스를 선택한다. ( schedule )
- 선택된 PCB의 프로세스 상태를 running으로 바꾼다. 
- 프로세서 문맥을 선택된 프로세스의 저장된 문맥으로 복구한다. 

캐시 지역성을 날려버린다. 


## 5. 프로세스 생성 (Directed)

### 프로세스 생성 이유
 - 새로운 일괄 처리 작업
 - 대화형 로그온 (?? 아마 터미널로 유저가 실행하는 듯)
 - 서비스 제공을 위해 운영체제가 생성
 - 기존 프로세스가 생성 (모듈화, 병렬성을 위해)

### 생성 방법

1. 새로운 메모리 공간 할당
   - PCB와 a.out을 위한 공간
2. 코드와 데이터를 메모리로 가져온다. 
   - call stack을 만든다. 
3. PCB 초기화
4. 새로운 프로세스 실행
   - 프로세스를 ready list에 넣는다. 

처음 프로세스를 만들때만 직접 만든다. 이후부터는 clone한다. 
- /sbin/init
- systemd 

## 5.1. 프로세스 생성 (cloning)

### cloning
- 부모 프로세스를 완전히 복사한다. 
  - 환경변수가 비슷함을 볼 수 있다. (bash -ps)
- fork를 이용한다. 
  - text, data, stack, PCB를 복사
    - data, stack은 별도로 만듬
    - text 같이 read only는 공유한다. 
  - pid랑 일부 관계 데이터를 수정
  - 새로운 PCB를 ready list에 추가

### replacing

- pid는 유지되나 code, data, heap, stack을 바꿔버림
- execve 이용한다. 

### 5.2. Copy on Write

- 초기 부모의 페이지는 자식에게 공유된다.
- 둘 중 누군가 공유된 페이지를 바꾸면 OS는 copy한다.
  - 더 이상 그 페이지를 공유하지 않는다.  

<img src="https://user-images.githubusercontent.com/80192345/229801601-1a8e6f52-e230-4e9e-ab6b-ceb6d54ea413.png" width=600>

### 6. Process Termination

#### Voluntary termination : exit(status)

- main에서 return 되면 발생 ( 아마 exit 자동호출 )
- exit 를 불러서 종료한다. 
- output data가 부모로 간다. (wait 이용) 
- 프로세스의 자원이 해제된다. 

#### Involuntary termination 

- 다른 프로세스(부모)나 OS에 의해 수행
- kill, abort
- 특정 시그널 받으면 종료 
  - SIGTERM, SIGINT

#### 왜 죽이냐?
1. 불필요
2. 잘못 실행
    - 논리, 자원, 시스템적인 이유
3. OS 정책
   - ex) 부모 죽으면 밑에 자식도 죽이자.

### 6.1. Zombie

- 프로세스가 exit으로 종료된 후, 커널은 해당 프로세스의 PCB를 바로 지우지 않는다. 
  - 이후 당연히 run은 안됨
  - reaped 될 때까지 존재
  - 좀비 모드임
  
#### Reaping
- exit status가 부모에게 전달된다. 
  - wait(&state) 에서 state에 exit status가 담김 
  - 이후 커널은 좀비 자식을 죽인다. 
- 부모가 wait을 이용해 수행한다. 

<img src="https://user-images.githubusercontent.com/80192345/229805061-83a3936d-f905-43aa-aa3d-d6709436969e.png" width=500>

#### 실행 옵션
- wait을 사용하면 부모는 기본적으로 blocked된다. 
- 안쓰면 동시에 돌아가나 좀비가 남는다. 
  - 커널 reaper가 회수하고 다니기도 한다. 
- waitpid(-1, &status, WNOHANG) 같은 옵션을 사용하면 동시에 작업을 하며 주기적으로 reaping할 수 있다. 


## 7. Orphan (Unix like OS)

고아
- 자식보다 부모가 먼저 죽음
- first process에 입양된다. /sbin/init

프로세스 종료 과정
- cleanup handlers가 자동으로 call됨
- Zombie 상태로 바꾸고 exit status가 커널에 저장됨
- 프로세스가 잡고있는 시스템 자원을 해제한다. 
- 부모에게 시그널을 보냄
- 고아 자식들을 부모에게 입양시킨다. 
