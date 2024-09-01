---
title: XV6 OS Overview
author: highcloud100
date: 2023-08-29 22:42:00 +0900
categories:
  - mit6.1810
  - OS
  - xv6
tags:
  - os
  - xv6
  - mit6.1810
render_with_liquid: false
---

https://pdos.csail.mit.edu/6.828/2022/lec/l-overview.txt

# Overview

- O/S 디자인과 구현에 대한 이해를 목표함

### OS의 목적이 무엇인가?

- 하드웨어를 추상화하여 편리하고 이식성있게 만듬
- 여러 어플리케이션간  하드웨어 다중화
- 버그 가질 수도 있는 어플리케이션을 고립시킴
- 보안을 위한 제어 공유
- 어플리케이션간 공유 허용
- 여러 어플 지원

### OS 커널이 일반적으로 제공하는 것들

- 프로세스
- 메모리 할당
- 파일 , 디렉토리
- 접근 권한 ( 보안 )
- IPC, network, time, terminals ...

### App과 커널 간 인터페이스는 무엇인가?

- 시스템 콜
```c
fd = open("out", 1);
write(fd, "hello\n", 6);
pid = fork();
```


### OS 디자인과 구현이 어렵고 흥미로운 이유?

- 여러 등가 교환이 있다.
	- 효율 vs 추상화/이식성/범용성
	- Powerful vs 간단한 인터페이스
	- 유연성 vs 보안
- 기능 간 상호 작용
- 여러 곳에서 사용

## Introduction to UNIX system calls

- App은 시스템 콜을 통해 OS와 연결된다.
	- 첫번째 과제에서 쓰게됨
- xv6에서 예제를 보여주겠다.
	- xv6는 간단한 UNIX 시스템과 비슷한 구조를 가지고 있다.
		-  간단하여 대부분 책에 설명이 되어있다.
	- 왜 UNIX인가?
		- 오픈소스다. 문서화 잘되어있다.
		- 깔끔한 디자인과 널리 사용되고 있다.
	- xv6는 RISC-V 에서 돌아간다.

### Ex) copy.c : copy input to output

```c
// copy.c: copy input to output.

#include "kernel/types.h"
#include "user/user.h"

int
main()
{
  char buf[64];

  while(1){
    int n = read(0, buf, sizeof(buf));
    if(n <= 0)
      break;
    write(1, buf, n);
  }

  exit(0);
}
```

- input에서 바이트를 읽어 output에 쓴다.
- read( ) , write( )는 시스템 콜이다.
	- 두 함수의 첫 인자는 FD (file descriptor)이다.
		- 커널은 FD로 어떤 열린 파일에 쓸지 판단함
			- read, write는 열린 파일에만 가능하다.
			- FD는 파일/디바이스/소켓 등에 연결됨
			- UNIX 컨벤션
				- 0 : standard input
				- 1 : standard output
	- Read의 두 번째 인자는 포인터이다. 
		- 읽을 메모리의 위치
	- 세번째 인자는 읽을 크기이다.
	- 반환 값은 읽은 크기 or -1 (for error)
- FD는 어디서 온 것인가?

### Ex) open.c : create a file

```c
// open.c: create a file, write to it.

#include "kernel/types.h"
#include "user/user.h"
#include "kernel/fcntl.h"

int
main()
{
  int fd = open("out", O_WRONLY | O_CREATE | O_TRUNC);
  write(fd, "ooo\n", 4);

  exit(0);
}
```

- open( )은 파일을 만들고 FD를 반환한다.
- FD는 작은 숫자이다.
- FD는 프로세스별 테이블에 인덱싱되어 있다. 이는 커널이 관리한다.
- 즉 다른 프로세스는 다른 FD name-spaces를 가진다.

### open( ) 같은 시스템 콜을 부르면 무슨 일이 벌어지나요?

- 함수 콜로 보이지만 사실 특별한 명령이다. 

1. 하드웨어는 사용자 레지스터 값들을 저장한다.
2. 하드웨어는 접근 권한을 높인다. 
3. 하드웨어는 예약된 entry point 로 점프한다. ( 커널 안에 있음 )
4. 커널 안에서 C code가 돌아가는 상태가 됨. 
5. 커널은 시스템 콜의 구현을 부른다.
	1. sys_open( )은 파일 시스템의 이름을 살펴본다.
	2. disk 기다림
	3. 커널의 자료구조 수정 ( 파일 블럭 캐시, FD table )
6. 유저 레지스터를 복구한다.
7. 접근 권한을 낮춘다.
8. 프로그램의 calling point로 돌아가 재개한다. 

### shell

