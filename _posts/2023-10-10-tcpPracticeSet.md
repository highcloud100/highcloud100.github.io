---
title: McgrawHill TCP, IP suit practice set
author: highcloud100
date: 2023-10-10 19:58:00 +0900
categories:
  - CS
  - network
tags:
  - cs
  - network
  - tcp
math: true
mermaid: true
render_with_liquid: false
---
---

- 머리도 아프고 시험 공부도 잘 안돼서 책에 연습문제를 풀어보려 한다.

---

####  An IP datagram is carrying a TCP segment destined for address 130.14.16.17. The destination port address is corrupted and it arrives at destination 130.14.16.19. How does the receiving TCP react to this error?


> checksum을 이용해 detect한다.

$$\text{checksum} = \text{pseudo header} + \text{tcp header + payload}$$

> 이기에 pseudo header에서 ip 값이 다른 것을 확인한다.

> 패킷은 버린다.

#### What is the maximum size of the TCP header? What is the minimum size of the TCP header?

> 헤더의 HLEN field는 4비트이다. 헤더 사이즈 % 4 값을 담는다. 이에 표현 가능한 범위는 20(default) ~ 60이다. 고로 60byte가 최대다.

#### If the value of HLEN is 0111, how many bytes of option are included in the segment?

> 7x4 = 28, default가 20byte이기에 옵션은 8byte

#### What can you say about the TCP segment in which the value of the control field is one of the following: a. 000000 b. 000001 c. 010001 d. 000100 e. 000010 f. 010010

> URG / ACK / PSH / RST / SYN / FIN

> a : 그냥 패킷 전송, ack 없음

> b : FIN 종료 요청

> c : FIN+ACK / server가 보통 passive close할 때 -> 3way handshake

> d : RST 요청

> e :  SYN 요청

> f : SYN + ACK

#### The following is a dump of a TCP header in hexadecimal format.

```

(0532_0017 0000_0001 0000_0000 5002_07FF 0000_0000)_16_

```

a. What is the source port number?

> 1330

b. What is the destination port number?

> 23

c. What the sequence number?

> 1

d. What is the acknowledgment number?

> 0

e. What is the length of the header?

> 5x4 = 20

f. What is the type of the segment?

> SYN

g. What is the window size?

> 2,047

#### To make the initial sequence number a random number, most systems start the counter at 1 during bootstrap and increment the counter by 64,000 every half second. How long does it take for the counter to wrap around?

> 2^32-1 = 4,294,967,295

> 1 + 6400 x Time = 4,294,967,295

> TIME / 2 = 335,544 s

#### In a TCP connection, the initial sequence number at the client site is 2,171. The client opens the connection, sends only one segment carrying 1,000 bytes of data, and closes the connection. What is the value of the sequence number in each of the following segments sent by the client?

a. The SYN segment?

>2171

b. The data segment?

>2172 / syn에서 1byte 소모

c. The FIN segment?

>3172 / 1000byte 보냈기에 그 다음 3172

#### In a connection, the value of cwnd is 3000 and the value of rwnd is 5000. The host has sent 2,000 bytes, which have not been acknowledged. How many more bytes can be sent?

> window = 3000

> 1000byte 가능 / 윈도우는 ack이 와야 왼쪽 벽이 오른쪽으로 이동한다.

#### TCP opens a connection using an initial sequence number (ISN) of 14,534. The other party opens the connection with an ISN of 21,732.

a. Show the three TCP segments during the connection establishment.

>SYN : 14534 / -

>SYN + ACK : 21732 / 14535

>ACK : - / 21733

b. Show the contents of the segments during the data transmission if the initiator sends a segment containing the message “Hello dear customer” and the other party answers with a segment containing “Hi there seller.”

> Hello-dear-customer = 19글자

> 19 x 1byte -> 20byte (1 byte padding 16비트의 배수로 만듬 )

> segment : 14535 / 21733

>

>Hi there seller = 15글자

>15byte -> 16byte ( 1byte padding )

>segment : 21733 / 14555

c. Show the contents of the segments during the connection termination.

> FIN : 14555 / 21749

> FIN + ACK : 21749 / 14556

> ACK : - / 21750

#### A TCP connection is using a window size of 10,000 bytes and the previous acknowledgment number was 22,001. It receives a segment with acknowledgment number 24,001 and window size advertisement of 12,000. Draw a diagram to show the situation of the window before and after.

![](/assets/img/Pasted%20image%2020231010220329.png)

#### A window holds bytes 2001 to 5000. The next byte to be sent is 3001. Draw a fig[1]ure to show the situation of the window after the following two events. a. An ACK segment with the acknowledgment number 2500 and window size advertisement 4000 is received. b. A segment carrying 1,000 bytes is sent.

![](/assets/img/Pasted%20image%2020231010220835.png)

#### A TCP connection is in the ESTABLISHED state. The following events occur one after another:

a. A FIN segment is received.

b. The application sends a “close” message.

What is the state of the connection after each event? What is the action after each event?

> a : CLOSE WAIT

> b : LAST ACK ( server ) , FIN-WAIT1 ( client )

#### A TCP connection is in the ESTABLISHED state. The following events occur one after another:

a. The application sends a “close” message.

b. An ACK segment is received.

What is the state of the connection after each event? What is the action after each event

> a : FIN-WAIT1

> b : FINE WAIT2

#### Show a congestion control diagram like Figure 15.37 using the following scenario. Assume a maximum window size of 64 segments. a. Three duplicate ACKs are received after the fourth RTT. b. A time-out occurs after the sixth RTT.

![](/assets/img/Pasted%20image%2020231010222844.png)