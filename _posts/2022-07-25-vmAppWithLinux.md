---
title: VM application with linux
author: highcloud
date: 2022-07-25 19:55:00 +0900
categories: [CS, OS]
tags: [os, cs, summer, 실습과 그림으로 배우는 리눅스 구조]
render_with_liquid: false
---

# 파일 맵 
---
- [MMF : 메모리 맵 파일](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EB%A6%AC_%EB%A7%B5_%ED%8C%8C%EC%9D%BC)이라 부른다.
- 파일을 다루는 방법 중 하나이다.
- 프로세스의 가상 메모리 주소 공간에 파일을 매핑하여 사용하는 방법이다.

<img width="500" alt="image" src="https://user-images.githubusercontent.com/80192345/180723991-5f7ff1c0-0632-4aed-8f8d-8c1c0f7cf987.png">

- mmap( ) 함수를 특정한 방법으로 호출하면 파일의 내용을 메모리에 읽어, 가상 주소 공간에 매핑 할 수 있다.
	- 입출력을 수행하지 않고, 메모리에 접근하듯 사용한다.
	- 수정한 내용은 이후 저장 장치 내의 파일에 써진다.


## 실험
---

1. 프로세스의 메모리 맵의 정보를 출력한다.
2. testfile을 연다
3. mmap( ) 으로 메모리 공간에 매핑한다.
4. 프로세스의 메모리 맵 정보 표시
5. 매핑된 영역의 데이터를 읽는다.
6. 매핑된 영역의 데이터를 덮어쓴다.

- 우선 testfile을 만든다.

```bash
echo hello >testfile
```

- 이후 코드를 짠다.

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <err.h>

#define BUFFER_SIZE     1000
#define ALLOC_SIZE      (100*1024*1024)

static char command[BUFFER_SIZE];
static char file_contents[BUFFER_SIZE];
static char overwrite_data[] = "HELLO";