- command line interface이다.  
- shell은 "$"를 프린트한다. 
- shell은 UNIX command-line Utilities를 돌리게 해준다.
	- ls, ls > out, grep x  < out ...

### Ex) fork.c : create a new process

- shell은 새로운 프로세스를 사용자가 타이핑한 명령마다 만든다.
- fork( ) 시스템콜이 프로세스를 만든다. 
- 커널은 프로세스의 복제본을 만든다.
	- 명령어, 데이터, 레지스터, FD, 현재 디렉토리,,,
- 유일하게 다른 것은 fork가 반환하는 값이다.
	- pid 는 부모에게
	- 0 은 자식에게 
	- 즉 프로세스가 복제되고 fork 시점에서 두 프로세스 모두 실행됨

### Ex) exec.c : replace calling process with an executable file

```shell
$ echo a b c
```
- shell은 어떻게 프로그램을 돌리나? 
	- 프로그램은 파일에 저장되어 있다. 
		- 링커와 컴파일러에 의해 만들어짐
	- echo라 불리는 파일이 저장되어 있음 
	- exec( )은 현재 프로세스를 실행 가능한 파일로 교체함
		- 기존 명령과 메모리를 버리고
		- 새로운 명령과 메모리를 file로부터 읽어옴
		- 다만 FD는 유지된다.
	- exec ( filename, argument array )
 
### Ex) forkexec.c : fork() a new process, exec() a program

```c 
  
#include "kernel/types.h"
#include "user/user.h"

// forkexec.c: fork then exec

int
main()
{
  int pid, status;

  pid = fork();
  if(pid == 0){
    char *argv[] = { "echo", "THIS", "IS", "ECHO", 0 };
    exec("echo", argv);
    printf("exec failed!\n");
    exit(1);
  } else {
    printf("parent waiting\n");
    wait(&status);
    printf("the child exited with status %d\n", status);
  }

  exit(0);
}
```

 - fork는 복사하고 exec은 복사된 메모리를 날린다. 
	 - 이는 새로운 프로그램을 돌리는 방법이지만 낭비가 심하다.
	 - copy-on-write lab에서 해결할 것이다.

### Ex) redirect.c : redirect the output of a command

```shell
$echo hello > out
```

- shell은 위 명령에 대해 무슨 행동을 할까?
	- fork , 자식의 FD 1을 바꾼다. 
	- exec echo
- open( )은 항상 사용되지 않은 가장 낮은 FD를 사용한다.
- fork와 exec의 분리는 exec 직전에 자식의 FD를 바꿀 수 있게 만들어준다.
	- exec은 FD를 유지한다.
	- 명령어는 단순히 FD 0과 1을 사용할 뿐, 무엇이 0과 1에 연결되었는지 상관 안 쓴다.
- 고로 shell은 프로그램에 대해 신경 쓸 필요 없이 I/O redirection이 가능하다. 

### Ex) pipe1.c : communicate through a pipe

```c
// pipe1.c: communication over a pipe

#include "kernel/types.h"
#include "user/user.h"

int
main()
{
  int fds[2];
  char buf[100];
  int n;

  // create a pipe, with two FDs in fds[0], fds[1].
  pipe(fds);
  
  write(fds[1], "this is pipe1\n", 14);
  n = read(fds[0], buf, sizeof(buf));

  write(1, buf, n);

  exit(0);
}
```

- 그렇다면 위의 redirect 명령을 어떻게 구현했을까?
	- pipe( )를 이용한다.
	- pipe 시스템 콜은 두개의 FD를 만든다.
		- 첫번째 - 읽기전용
		- 두번째 - 쓰기전용
	- 커널은 각 파이프들을 위한 버퍼를 유지한다.


### Ex) pipe2.c : communicate between processes

```c
#include "kernel/types.h"
#include "user/user.h"

// pipe2.c: communication between two processes

int
main()
{
  int n, pid;
  int fds[2];
  char buf[100];
  
  // create a pipe, with two FDs in fds[0], fds[1].
  pipe(fds);

  pid = fork();
  if (pid == 0) {
    write(fds[1], "this is pipe2\n", 14);
  } else {
    n = read(fds[0], buf, sizeof(buf));
    write(1, buf, n);
  }

  exit(0);
}
```

```shell
$ ls  | grep x
```
- pipe는 fork와 함께 이용된다. 
	- shell은 pipe를 만들고 fork한다.
	- ls의 FD 1번을 pipe의 write로 연결한다.
	- grep의 FD 0번을 pipe의 read로 바꾼다.
		- 위의 코드는 standard input, output을 바꾸지 않는 간단한 코드이다.
		- standard i/o를 바꾸면 프로그램의 수정 없이 redirect 가능하다. 