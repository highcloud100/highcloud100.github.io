---
title: Schedule
author: highcloud
date: 2022-07-15 13:00:00 +0900
categories: [CS, OS]
tags: [os, cs, summer]
render_with_liquid: false
---

<h1 id="process-state">Process State</h1>
<hr>
<h2 id="active-status">Active status</h2>
<hr>
<ul>
<li>프로세스의 상태를 정리해보자.</li>
<li>생성, 준비, 실행, 대기, 완료 -&gt; 5가지 상태</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/178965720-fce90c6f-d281-40e0-85ff-cb2589d47b86.png" alt=""></p>
<ul>
<li>
<p>Ready</p>
<ul>
<li>cpu 스케줄러의 호출을 기다리는 상태</li>
</ul>
</li>
<li>
<p>Running</p>
<ul>
<li>cpu를 사용하고 있는 상태</li>
<li>I/O 요청 시 cpu를 뺏김 -&gt; I/O까지 wait</li>
<li>timeslice를 모두 소모하면 뺏김</li>
</ul>
</li>
<li>
<p>Waiting</p>
<ul>
<li>blocking status</li>
<li>입출력이 완료될 때까지 기다리는 상태이다.</li>
<li>입출력이 끝나면 인터럽트 발생
<ul>
<li>해당 프로세스 준비 상태로 이동</li>
</ul>
</li>
</ul>
</li>
<li>
<p>Zombie</p>
<ul>
<li>pcb만 남기고 다 지워짐</li>
<li>이후 부모에게 신호를 보낸다. ( 회수 요청 )</li>
<li>부모 프로세스는 wait 시스템 콜을 통해 자식 프로세스의 종료 상태를 읽어 들인다.
<ul>
<li>부모가 os에게 자식이 종료되었는지 확인함</li>
<li>이후 완전히 pcb까지 소멸된다.</li>
</ul>
</li>
</ul>
</li>
</ul>
<blockquote>
<p>고아 프로세스<br>
부모가 wait 대신 그냥 종료되버림<br>
init을 부모로 삼는다. init은 주기적은 wait 요청으로 고아를 거둠<br>
<img src="https://user-images.githubusercontent.com/80192345/178975180-03dab8ae-9efe-48f8-ab38-81130019fb05.png" alt=""></p>
</blockquote>
<h2 id="특별한-상태">특별한 상태</h2>
<hr>
<h3 id="휴식-상태">휴식 상태</h3>
<hr>
<ul>
<li>pause status</li>
<li>작업을 일시적으로 쉬고 있는 상태</li>
</ul>
<h3 id="보류-상태">보류 상태</h3>
<hr>
<ul>
<li>suspend status</li>
<li>메모리에서 잠시 쫓겨난 상태
<ul>
<li>메모리가 꽉차서 밀려남</li>
<li>오류 발생</li>
<li>바이러스, 악의적 행동</li>
<li>매우 긴 주기</li>
<li>입출력 계속 지연</li>
</ul>
</li>
<li>메모리 밖인 swap area에 보관됨</li>
<li>이후 보류 준비 상태를 지나 준비 상태로 이동한다.</li>
</ul>
<h1 id="process-scheduler">Process scheduler</h1>
<hr>
<ul>
<li>cpu 스케줄러는 프로세스의 모든 상태 변화를 조정한다.</li>
<li><a href="https://en.wikipedia.org/wiki/Scheduling_(computing)">목적은 찾아보자</a></li>
</ul>
<h2 id="스케줄링-단계">스케줄링 단계</h2>
<hr>
<ul>
<li>스케줄링은 3단계로 나뉜다.
<ol>
<li>고수준 스케줄링</li>
<li>저수준 스케줄링</li>
<li>중간 수준 스케줄링</li>
</ol>
</li>
</ul>
<h3 id="high-level-scheduling">High level scheduling</h3>
<hr>
<ul>
<li>작업 스케줄링 ( job )이라고도 부른다. 전체 작업 수를 조절한다.</li>
<li>작업은 os의 가장 큰 일의 단위로 하나 혹은 여러 개의 프로세스로 이루어진다.</li>
<li>작업이 시작되면 자원이 소모되기에 기존 작업에 영향이 간다.
<ul>
<li>이에 작업을 승인할지 거부할지 결정한다.</li>
<li>Admission scheduling 승인 스케줄링이라고도 부른다.</li>
</ul>
</li>
<li>동시에 실행 가능한 프로세스의 총 개수가 정해진다.</li>
</ul>
<h3 id="middle-level-scheduling">Middle level scheduling</h3>
<hr>
<ul>
<li>suspend와 active로 활성화된 프로세스 수를 조절한다.
<ul>
<li>일부 프로세스를 suspend 상태로 돌려 나머지가 원활하게 만든다.</li>
</ul>
</li>
<li>고수준과 저수준 스케줄링 사이의 버퍼 역할</li>
</ul>
<h3 id="low-level-scheduling">Low level scheduling</h3>
<hr>
<ul>
<li>cpu에 프로세스를 배당하는 작업을 한다.</li>
<li>빠르게 작동하여 short-term scheduling 이라도 부른다.</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/178981275-a167f4d8-c476-4cb7-b3ba-d99093cada4e.png" alt=""></p>
<h2 id="preemptive--non-preemptive-scheduling">Preemptive &amp; Non preemptive scheduling</h2>
<hr>
<ul>
<li>선점형 스케줄링
<ul>
<li>운영체제가 강제로 cpu를 뺏을 수 있다.</li>
</ul>
</li>
<li>비 선점형 스케줄링
<ul>
<li>운영체제가 강제로 뺏을 수 없다.</li>
</ul>
</li>
</ul>
<h3 id="preemptive">Preemptive</h3>
<hr>
<ul>
<li>선점형 스케줄링은 인터럽트처럼 cpu를 뺏을 수 있다.
<ul>
<li>문맥 교환 같은 낭비가 생긴다.</li>
<li>시분할 시스템에 사용된다.
<ul>
<li>사실상 대부분</li>
</ul>
</li>
</ul>
</li>
</ul>
<h3 id="non-preemptive">Non preemptive</h3>
<hr>
<ul>
<li>한 프로세스가 실행 상태에 들어가면 프로세스가 멈추지 않으면, cpu를 뺏을 수 없다.</li>
<li>스케줄링이나 문맥 교환의 오버헤드가 낮다.</li>
<li>다만 전체 시스템의 처리율이 떨어진다.
<ul>
<li>과거 일괄 작업 시스템에서 사용되었다.</li>
</ul>
</li>
</ul>
<h2 id="priority">Priority</h2>
<hr>
<ul>
<li>프로세스는 큐로 관리된다.</li>
<li>이때 우선순위와 그 척도가 필요하다.
<ul>
<li>기본적으로 커널 프로세스가 일반 프로세스보다 우선순위가 높다.</li>
<li>임의로 사용자가 지정할 수 있다.</li>
<li>전면 프로세스가 후면 프로세스보다 우선순위가 보통 높다.</li>
<li>입출력 집중 프로세스가 cpu 집중 프로세스보다 보통 높다.
<ul>
<li>입출력 집중 프로세스는 cpu를 잠깐 쓰고 넘기기 때문이다.</li>
</ul>
</li>
</ul>
</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/178984246-483e0470-2ea8-47c0-83d7-c8abd478a4c9.png" alt=""></p>
<h2 id="queue">Queue</h2>
<hr>
<h3 id="cpu-사용하는-프로세스들">cpu 사용하는 프로세스들</h3>
<p><img src="https://user-images.githubusercontent.com/80192345/178993511-5dac8a56-d654-40c9-8bc4-e56433faba42.png" alt=""></p>
<ul>
<li>pcb를 연결한 큐가 있다.</li>
<li>이때 우선순위에 따라 여러 큐를 만들어 multiple queue로 관리된다.
<ul>
<li>모두 검색하는 오버헤드를 줄임</li>
</ul>
</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/178993901-703b5262-ade6-4bc1-92ad-0f38f44010f3.png" alt=""></p>
<ul>
<li>이때 bitmap 배열을 이용해서 해당 우선순위의 큐가 pcb(task)를 가지고 있는지 확인한다.
<ul>
<li>만약 1이라면 해당 큐를 순회하며 cpu를 할당한다.</li>
</ul>
</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/178994332-fd15d188-1ea6-4dcf-b445-6848f4d88cab.png" alt=""></p>
<ul>
<li>프로세스가 time slice를 모두 소모하면 expired array로 넘어간다.
<ul>
<li>해당 우선순위에 맞게</li>
</ul>
</li>
<li>active array가 사용이 다 되면 expired 에 time slice를 배정하고 active array로 바꾼다.</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/178995050-19ba2c1f-fe25-49a9-a444-92ad8d77f50f.png" alt=""></p>
<h3 id="대기-상태의-프로세스들">대기 상태의 프로세스들</h3>
<ul>
<li>
<p>위에서 설명한 큐는 실행 상태를 다룬다.</p>
<ul>
<li>대기 상태의 프로세스도 큐를 가지고 있다.</li>
</ul>
</li>
<li>
<p>대기 상태 다단계 큐는 같은 입출력 장치끼리 모아 놓는다.</p>
<ul>
<li>입출력이 동시에 끝나면 여러 인터럽트가 한번에 처리된다.</li>
<li>이를 위해 인터럽트 벡터를 사용하며 입출력이 완료된 pcb 들은 준비 상태로 이동한다.</li>
</ul>
</li>
</ul>
<h3 id="정리">정리</h3>
<p><img src="https://user-images.githubusercontent.com/80192345/178999987-7ff0e5f9-df36-4a66-a1ea-9ae432e72b2c.png" alt=""></p>
<h1 id="scheduling-algorithm">Scheduling algorithm</h1>
<hr>
<ul>
<li>스케줄링 알고리즘을 알아보자.</li>
</ul>
<h2 id="종류">종류</h2>
<hr>

