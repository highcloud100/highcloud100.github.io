---
title: Scheduling1
author: highcloud
date: 2023-04-06  18:46:00 +0900
categories: [CS, CSE3206_OS]
tags: [ os ]
render_with_liquid: false
---

# Term
- w : 실행을 기다린 시간
- s : 실행 완료까지 필요한 cpu 시간
- e : 지금까지 cpu 사용한 시간 

# FIFO / first in first out

- Selection funciton
  - max[w]
- Decision mode
  - Non preemptive
    - 작업이 끝나거나, I/O 요청시 다음 프로세서 실행
먼저 큐에 온 순서대로 실행한다. 

장점
- 간단하다. 
- 오버헤드가 없다. 

단점
- convoy 효과가 있다. 
  - 긴 job이 앞에 오면 짧은 job이 반환 시간이 긴 job에 영향을 받는다.  
- 프로세서를 많이 사용하는 job이 유리하다. 
  - I/O를 자주 사용하는 잡은 뒤로 밀리게된다.
  - I/O utilization에 좋지 않다.  


# SPN / Shortest Process Next

- Selection function
  - min[s]
- Decision mode
  - Non preemptive

짧은 job이 먼저 수행된다면 반환시간을 줄일 수 있다. 
> 도착시간이 같을 때 SPN은 최적이다. 

장점
- 개선된 반환시간
- convoy 완화
  - 그러나 긴 job이 수행 중일때 짧은 job이 오면 convoy again

단점
- starvation
  - 긴 job이 영원히 실행 안될 수 있다. 
- 실행시간을 예측하기란 쉽지 않다. 
  - 누적 평균을 이용해 이전 데이터를 가지고 예측한다. 


# SRT / Shortest Remaining Time

- Selection function
  - min[s-e]
- Decision mode
  - Preemptive
    - 새로운 job이 도착하면 선점 판단

필요한 수행 시간에서 실행한 시간을 빼 기준으로 사용한다. 

장점
- 반환시간이 spn보다 좋다.
  - 기존에는 도착시간이 같을때 최적이었지만, 이제는 달라도 된다. 

단점
- 선점으로 인한 오버헤드
  - switch 빈번
- starvation
  - 긴 job이 손해를 본다. 
   실행 시간 예측 어려움

# HRRN / Highest Response Ration Next

- Selection function
  - min[(w+s) / s]
- Decision mode
  - Non preemptive

기아 현상을 해결해보자!

오래 대기한 프로세서들 보상하기 위한 기준
- w가 크면 우선시
- s가 작으면 우선시
  - 경쟁하게됨

장점
- 긴 job들도 짧은 job들과 경쟁해서 수행 가능

단점
- 수행 시간 예측 어떻게 할건데?
- 긴 job이 먼저 사용하고 있으면 convoy again

# Round-Robin

- Selection function
  - contant / fifo
- Decision mode
  - preemptive
    - 주어진 시간을 다 사용하면 다음 프로세스를 실행

반응시간을 짧게하기 위해서
기존 FIFO에서 짧은 job에게 어드밴티지를 주기위해서

> 타임 슬라이스가 커지면 반응 시간은 작아진다. 
> 반환 시간은 늘어진다. 

각 프로세스는 (n-1)*q 초과의 시간을 기다리지 않는다. 
> q : time slice
> 오버헤드 무시

장점 
- 짧은 반응 시간
- starvation x
- I/O utilization 
  - I/O가 쉬지 않게 바로 굴려줄 수 있다.

단점
- cpu bound 된 job은 시간을 다 사용하나 I/O bound된 job은 시간을 다 못씀
- 반환 시간이 느려진다. 

### 타임 슬라이스의 선택

- 최소 interaction 시간보다는 충분해야 한다. 
  - 하나의 interaction을 두 번에 나누면 현저히 느려진다. 
- 오버헤드를 고려해야 한다. 
  - 스왑 시간보다는 길어야한다. 
- 긴 작업은 긴 시간 슬라이스가 유리하고 짧은 작업은 작은 슬라이스가 유리하다. 


# VRR / virtual Round Robin

I/O 작업을 하는 프로세스들에게 어드밴티지를 준다. 
- 타임슬라이스를 다 사용하지 못한 프로세스는 임시 큐로 이동한다.  
- 임시 큐에 있는 프로세스가 먼저 실행된다. 이때 남은 시간씩만 수행 가능하다.

# Multi-Level Feedback Queues

- 큐가 여러개 존재한다. 
- 타임 슬라이스를 다 사용하면 낮은 단계의 큐로 강등당한다. 
- 높은 단계의 큐에는 짧은 작업이 남게된다.
- 높은 큐부터 수행해주며 큐가 비면 다음 단계의 큐를 수행시킨다. 
- 큐가 내려갈 수록 타임 슬라이스가 보통 2배가 된다.
- 긴 job은 기아 현상을 겪는다. 
  -  이를 위해 일정 시간마다 위 큐로 승급시켜준다.