int main(void)
{
        pid_t pid;

        pid = getpid();
        snprintf(command, BUFFER_SIZE, "cat /proc/%d/maps", pid);

        puts("*** memory map before mapping file ***");
        fflush(stdout);
        system(command);

        int fd;
        fd = open("testfile", O_RDWR);
        if (fd == -1)
                err(EXIT_FAILURE, "open() failed");

        char * file_contents;
        file_contents = mmap(NULL, ALLOC_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
        if (file_contents == (void *) -1) {
                warn("mmap() failed");
                goto close_file;
        }

        puts("");
        printf("*** succeeded to map file: address = %p; size = 0x%x ***\n",
                        file_contents, ALLOC_SIZE);

        puts("");
        puts("*** memory map after mapping file ***");
        fflush(stdout);
        system(command);

        puts("");
        printf("*** file contents before overwrite mapped region: %s", file_contents);

        memcpy(file_contents, overwrite_data, strlen(overwrite_data));

        puts("");
        printf("*** overwritten mapped region with: %s\n", file_contents);

        if (munmap(file_contents, ALLOC_SIZE) == -1)
                warn("munmap() failed");
close_file:
        if (close(fd) == -1)
                warn("close() failed");
        exit(EXIT_SUCCESS);
}
```

<img width="700" alt="image" src="https://user-images.githubusercontent.com/80192345/180725602-de6b9004-ba19-447b-9691-a12411f29b8c.png">

- 위의 결과를 확인 할 수 있다.
- 7f3fe6e55000-7f3fed255000 rw-s 00000000 08:05 8522195                    /home/we/projects/os/testfile 줄을 보면 매핑 되었음을 확인 할 수 있다.

<img width="265" alt="image" src="https://user-images.githubusercontent.com/80192345/180725899-9acdc4ba-e9ff-47c5-9eed-e34254159054.png">

- 덮어 써짐도 확인 가능하다.

- write, fprintf 함수를 쓰지 않고 메모리 영역에 memcpy로 복사만 해도 파일의 내용이 바뀐 것을 확인할 수 있다.


# 디맨드 페이징
---

- 기존에 설명한 메모리 할당 방법은 메모리 낭비 문제가 있다.
	1. 커널이 필요한 메모리를 확보하고
	2. 페이지 테이블을 설정하여 가상 주소 공간을 물리 주소에 매핑한다.

- 확보한 메모리 중에는 프로세스가 종료할 때까지 사용하지 않는 영역들이 있기 때문이다.
	- 프로그램의 사용하지 않는 코드와 데이터 영역
	- glibc가 확보한 메모리 맵 중 malloc으로 확보하지 않은 부분

- 이를 해결하기 위해 나온 방법이 디맨드 페이징이다.

## 페이지의 상태
---
- 기존의 페이지 상태에 3번째 상태를 추가한다.
	1. 프로세스에 할당되지 않음
	2. 프로세스에 할당되었고 물리 메모리에도 할당됨
	3. 프로세스에는 할당되었으나 물리 메모리에는 할당되지 않음

- 프로세스가 생성되면 코드 영역이나 데이터 영역에 프로세스가 영역을 얻었음을 표시한다.
	- 다만 물리 메모리는 할당되지 않았다.
- 엔트리 포인트로부터 실행이 시작되면 엔트리 포인트에 해당하는 페이지용 물리 메모리가 할당된다.

### 처리 흐름

1. 프로그램이 엔트리 포인트 접근
2. CPU가 테이블을 통해 가상 주소가 물리 주소에 아직 매핑되지 않음을 확인
3. CPU에 페이지 폴트 발생
4. 커널의 페이지 폴트 핸들러가 물리 메모리를 해당 페이지에 할당
	- 페이지 폴트 지워짐
5. 사용자 모드로 돌아와서 프로세스가 실행 계속함

> 프로세스는 페이지 폴트가 발생한지 모른다.


<img width="500" alt="image" src="https://user-images.githubusercontent.com/80192345/180751497-2f774090-999d-4897-b2db-e850f8a853fd.png">


- 프로세스가 mmap( ) 함수로 메모리를 확보하는 것을 "가상 메모리를 확보했음"이라 부른다.
	- 가상 메모리가 물리 메모리를 확보해 연결되면 "물리 메모리를 획득했음"이라 부른다.

- mmap 의 성공과 상관없이 물리 메모리를 확보하려 할 때 충분하지 않으면 물리 메모리 부족이 발생한다.  

## 실험 
---

### 프로그램의 흐름

1. 메모리 획득 전을 알림
2. 메모리 획득함
3. 메모리 획득 후를 알림
4. 10mb씩 접근함 -> 물리 메모리 할당

```c
#include <unistd.h>
#include <time.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <err.h>

#define BUFFER_SIZE     (100 * 1024 * 1024)
#define NCYCLE          10
#define PAGE_SIZE       4096

int main(void)
{
        char *p;
        time_t t;
        char *s;

        t = time(NULL);
        s = ctime(&t);
        printf("%.*s: before allocation, please press Enter key\n",
                        (int)(strlen(s) - 1), s);
        getchar();

        p = malloc(BUFFER_SIZE);
        if (p == NULL)
                err(EXIT_FAILURE, "malloc() failed");

        t = time(NULL);
        s = ctime(&t);
        printf("%.*s: allocated %dMB, please press Enter key\n",
                        (int)(strlen(s) - 1), s, BUFFER_SIZE / (1024 * 1024));
        getchar();

        int i;
        for (i = 0; i < BUFFER_SIZE; i += PAGE_SIZE) {
                p[i] = 0;
                int cycle = i / (BUFFER_SIZE / NCYCLE);
                if (cycle != 0 && i % (BUFFER_SIZE / NCYCLE) == 0) {
                        t = time(NULL);
                        s = ctime(&t);
                        printf("%.*s: touched %dMB\n",
                                        (int) (strlen(s) - 1), s, i / (1024*1024));
                        sleep(1);
                }
        }

        t = time(NULL);
        s = ctime(&t);
        printf("%.*s: touched %dMB, please press Enter key\n",
                        (int) (strlen(s) - 1), s, BUFFER_SIZE / (1024 * 1024));
        getchar();

        exit(EXIT_SUCCESS);
}
``` 

- 실행 시키면 다음과 같은 결과를 확인 할 수 있다.

<img width="422" alt="image" src="https://user-images.githubusercontent.com/80192345/180756450-a9524334-ae9b-4dc3-a12d-9c82c38c775b.png">

- 동시에 다른 터미널에서 확인해보면 

<img width="832" alt="image" src="https://user-images.githubusercontent.com/80192345/180756477-a5c76d68-d453-4ac6-9a94-3dc258f42287.png">

- 다음과 같은 결과를 확인 할 수 있다.
- 100mb가 할당 받은 상태에서는 kbmemused 값의 변화가 거의 없다
- 이후 10mb씩 접근하면 kbmemused 값이 10mb 씩 증가하는 모습을 볼 수 있다.
- 모든 접근이 종료되면 100mb 늘어난 것을 확인 할 수 있다.

> kbmemused : 시스템의 물리 메모리 사용량


# 가상/물리 메모리 부족
---
## 가상 메모리 부족
---
- 말 그대로 가상 메모리 공간을 모두 사용했을 때 추가 메모리를 요청하면 발생한다.
- 32비트 기준 4GB가 최대 공간인데 이를 넘어서면 발생한다.
- 물리 메모리의 상태와 상관이 없다.

## 물리 메모리 부족
---
- 진짜 물리 메모리를 다 쓴 경우이다.

# Copy on Write
---
- fork ( ) 시스템 콜을 가상 메모리를 사용하여 고속화한다.
- fork 가 발생하면 페이지 테이블만 복사한다.

> PCB에 페이지 테이블이 포함된다. 사실 PCB를 복사한다.

- 이후 페이지 테이블의 쓰기 권한을 자식과 부모 모두 무효화 한다.
	- 둘 다 쓰지 못한다.

- 이후 누군가 쓰려하면 다음의 흐름이 진행된다.

1. 페이지에 쓰기는 허용하지 않아 CPU 페이지 폴트가 발생
2. 커널의 페이지 폴트 핸들러 등장
3. 접근한 페이지를 다른 장소에 복사하고, 쓰려한 프로세스에 할당해준다.
4. 이후 내용을 작성한다.
5. 페이지 테이블 엔트리를 업데이트한다.
	- 공유 상황이 풀렸기에 쓰기를 둘 다 허용한다.

 
- 쓰기가 발생할 때 물리 메모리를 복사하므로 CoW라고 부른다.
	- fork가 성공해도 이후에 물리 페이지가 필요할 때 물리 메모리 부족이 발생한다.

<img width="647" alt="image" src="https://user-images.githubusercontent.com/80192345/180761014-89bc2ca4-6311-4a3e-a642-048485de02df.png">

