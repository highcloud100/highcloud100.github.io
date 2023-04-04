---
title: Process Description and Control 1
author: highcloud
date: 2023-04-04  17:44:54 +0900
categories: [CS, CSE3206_OS]
tags: [os]
render_with_liquid: false
---

# Process Description and Control 1

## 1. 프로세스란 무엇인가?

프로세스의 정의
- 실행 중인 프로그램
- 프로세서에 할당되고 실행되는 것
- 명령어의 실행 흐름과 현재 상태, 관련된 시스템 자원으로 특징되는 활동의 단위

## 2. 프로세스와 프로그램의 차이

- "프로그램"은 디스크에 바이너리 형태로 저장된 상태이다. 
- "프로세스"는 활동하는 상태로 이는 실행 흐름이 있다는 뜻이다. 
  - 실행 파일이 메모리에 올라왔을 때 프로그램에서 프로세스가 된다. 
- 프로세스는 두가지 중요한 요소가 있다. 
  - 코드
  - 코드에 관련된 데이터


### Stack
- 실행 흐름을 만들기 위해 스택이라는 저장 공간이 필요하다.
  - call stack ( stack in OS )
      - procedure calls를 tracking하거나 parameter를 전달하는데 stack을 사용
- 프로세서가 call을 하면
  - return address를 스택에 저장한다.
  - parameters를 스택에 저장해 called procedure에 전달 가능하다.
- call stack은 stack frames로 구성되어 있다. 


## 3. PCB

- OS에서 가장 중요한 자료구조이다. 
  - struct로 구현됨

### main purpose
- interrupt가 발생시 프로세스를 교환하여 실행 가능하게 만든다.
- 즉 interrupt 이후에도 interrupt가 발생 안한 것 같이 실행된다.  
- 이를 위해 interrupt시 pc나 registers 같은 context data를 PCB에 저장한다. 
- multiple processes와 multiprocessing을 지우너하기 위함이다. 

### Elements of a Process control block
1. Identifiers
   - PID, PPID ...
2. Processor State Information
   - 각종 레지스터들
     - PC, condition codes
     - status information : interrupt enabled flag, execution mode
     - Stack pointer
3. Process Control Information
   - 프로세스 상태
     - running, ready, blocked ...
   - 우선순위, 각종 스케줄 관련 정보
   - Event
     - 프로세스가 기다리는 이벤트 
   - Memory Management
     - 페이지 테이블 관련한 포인터들
   - 자료구조
     - 다른 프로세스간 연결 리스트 같은 것들


PCB는 모든 OS 모듈에서 읽고 수정할 수 있다. 
OS는 PCB의 링크드 리스트로 구현된 큐를 이용해 프로세스 관리한다.


## 4. Process Image

os가 프로세스를 컨트롤하고 관리하기 위한 정보들
1. user level context
   - user program, data, stack
   - 프로그램 요소들 ( text, data ...)과 실행시 사용하는 stack
2. system level context
   - PCB
     - os가 프로세스를 관리하기 위해 필요한 정보

프로세스를 실행하기 위해서 프로세스 이미지가 메인 메모리에 반드시 올라와야한다. 

- 프로세스 이미지
    - user data
    - user program 
    - stack
    - PCB


## 5. 프로세스의 실행 흐름

- process switch를 통해 번갈아 실행됨
  - mode switch가 선행되고 이후 process switch
- 언제 process switch 발생하는가?
    - timer interrupt
    - system call ( I/O )
    - Exception

> 인터럽트가 반드시 process switch를 발생 시키지 않는다. 

## 5-1. Two-State Process Model

<figure>
    <img src="https://media.geeksforgeeks.org/wp-content/uploads/20211208184424/gfgf1.PNG">
    <figcaption style="text-align: center">geeksforgeeks</figcaption>
</figure>

- time out된 process와 I/O 연산을 기다리는 process가 같은 큐에 저장된다. 
  - 대기 큐에 바로 실행 가능한 프로세스와 그렇지 않은 프로세스가 섞여있어 오버헤드가 발생한다. 
  - I/O 기다리는 프로세스는 dispatch되고 바로 pause될 것이다.  


## 5-2 Five-State Process Model

- Not running state를 Ready와 Blocked로 나눔
  - Blocked에 있는 프로세스들은 event를 받으면 Ready 상태로 이동함
<figure>
    <img src="https://media.geeksforgeeks.org/wp-content/uploads/20211221191700/5stae.PNG">
    <figcaption style="text-align: center">geeksforgeeks</figcaption>
</figure>


- Ready: 바로 실행 가능한 프로세스들을 모아둠
- New : 아직 실행 풀에 들어가지 못함 ( admit 받으면 입장함)
- Blocked: 이벤트가 발생할 때까지 기다리는 프로세스
- Exit : 종료된 프로세스들

큰 OS에서는 Blocked를 이벤트 종류 별로 여러 큐를 만들어 사용하는 것이 효율적이다. ( 이벤트 서칭에서 이득 )

## 5-3 Seven-State Process Model

#### Swapping
- 메인메모리의 프로세스를 디스크로 욺겨야 할 상황이 있다. 
- blocked, ready 상태에서 디스크로 옮겨지면 suspend 상태가 된다. 
<figure>
    <img src="https://d2vlcm61l7u1fs.cloudfront.net/media%2F28b%2F28bf9cb9-cd26-42f3-b007-09757e5da940%2FphpZ0uG64.png">
    <figcaption style="text-align: center">chegg</figcaption>
</figure>