<table>
<thead>
<tr>
<th>구분</th>
<th>종류</th>
</tr>
</thead>
<tbody>
<tr>
<td>비선점형</td>
<td>FCFS, SJF, HRN</td>
</tr>
<tr>
<td>선점형</td>
<td>라운드 로빈, SRT, 다단계 큐, 다단계 피드백 큐</td>
</tr>
<tr>
<td>둘 다 가능</td>
<td>우선순위 스케줄링</td>
</tr>
</tbody>
</table><h2 id="성능-기준">성능 기준</h2>
<hr>
<p><img src="https://user-images.githubusercontent.com/80192345/179002876-985da6e5-b438-4975-afd6-a0fa85b45167.png" alt=""></p>
<ul>
<li>대기 시간 : 프로세스 생성 후 실행까지 걸린 시간</li>
<li>응답 시간 : 첫 작업을 시작한 후 첫 번째 반응까지 걸린 시간</li>
<li>실행 시간 : 프로세스 작업이 시작된 후 종료되기까지 시간</li>
<li>반환 시간 : 대기 시간을 포함하여 실행이 종료될 때까지의 시간</li>
</ul>
<h3 id="기준">기준</h3>
<ul>
<li>보통 스케줄링 알고리즘 성능 비교에는 평균 대기 시간을 사용한다.
<ul>
<li>모든 프로세스들이 실행되기까지 걸린 대기 시간의 평균</li>
<li>다만 순서나 작업 패턴에 따라 역전되기도 한다.</li>
</ul>
</li>
</ul>
<h2 id="fcfs">FCFS</h2>
<hr>
<ul>
<li>비선점형 방식</li>
<li>first come first served</li>
<li>선입선출 스케줄링
<ul>
<li>큐가 하나라 모든 프로세스는 우선순위가 동일하다.</li>
</ul>
</li>
<li>즉 하나씩 순서대로 실행된다.
<ul>
<li>일괄 작업 시스템</li>
</ul>
</li>
</ul>
<h3 id="단점">단점</h3>
<ul>
<li>convoy effect</li>
<li>입출력 요청시 cpu가 놀게됨</li>
</ul>
<h2 id="sjf">SJF</h2>
<hr>
<ul>
<li>비선점형 방식</li>
<li>shortest job first
<ul>
<li>shortest process first, 최단 프로세스 우선 스케줄링</li>
</ul>
</li>
<li>실행 시간이 가장 짧은 작업부터 cpu를 할당</li>
<li>convoy 효과를 완화함</li>
</ul>
<h3 id="단점-1">단점</h3>
<ol>
<li>운영체제가 프로세스의 종료 시간을 정확히 예측하기 어렵다.
<ul>
<li>즉 프로세스의 작업 길이를 알아야 스케줄을 하는데 길이를 모른다.
<ul>
<li>현대 프로세스는 사용자와의 상호작용이 빈번하기 때문이다.</li>
</ul>
</li>
<li>고로 적용하기 어렵다.</li>
</ul>
</li>
<li>공평하지 않다.
<ul>
<li>starvation</li>
<li>infinite blocking</li>
<li>이를 완화하기 위한 방법으로 aging이 있으나 기준 선정의 한계가 있다.</li>
</ul>
</li>
</ol>
<h2 id="hrn">HRN</h2>
<hr>
<ul>
<li>비선점형</li>
<li>highest response ratio next</li>
<li>SJF의 아사 현상을 해결하기 위한 알고리즘이다.
<ul>
<li>HRN은 서비스를 받기 위해 기다린 시간과 cpu 사용 시간을 고려하여 스케줄링한다.</li>
<li>즉 대기 시간을 고려해 아사를 완화한다.</li>
<li>스케줄링에 에이징을 구현한 것</li>
</ul>
</li>
</ul>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mtext>우선순위</mtext><mo>=</mo><mfrac><mtext>대기&nbsp;시간&nbsp;+&nbsp;CPU&nbsp;사용&nbsp;시간&nbsp;</mtext><mtext>CPU&nbsp;사용시간&nbsp;</mtext></mfrac></mrow><annotation encoding="application/x-tex">
\text{우선순위} ={ \text{대기 시간 + CPU 사용 시간 } \over \text{CPU 사용시간 }}
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord text"><span class="mord hangul_fallback">우선순위</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 2.04633em; vertical-align: -0.686em;"></span><span class="mord"><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.36033em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord text"><span class="mord">CPU&nbsp;</span><span class="mord hangul_fallback">사용시간</span><span class="mord">&nbsp;</span></span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.677em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord text"><span class="mord hangul_fallback">대기</span><span class="mord">&nbsp;</span><span class="mord hangul_fallback">시간</span><span class="mord">&nbsp;+&nbsp;CPU&nbsp;</span><span class="mord hangul_fallback">사용</span><span class="mord">&nbsp;</span><span class="mord hangul_fallback">시간</span><span class="mord">&nbsp;</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span></span></span></span></span></span></p>
<h3 id="단점-2">단점</h3>
<ul>
<li>여전히 공평성이 위배되어 많이 사용되지 않는다.</li>
</ul>
<h2 id="rr">RR</h2>
<hr>
<ul>
<li>선점형 알고리즘 중 가장 단순하고 대표적인 방식</li>
<li>round robin</li>
<li>한 프로세스가 할당 받은 시간 동안만 작업하고 큐의 맨 뒤로 보내지는 알고리즘
<ul>
<li>완료 될 때까지 계속 순환</li>
</ul>
</li>
<li>FCFS와 차이점은 time slice가 있다는 것이다.
<ul>
<li>이는 convoy 효과를 완화한다.</li>
</ul>
</li>
</ul>
<h3 id="유의점">유의점</h3>
<ul>
<li>문맥 교환에 대한 오버헤드를 생각해야 한다.
<ul>
<li>이에 타임 슬라이스의 크기가 중요하다.</li>
</ul>
</li>
<li>타임 슬라이스가 너무 크면 FCFS와 다르지 않다.</li>
<li>타임 슬라이스가 너무 작으면 문맥 교환 오버헤드가 커진다.</li>
</ul>
<h2 id="srt">SRT</h2>
<hr>
<ul>
<li>shortest remaining time
<ul>
<li>최소 잔류 시간 우선 스케줄링</li>
</ul>
</li>
<li>SJF와 RR을 혼합한 방법</li>
<li>기본적으로 RR 이다.
<ul>
<li>다만 작업 시간이 가장 적은 프로세스를 선택하여 cpu를 할당한다.</li>
</ul>
</li>
</ul>
<h3 id="단점-3">단점</h3>
<ul>
<li>실행중인 프로세스와 큐의 프로세스들의 시간을 주기적으로 계산해야한다.</li>
<li>문맥 교환 오버헤드</li>
<li>프로세스의 종료 시간 예측 어려움, 아사 현상</li>
<li>잘 안쓰임</li>
</ul>
<h2 id="priority-1">Priority</h2>
<hr>
<ul>
<li>중요도를 기준으로 우선 순위를 반영해 스케줄링한다.</li>
</ul>
<h3 id="단점-4">단점</h3>
<ul>
<li>공평성 위배와 아사 현상</li>
</ul>
<h2 id="mlq">MLQ</h2>
<hr>
<p><img src="https://user-images.githubusercontent.com/80192345/179135646-c6c31001-fb09-46f4-9f1b-c03fd220f580.png" alt=""></p>
<ul>
<li>
<p>MultiLevel Queue Scheduling 다단계 큐 스케줄링</p>
</li>
<li>
<p>우선 순위에 따라 준비 큐를 여러 개 사용하는 방식</p>
</li>
<li>
<p>고정형 우선순위를 사용한다.</p>
<ul>
<li>프로세스가 시간을 소모하면 원래 큐로 돌아간다.</li>
<li>상단의 큐의 모든 프로세스의 작업이 끝나야 다음 큐의 작업이 실행된다.</li>
<li>선점형 방식으로 우선순위가 높은 프로세스가 먼저 작동한다.
<ul>
<li>만약 하위 큐의 프로세스 진행 중 상위 단계 큐에 프로세스가 도착하면 뺏어버림</li>
</ul>
</li>
</ul>
</li>
<li>
<p>각각의 큐에 대한 다른 스케줄링 알고리즘 적용가능하다.</p>
</li>
<li>
<p>큐의 우선순위에 따라 타임 슬라이스를 조절할 수 있다.</p>
</li>
</ul>
<h3 id="단점-5">단점</h3>
<ul>
<li>우선 순위가 높은 큐가 완료되어야 다음 큐가 실행된다.
<ul>
<li>기아 현상</li>
</ul>
</li>
</ul>
<h2 id="mfq">MFQ</h2>
<hr>
<ul>
<li>multilevel feedback queue</li>
<li>우선순위가 낮은 프로세스들의 문제를 보완한 방식</li>
<li>cpu를 사용한 프로세스는 우선순위가 한 단계 낮아진다.
<ul>
<li>즉 다음 단계 큐로 들어간다.</li>
</ul>
</li>
<li>우선순위에 따라 큐의 타임 슬라이스가 다르다.
<ul>
<li>낮은 순위는 cpu를 얻을 확률이 상대적으로 낮다.</li>
<li>그래서 오래 사용하도록 타임 슬라이스를 크게 만들어준다.</li>
<li>마지막 큐의 타임 슬라이스는 무한대의 타임 슬라이스를 가지며 이는 FCFS 로 동작하는 것과 같다.</li>
</ul>
</li>
<li>MFQ는 오늘날의 운영체제가 사용하는 방식이다. 변동 우선순위 알고리즘의 전형적인 예이다.</li>
</ul>
<blockquote>
<p>물론 커널 프로세스는 일반 프로세스의 큐로 삽입되지 않는다.</p>
</blockquote>
<h2 id="룰">룰</h2>
<ul>
<li>짧은 작업에 우선권이 있다.</li>
<li>입출력 관련 프로세스에 우선권을 준다.</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/179147936-08e42e2e-8d61-40ad-9e2a-1b9883327f99.png" alt=""></p>

