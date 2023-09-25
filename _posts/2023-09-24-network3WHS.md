---
title: tcp 3WHS in linux kernel 이것저것
author: highcloud100
date: 2023-09-24 22:06:00 +0900
categories:
  - CS
  - network
tags:
  - cs
  - network
  - socket
  - linux
render_with_liquid: false
---
## Socket 프로그래밍에서 3WHS는 어떻게 작동하는가?
---

- 혼자 삽질하다 좋은 글을 찾았다. 
- https://levelup.gitconnected.com/deep-dive-into-tcp-connection-establishment-process-f6cfb7b4e8e1

- https://stackoverflow.com/questions/63232891/confusion-about-syn-queue-and-accept-queue

### ack 이 도착
1. listen 소켓이 있는지 찾아본다. 
	1. [looked up](https://github.com/torvalds/linux/blob/4d6d4c7f541d7027beed4fb86eb2c451bd8d6fff/net/ipv4/tcp_ipv4.c#L2017)
	2. [__inet_lookup](https://github.com/torvalds/linux/blob/4d6d4c7f541d7027beed4fb86eb2c451bd8d6fff/include/net/inet_hashtables.h#L392)
	3. [__inet_lookup_established](https://github.com/torvalds/linux/blob/4d6d4c7f541d7027beed4fb86eb2c451bd8d6fff/net/ipv4/inet_hashtables.c#L471)
		1. [ehash](https://github.com/torvalds/linux/blob/4d6d4c7f541d7027beed4fb86eb2c451bd8d6fff/net/ipv4/inet_hashtables.c#L486)를 [look up](https://github.com/torvalds/linux/blob/4d6d4c7f541d7027beed4fb86eb2c451bd8d6fff/net/ipv4/inet_hashtables.c#L484-L500)한다.
2. 일시적인 request_sock을 sock으로 cast
	1. TCP_NEW_SYN_RECV 상태이기에 
	2. [이 if문으로 분기](https://github.com/torvalds/linux/blob/4d6d4c7f541d7027beed4fb86eb2c451bd8d6fff/net/ipv4/tcp_ipv4.c#L2027)
	3. [tcp_check_req](https://github.com/torvalds/linux/blob/4d6d4c7f541d7027beed4fb86eb2c451bd8d6fff/net/ipv4/tcp_ipv4.c#L2070) 가 child socket만듬 ( cloning listening socket)
	4. [TCP_SYN_RECV](https://github.com/torvalds/linux/blob/f1fcbaa18b28dec10281551dfe6ed3a3ed80e3d6/net/ipv4/inet_connection_sock.c#L1145)상태 
		1. 이후 __inet_lookup_skb가 찾을 수 있도록 [ehash에 넣어준다.](https://github.com/torvalds/linux/blob/f1fcbaa18b28dec10281551dfe6ed3a3ed80e3d6/net/ipv4/tcp_ipv4.c#L1630) 
	5. [mini socket](https://github.com/torvalds/linux/blob/4d6d4c7f541d7027beed4fb86eb2c451bd8d6fff/net/ipv4/inet_hashtables.c#L662)ehash에서 삭제, normal 소켓이 ehash에 추가됐기에...
3. 

## 삽질
---
- listen은 단순히 listen상태로 만들어준다. 
- 이후 connect 요청이 오면 대기 큐에 어떻게 관리가 되는가?
- accept에서는 큐에서 어떻게 가져와 사용하는가? ( 마지막 ack의 처리는 어떻게? )
![](/assets/img/Pasted%20image%2020230924224011.png)
- tcp_v4_rcv 에서 -> tcp_v4_do_rcv를 불러서 tcp요청 처리

![](/assets/img/Pasted%20image%2020230924224452.png)
- tcp_rcv_state_process가 불림


![](/assets/img/Pasted%20image%2020230924220650.png)
- listen 상태일 때 
- /*
 *	This function implements the receiving procedure of RFC 793 for
 *	all states except ESTABLISHED and TIME_WAIT.
 *	It's called from both tcp_v4_rcv and tcp_v6_rcv and should be
 *	address independent.
 */


![](/assets/img/Pasted%20image%2020230924220738.png)

- conn_request 함수를 부름
	- 구조체에 정의되어 있음

![](/assets/img/Pasted%20image%2020230924221006.png)

- 구조체는 ipv4_specific으로 구현

![](/assets/img/Pasted%20image%2020230924221107.png)
- tcp_v4_conn_request 라는 함수로 맵핑됨 ( tcp_ipv4.c) 
![](/assets/img/Pasted%20image%2020230924221242.png)
- tcp_conn_request로 움겨감
	- tcp_input.c 에 구현됨
![](/assets/img/Pasted%20image%2020230924221500.png)
- fastopen 은 tfo 방법을 위한 것 같다.
- else 구문을 보면 
	- syn ack 보내는 것 확인 가능

- accept은 다음 함수에서 처리

![](/assets/img/Pasted%20image%2020230924230552.png)
![](/assets/img/Pasted%20image%2020230924230625.png)

- 그림에는 없지만 reqsk_put 함수가 accept 마지막에 불린다. 
	- req을 free해준다. 
	- 다만 tfo 설정이면 null 값으로 만들어 살려주고 reqsk_fastopen_remove가 지우는 것 같다.

https://charsyam.wordpress.com/2018/01/09/%ec%9e%85-%ea%b0%9c%eb%b0%9c-ipv4-tcp-socket-listen-%ec%97%90%ec%84%9c-accept-%ea%b9%8c%ec%a7%80/


Yes, the 3-way handshake can be performed at any time after `listen()` exits. But how the handshake is managed depends on the kernel. When a new connection arrives, some kernels complete the 3-way handshake immediately and put the connection into a separate queue for `accept()` to pull from. Some kernels only perform a partial handshake and then finish it only when `accept()` is actually called. 

– [Remy Lebeau](https://stackoverflow.com/users/65863/remy-lebeau "556,824 reputation")

 [Jan 12, 2021 at 19:20](https://stackoverflow.com/questions/65689312/why-does-accept-block-when-listen-is-the-very-first-involved-in-tcp#comment116145241_65690315)


- 이걸로 어느 정도 의문이 해결됨
	- 백로그 큐가 두 개로 관리된다.
	- incomplete connection queue
	-  complete connection queue
https://notes.shichao.io/unp/ch4/ <- 책이 좋긴하다
https://levelup.gitconnected.com/deep-dive-into-tcp-connection-establishment-process-f6cfb7b4e8e1


![](/assets/img/Pasted%20image%2020230925094017.png)
- inet_csk_accept 함수에서 가져오는 큐의 struct 정의 이다. 
	- established 된 소켓들을 가지고 있는다. 
