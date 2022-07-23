---
title: Memory management with linux
author: highcloud
date: 2022-07-23 18:30:00 +0900
categories: [CS, OS]
tags: [os, cs, summer, 실습과 그림으로 배우는 리눅스 구조]
render_with_liquid: false
---

# 메모리 관리
---
## 메모리의 통계 정보
---

- 시스템의 총 메모리의 양과 사용 중인 메모리의 양을 free 명령어로 볼 수 있다.

```shell
free 
```

![](https://user-images.githubusercontent.com/80192345/180595112-1c4c2cef-ab67-40af-908c-fdfb3061668f.png)

- Mem 줄의 중요한 필드를 알아보자.
	- total : 시스템에 탑재된 전체 메모리 용량
	- free : 표기상 이용하지 않는 메모리
	- buff / cache :  버퍼 캐시, 페이지 캐시가 사용하는 메모리다.
		- 빈 메모리가 부족하면 커널이 해제한다.
	- available : 실질적으로 사용 가능한 메모리다.
		- free 필드 값과 메모리가 부족하면 해제되는 커널 내의 메모리 영역 사이즈를 더한 값이다. 
	- swap : swap area의 크기이다.

![](https://user-images.githubusercontent.com/80192345/180595520-d18a49ff-8613-4184-adbb-dc01d4160b66.png )

- 다음 명령어를 통해 메모리 관련 통계 정보를 얻을 수 있다.

```shell
sar -r 1
```

![](https://user-images.githubusercontent.com/80192345/180595568-7d194b1b-1197-44c2-bd40-6d3c3740c871.png )

## 메모리 부족
---

- 메모리가 부족하면 해제 가능한 공간을 해제하여 메모리를 늘린다.
- 이후에도 계속 부족하면 OOM 상태가 된다.
	- out of memory
- 이때는 적절한 프로세스를 선택해 강제로 종료 시킨다.
	- OOM Killer 라는 기능이다. [참고](https://www.kernel.org/doc/gorman/html/understand/understand016.html)

> 서버에서 프로세스가 강제 종료가 되면 문제를 발생 시킨다.

> sysctl 의 'vm.panic_on_oom' 파라미터의 기본 값을 변경해 프로세스가 아닌 시스템을 강제 종료하는 방법도 있다.

![](https://user-images.githubusercontent.com/80192345/180595819-449ee8f7-4994-4b37-8863-5d4b100c31d4.png )

## 단순 메모리 할당
---
- 메모리를 할당하는 일은 두 타이밍에서 벌어진다.
	1. 프로세스를 생성할 때
	2. 프로세스를 생성한 뒤 추가로 동적 메모리를 할당할 때

- 동적 할당하는 부분을 이야기해보자.
	- 프로세스가 추가 메모리를 요청한다.
	- 메모리 확보용 시스템 콜을 호출한다.
	- 커널은 필요한 사이즈만큼 빈 메모리에서 가져와 시작 주소를 반환해준다.

### 문제점
- 이런 방식에는 문제점이 있다.
	1. 메모리 단편화
	2. 다른 용도의 메모리에 접근 가능
	3. 여러 프로세스를 다루기 곤란
		 - 프로세스마다 주소를 겹치지 않게 프로그래밍 해야 된다.

## 가상 메모리
---
- 위에서 언급된 여러 문제점을 해결하기 위한 방법이다.
- 개념적인 내용은 이전 글 참고

### Page fault
- 만약 물리 메모리에 매핑되어 있지 않은 주소에 접근하면 ?
	- page fault 라는 인터럽트가 발생한다.

- 실행 중인 명령이 중단되고 페이지 폴트 핸들러라는 인터럽트 핸들러가 동작한다.
- SIGSEGV 시그널을 프로세스에 통지하며, 프로세스는 강제 종료된다.

- 직접 이를 확인해보자

### 실험

```c
#include <stdio.h>
#include <stdlib.h>

int main(void){
        int *p = NULL;
        puts("before invalid access");
        *p=0;
        puts("after invalid access");
        exit(EXIT_SUCCESS);
}
```

![](https://user-images.githubusercontent.com/80192345/180598135-6d25b503-ff9e-4b7f-b377-b706ce861eea.png )


## 메모리 할당
---

- 프로세스를 생성할 때 실행 파일에서 여러 보조 정보를 얻는다.

이름 | 값
|-- | --|
코드 영역의 파일상 오프셋 | 100
코드 영역 사이즈 | 100
코드 영역의 메모리 맵 시작 주소 | 0
데이터 영역의 파일상 오프셋 | 200
데이터 영역 사이즈 | 200
데이터 영역의 메모리 맵 시작주소  | 100
엔트리 포인트 | 0

- 프로그램을 실행하는데 필요한 메모리 사이즈는 
	- "코드 영역 사이즈" + "데이터 영역 사이즈" 이다.
	- 즉 표를 기준으로 300이 필요하다.

### 추가적인 메모리 할당

- 새 메모리를 요구하면 커널은 새로운 메모리를 할당하여 대응하는 페이지 테이블을 작성하고, 할당된 물리 주소에 대응하는 가상 주소를 프로세스에 반환한다.

### 실험

```c
#include <unistd.h>
#include <sys/mman.h>
#include <stdio.h>
#include <stdlib.h>
#include <err.h>

#define BUFFER_SIZE 1000
#define ALLOC_SIZE (100*1024*1024)

static char command[BUFFER_SIZE];

int main(void){
        pid_t pid;
        pid = getpid();
        snprintf(command, BUFFER_SIZE, "cat /proc/%d/maps", pid);

        puts("___ memory map before memory allocation ___");
        fflush(stdout);
        system(command);

        void *new_memory;
        new_memory = mmap(NULL, ALLOC_SIZE, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1,0);
        if(new_memory == (void*) -1)
                err(EXIT_FAILURE, "mmap() failed");

        puts("");

        printf("___succeeded to  allocate memeory : address = %p; size = 0x%x ___\n", new_memory, ALLOC_SIZE);
        puts("");
        puts("___memory map after memory allocation___");
        fflush(stdout);
        system(command);

        if(munmap(new_memory, ALLOC_SIZE) == -1)
                err(EXIT_FAILURE, "MUNMAP() FAILED");
        exit(EXIT_SUCCESS);

}
```

![](https://user-images.githubusercontent.com/80192345/180598780-e22915d8-b810-407e-9e86-49268c956893.png )

- mmap( ) 함수는 리눅스 커널에 새로운 메모리를 요구하는 시스템 콜을 호출한다.
- system( ) 함수는 첫 파라미터에 지정된 명령어를 리눅스 시스템에서 실행한다.

- 중간에 출력된 문장을 보면 새로 할당된 메모리의 주소를 확인 할 수 있다.


![](https://user-images.githubusercontent.com/80192345/180598907-c2740c3d-01e4-4602-953b-c686f279a25f.png)

- 100MB 가 할당된 것을 확인 할 수 있다.


## 고수준 레벨에서의 메모리 할당
---

- C의 표준 라이브러리에 있는 malloc( ) 함수가 메모리 확보 함수이다.
	- 내부적으로는 mmap( ) 함수를 호출한다.

- mmap( ) 함수는 페이지 단위로 메모리를 확보한다.
	- malloc( ) 함수는 바이트 단위로 확보한다.

- glibc 에서는 사전에 mmap( )으로 메모리 풀을 만든다.
	- 이후 malloc( )이 호출되면 메모리 풀에서 일부를 반환한다.
	- 만약 부족하면 mmap( )으로 새로운 메모리를 확보한다.

