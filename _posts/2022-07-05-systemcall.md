---
title: System Call
author: highcloud
date: 2022-07-05 14:30:00 +0900
categories: [CS, OS]
tags: [os, cs, summer]
render_with_liquid: false
---

<h1 id="systemcall">SystemCall</h1>
<hr>
<ul>
<li>system call은 커널에게 I/O 작업을 요청하는 것이다.</li>
<li>하드웨어의 mode bit 를 사용하여 커널모드와 유저모드로 나눈다.
<ul>
<li>cpu의 mode bit 가 0이면 kernel mode 이며, cpu는 어떤 메모리의 영역이든 접근할 수 있다.</li>
<li>mode bit이 1이면 user mode로 cpu는 파일에 영향을 주는 명령은 사용이 불가능하다.</li>
</ul>
</li>
</ul>
<blockquote>
<p>특권 명령과 일반 명령이라도 부른다.</p>
</blockquote>
<p><img src="https://t1.daumcdn.net/cfile/tistory/9967A6385F6366A322?original" alt=""><br>
<img src="https://t1.daumcdn.net/cfile/tistory/993592335F6366A328?original" alt=""><br>
<img src="https://t1.daumcdn.net/cfile/tistory/9926293C5F6366A308?original" alt=""></p>
<ul>
<li>mode bit을 검사하는 과정을 알아보자.
<ol>
<li>우선 PC레지스터로 명령어의 주소를 보낸다.
<ul>
<li>이때 MMU를 통과한다.</li>
<li>요청된 명령어의 주소가 로컬 범위 밖이라면 cpu를 빼앗긴다.</li>
</ul>
</li>
<li>Decode 단계에서 명령어를 파악 및 검사한다.
<ul>
<li>만약 권한을 넘어서면 빼앗긴다.</li>
</ul>
</li>
</ol>
</li>
</ul>
<h2 id="사용자-입장--source-code">사용자 입장 / Source code</h2>
<hr>
<h3 id="컴파일">컴파일</h3>
<ul>
<li>우리가 코드를 짤 때는 I/O 코드를 짠다.</li>
<li>사실 컴파일 과정에서 컴파일러가 chmodk 명령을 집어 넣는다.</li>
<li>그리고 I/O statements는 바이너리( a.out )에서 보이지 않고 커널에서 존재하게 된다.
<blockquote>
<p>change mode (protection) kernel</p>
</blockquote>
</li>
</ul>
<h3 id="실행">실행</h3>
<ul>
<li>chmodk는 privilieged 명령이다.
<ul>
<li>즉 user_mode에서 실행이 불가능하다.</li>
</ul>
</li>
<li>이때 cpu 뺏기며 HW trap이 걸리게 된다.
<ul>
<li>Cpu state vector를 저장해둔다.</li>
</ul>
</li>
<li>trap handler가 작동하며 kernel mode로 바뀐다.
<ul>
<li>trap handler는 trap을 일으킨 정보를 통해 kernel 함수를 실행한다.</li>
</ul>
</li>
<li>이후 cpu 모드를 user_mode로 돌리고, 저장된 state vector 복구한다.</li>
<li>interrupted 된 address로 복귀한다,</li>
</ul>
<p><img src="https://t1.daumcdn.net/cfile/tistory/99ADA24F5F6366A31F?original" alt=""><br>
<img src="https://t1.daumcdn.net/cfile/tistory/992A514A5F6366A327?original" alt=""><br>
<img src="https://t1.daumcdn.net/cfile/tistory/9990524E5F6366A323?original" alt=""></p>
<ul>
<li>모든 프로그램은 두 개의 스택이 필요하다.
<ul>
<li>커널용, 유저용</li>
<li>함수 호출 시 사용된다.</li>
</ul>
</li>
</ul>
<p><img src="https://t1.daumcdn.net/cfile/tistory/99FDFA415F6366A322?original" alt=""></p>
<h2 id="system-call-흐름-정리">System call 흐름 정리</h2>
<hr>
<p><img src="https://t1.daumcdn.net/cfile/tistory/990E7D4D5F6366A327?original" alt=""></p>
<h2 id="system-call--자세히...">System call  자세히…</h2>
<hr>
<h3 id="system-call-흐름">system call 흐름</h3>
<p><img src="https://t1.daumcdn.net/cfile/tistory/99C90F375F6367BA16?original" alt=""></p>
<ol>
<li>printf를 하면 라이브러리로 달려간다.</li>
<li>라이브러리에서 system call wrapper 함수를 부름 ( write(2) )</li>
</ol>
<p><img src="https://t1.daumcdn.net/cfile/tistory/9920F24B5F6367BA22?original" alt=""></p>
<ul>
<li>write( ) 에서 system call number를 넘김, 그리고 trap을 발생시킴
<ul>
<li>system call number는 정해져 있다. 컴파일러마다 다르다.</li>
</ul>
<blockquote>
<p>여기까지 컴파일러 / 이후 OS</p>
</blockquote>
</li>
<li>sys_call()에서 number를 이용해 system call table에서 sys_write( )를 찾아 실행함</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/177254990-a1aa3105-abd1-4fa6-a4fe-a37b6198b182.png" alt=""></p>
<h3 id="정보는-어떻게-교환하는가">정보는 어떻게 교환하는가</h3>
<ul>
<li>kernel 모드로 변경되면 이때 발생하는 I/O 정보는 어떻게 user 에게 전달 될까?</li>
<li>kernel 은 user와 달리 모든 메모리에 접근 할 수 있다.
<ul>
<li>즉 kernel은 user의 정보를 얻거나 전달할 수 있다.</li>
</ul>
</li>
<li>다음과 같은 함수들을 통해 구현된다.</li>
</ul>
<p><img src="https://t1.daumcdn.net/cfile/tistory/99926A4A5F6367BA26?original" alt=""></p>

