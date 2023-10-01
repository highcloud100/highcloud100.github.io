---
title: Flow Control in TCP
author: highcloud100
date: 2023-09-30 17:24:00 +0900
categories:
  - CS
  - network
tags:
  - cs
  - network
  - tcp
render_with_liquid: false
---
reference : Tcp/IP Protocol Suite/Forouzan, Behrouz A.
# Flow Control
---
- flow control balances 
	- the rate  a producer creates data with
	- the rate a consumer can use the data
- TCP seperates flow control from error control
	- we assume that tcp is error free

![](/assets/img/Pasted%20image%2020230930172802.png)

- flow control feedback
	- receiving TCP -> sending TCP -> sending process
	- most implements of TCP don't provide flow control feedback from the receving process to receiving TCP

- sending TCP -> sending process
	- achieved through simple rejection of data by sending TCP
		- when its window is full

- so we will focus on  "receiving TCP -> sending TCP"

## Opening and Closing window
---
- TCP forces the sender and receiver to adjust their window sizes
- receive window
	- close : more bytes arrive from the sender
	- open : more bytes are pulled by the process
- send window
	- close : when new ack allows it to do so
	- open : when rwnd(from revceiver) aloow it to do so
		- receive window size

![](/assets/img/Pasted%20image%2020230930194404.png)

## Shrinking of window
---
- send window can shrink if the receiver defines a value for rwnd
	- but receive window can't shrink
- some implementations don't allow the shrinking of the send window
	- so the receiver needs to keep the following relationship

$$\text{new ackNO} +  \text{new rwnd} \ge \text{last ackNO} +  \text{last rwnd}$$

- so does not allow the right wall ofr the send window to move to the left
	- the left side of the inequality represents theh new position of the right wall
	- the right side shows the old position of the right wall

- if you violate this mandate, it causes the following problem

$$210 \text{ (new ack) } + 4 \text{ (new rwnd) } < 206 \text{ (last ack) } + 12 \text{ (last rwnd) }$$

![](/assets/img/Pasted%20image%2020230930202149.png)

> ackNo = 216 seems to be typo

- sender sent bytes 206~214
- 206~209 are acked and purged but new rwnd value is 4.
	- shrink!
- byte 214 has been already sent is outside the window
	- the receiver  does not know which of the bytes 210 to 217 has already been sent
- one way to prevent this situation 
	- receiver should wait until more bytes are consumed by its process
	- to meet the relationship above

## Silly Window Syndrome
---
- this problem can arise in the sliding window op when either
	- sending application creates data slowly
	- receiving application consumes data slowly
	- or both
- if TCP sends segments containing only 1byte of data
	- it needs 41byte datagram
		- 20bytes - tcp header
		- 20bytes - ip header ...
	- overhead 41/1
- The inefficiency is even worse after accounting for the data link, physical layer overhead

### Syndrome created by the sender
---
- when the sender creates data slowly, it cause the silly window syndrome
- The solution is to prevent the sending TCP from sending the data byte by byte
	- TCP must be forced to wait and collect data to send in a larger block
	- How long wait?
		- too long -> delay system
		- not enough -> syndrome again

#### Nagle's Algorithm
---  
1. TCP send the first piece of data it received from the app 
	1. even if it is only 1 byte
2. After sending the first segment, TCP accumulates data in the output buffer.
	1. wait until
		1. receiving ACK for first segment
		2. enough data has accumulated to fill maximum size segment
3. repeat


### Syndrome create by the receiver
---
- Suppose 
	- the sending program create data in blocks of 1kb
	- receving program consumes data 1byte at a time
	- receving buffer of reveving TCP is 4kb

- the sender sends the 4kb of data 
- the receving buffer is full
	- it advertises a window size of zero
	- sender should stop
- the 1 byte consumed by receiver
	- now there is 1 byte of space in the receving buffer
	- announce window size of 1 byte
- sending TCP which is eagerly waiting to send data sends a segment carrying 1byte
- silly window syndrome again...

#### Clark's Solution
---
- send an ack as soon as the data arrive
	- but set rwnd to "zero"
	- untile either there is enough space to accommodate a seg of max size or least half of the receive buffers is empty

#### Delayed Ack
---
- delay sending the ack 
	- when a segment arrives, not ack immediately
- wait until there is a decent amount of space in its incoming buffer
	- delaying ack prevents the sending TCP from sliding its window
- also has another advantage
	- reduces traffic
- however there also disadvantage
	- sender unnecessarily retransmitting the unacknowledged segment
- TCP balances the adv, disadv
	- should not be delayed by more than 500ms