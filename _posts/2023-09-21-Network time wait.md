---
title: Network time wait
author: highcloud100
date: 2023-09-21 22:13:00 +0900
categories:
  - CS
  - network
tags:
  - time_wait
  - socket
render_with_liquid: false
---
https://tech.kakao.com/2016/04/21/closewait-timewait/

![](/assets/img/Pasted%20image%2020230921224342.png)

- 다음 상황은 hserver에서 소켓을 열고 hclient에서 접속한 뒤,hclient는 close, hserver는 close하고 대기하는 장면이다.  
- 오른쪽 위를 보면 tcp 소켓이 fd에 저장되어 있다가, close 이후 삭제된 모습이다. 
- 하지만 netstat -tonp 명령어를 보면 10223 포트의 소켓은 time waitg하는 상태이다. 
	- 커널로 넘어간 것인가?


![](/assets/img/Pasted%20image%2020230921232821.png)

- tcp 상태 = 소켓의 상태
	- 어쩌면 당연한 거
	- 포트의 상태 아니다
		- 포트가 사용되고 있을 뿐이다.

![](/assets/img/Pasted%20image%2020230921233427.png)

-  netstat -antplFo-> tcp 소켓 정보 모두 출력
	- 10777 포트를 가지는 소켓 3개 존재

- 한쪽 client가 close를 실
![](/assets/img/Pasted%20image%2020230921233627.png)



![](/assets/img/Pasted%20image%2020230922000344.png)
- 서버에서 accept를 받은 뒤, 먼저 close를 불러 time wait 상태로 보내버림  
	- 다만 이후 새로운 연결을 만들 수 있었다.
	- 포트 밴 ? 
		- 소켓이 닫히는데 시간이 필요한 것이다. 
		- 다른 프로세스가 10888포트를 요구한다면 당연히 에러 생김
		- 다만 같은 프로세스에서 바로 재 연결을 한다면? 

![](/assets/img/Pasted%20image%2020230922093825.png)
- 클라이언트에서 10888서버에 연결하고 close를 불러 연결을 종료하고, 다시 연결한 상태이다. 
	- 기존 소켓이 time wait에 걸려 새로운 포트를 받은 것을 볼 수 있다. 