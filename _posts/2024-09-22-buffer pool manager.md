---
title: Buffer Pool Manager 과제
author: highcloud100
date: 2024-09-22 23:21:00 +0900
categories:
  - CS
  - DB
tags: 
math: true
mermaid: true
render_with_liquid: false
---

## 0. bustup vscode setting

우분투 환경은 기본적으로 gcc, g++이 잡혀있다. 
이를 clang14로 바꾸어야한다.

```shell
// clang-14 선택해준다. 
sudo update-alternatives --config c++ 
sudo update-alternatives --config cc

// make시에 simple test 안되면
sudo apt install libstdc++-12-dev 
```

https://stackoverflow.com/questions/26380407/cmake-clang-is-not-able-compile-a-simple-test-program-fedora-20

vscode extension에서 clangd를 설치해 사용한다. 
이때 include 폴더가 따로 위치하기에, 컴파일 db 정보를 만들어서 컴파일러가 확인하게 만들어야한다. 

https://clang.llvm.org/docs/JSONCompilationDatabase.html

[bear](https://github.com/rizsotto/Bear)를 이용해 compile_commands.json을 만들고, 프로젝트 루트 디렉토리로 이동시킨다. 

```
cd /build
bear -- make
mv compile_commands.json ~/bustub
```

## 1. 구현 

구현은 3 파트로 나뉜다. 
- replacer
	- decide which page to evict
- disk scheduler
	- execute disk i/o
- buffer pool manager
	- interface of pages and frames
 ![](/assets/img/Pasted%20image%2020241003162522.png)

## 2. Replacer

CS에서 작은 공간에 모든 데이터를 담아둘 수 없을 때, 늘 나오는 주제이다. 
핵심은 공간과 시간 요소를 활용하여 최대한 하위 레이어로 내려가지 않도록 만드는 것이다. 대표적인 알고리즘으로 LRU나 clock 방법이 있다. 

이런 알고리즘을 선택할 때는 접근 패턴을 고려해야 한다.
데이터베이스에서 특히 OLAP 작업 패턴에서는 순차적으로 scan하는 작업을 많이 하게 된다. 이 경우 LRU나 clock을 사용하면 sequential flooding 문제를 야기할 수 있다. 

> https://stackoverflow.com/questions/20464198/what-is-sequential-flooding

> most recently used page is often the best page to evict.

이는 LRU나 clock이 last access 정보만 관리하기 때문이다. 
얼마나 자주 접근 되었는지는 따지지 않는다. 

이런 문제를 다루는 여러 방법이 있지만, 과제에서는 LRU-K를 구현한다. 

> https://www.cs.cmu.edu/~natassa/courses/15-721/papers/p297-o_neil.pdf

time stamp를 사용해 page에 접근 할 때마다, time stamp를 기록한다.  
이때 page마다 K 개의 history까지 누적한다. 
쫓아낼 후보를 결정할 때는 $\text{current time stamp} - \text{k th time stamp}$ 가 가장 큰 page로 정한다. 
만약 history의 개수가 k개보다 작다면 (적어도 k번 참조되지 않음) 무한대의 값으로 계산한다. 
tie가 발생하면 최신의 time stamp가 가장 오래된 page를 쫓아낸다.  

그대로 구현해준다. 
- 중요한 점은 pin되어 있는 page는 쫓아내면 안된다는 점이다. 

![](/assets/img/Pasted%20image%2020241003193726.png)

여담
- DBMS는 query execution의 context를 알기 때문에, 페이지 우선순위에 대한 힌트를 제공할 수 있다.  
- mysql은 old, young list를 사용해 LRU-K를 근사하여 사용한다. 
	- 핵심은 순간의 접근인지, 빈번한 접근인지를 가려내기 위함이다. 
## 3. Disk scheduler 

최적화를 할 부분이 많겠지만, 간단히 fifo queue로 구현하였다. 

- Sequential vs Random I/O
- Critical Path Task vs Background Task
- Table vs Index vs Log vs Ephemeral Data
- Transation Information
- User-Based SLAs

OS는 이런 걸 알기 어렵다. 고로 최대한 DBMS가 다루어야 한다.
OS는 file cache(page, buffer cache)를 가지고 있다.
대부분의 DBMS는 이를 bypass한다. (O_DIRECT)
- redundant copies of pages
- different eviction policies.
- loss of control over file I/O

> DIRECT IO in PostgreSQL and double buffering
> https://www.linkedin.com/posts/krishnakumar-r-bb7b949_postgres-postgresql-kernel-activity-7191224981924552705-i-7R?utm_source=share&utm_medium=member_desktop

OS 관련한 문제로는 fsync가 있다. 
fsync를 실패했을 때, linux는 dirty page를 clean하다고 mark한다.
fsync 재 호출시 flush가 정상적으로 되었다고 반환하지만, 실제로는 아닐 수 있다. 
> https://wiki.postgresql.org/wiki/Fsync_Errors  
> https://learn.microsoft.com/ko-kr/archive/blogs/bobsql/sql-server-linux-fsync-and-buffered-i-o

![](/assets/img/Pasted%20image%2020241003195656.png)

## 4. Buffer Pool Manager

DB 전체의 흐름은 다음과 같다. 
![](/assets/img/Pasted%20image%2020241003195856.png)

buffer pool manager에서는 page table을 관리해주어야 한다. 
- page id와 framed idx 간의 연결이다.

중요한 점은 latch와 pin의 처리이다. 

누군가 write을 하고 있다면, 그 페이지에 대한 다른 접근은 block되어야 한다.
반대로 read 작업은 여러 곳에서 병렬적으로 이루어질 수 있다. 

Pin은 page가 사용 중인지 알려준다. 즉 Pin 상태라면 evict해서는 안된다. 

> Latch는 OS의 mutex이다. 
> Lock은 다른 개념에서 사용된다. 


![](/assets/img/Pasted%20image%2020241003200110.png)

구현은 latch를 어떻게 사용할지 생각해보고 접근하면 된다. 
실수했던 점은 frame id를 찾고서,  해당 frame이 잠겨 있다면 block하게 만들었다. 
이 문제를 인지 못해 어지러운 디버깅 과정을 겪을 수 있었다. 

> 테스트는 통과하는데, 벤치마크 10초 돌리니까 간혹 consistency가 깨졌다.

핵심은 frame의 latch를 잡았을 때, 그 frame idx가 기존의 page와 맵핑된 상태가 아닐 수 도 있다. 즉 다시 page table을 참조해야 했다. 

최적화는 하지 않았다.

![](/assets/img/Pasted%20image%2020241003200955.png)![](/assets/img/Pasted%20image%2020241003201147.png)

## 5. mmap I/O problems

OS를 최대한 사용하지 말아야한다.
다음의 문제가 있다. 

- transaction safety
	- OS can flush dirty pages at any time
- I/O stalls
	- DBMS는 어떤 page가 메모리에 있는지 모른다. 
	- OS는 page fault시 thread를 stall한다.
- Error Handling 
	- Difficult to validate pages.
	- Any access can cause a SIGBUS 
- Performance Issues
	- OS는 DB를 목적으로 만들어지지 않았다.

madvise, mlock, msync등을 이용하여 문제를 완화할 수 있다. 
되도록 DBMS는 온전히 자신이 control하기를 바란다. 

>https://db.cs.cmu.edu/mmap-cidr2022/

## 6. Buffer Pool Optimization

- Multiple buffer pools
- Pre-fetching
- Scan sharing
	- synchronized scans
- Buffer Pool Bypass
	- Light scans
	- sequntial scan 같은 경우, buffer pool에 fetch를 안해버린다.
	- replacer를 어지럽히지 않음
