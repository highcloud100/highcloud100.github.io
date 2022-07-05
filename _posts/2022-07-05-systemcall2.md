---
title: System Call with linux
author: highcloud
date: 2022-07-05 16:19:00 +0900
categories: [CS, OS]
tags: [os, cs, summer, 실습과 그림으로 배우는 리눅스 구조]
render_with_liquid: false
---

<h1 id="system-call-with-linux">System Call with Linux</h1>
<hr>
<h2 id="system-call-확인하기">system call 확인하기</h2>
<hr>
<p><img src="https://user-images.githubusercontent.com/80192345/177256790-a5f65579-93e2-43b6-a480-4268d27951f9.png" alt=""></p>
<ul>
<li>간단한 hello world 예제이다.</li>
<li>strace를 이용해 요청한 시스템 콜을 알아보자.</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/177257568-b9d15bf5-b8f2-4c9a-a798-fb00970e81c7.png" alt=""></p>
<ul>
<li>strace 각각의 줄은 1개의 시스템 콜이다.
<ul>
<li>write( ) 시스템 콜을 확인할 수 있다.</li>
<li>이외에도 많은 시스템 콜이 호출되었다.
<ul>
<li>대부분 main( ) 함수 앞뒤로 실행된 것들이며 프로그램의 시작과 종료 처리 중 호출된 것이다.</li>
</ul>
</li>
</ul>
</li>
</ul>
<h3 id="system-call의-소요-시간-확인하기">system call의 소요 시간 확인하기</h3>
<ul>
<li>strace -T 옵션을 이용
<ul>
<li>시스템 콜 처리에 걸린 시간을 측정 가능하다.</li>
<li>system 점유율이 높을 때 이 기능을 통해 부하를 찾아낸다.</li>
</ul>
</li>
</ul>
<h2 id="실험">실험</h2>
<hr>
<h3 id="cpu-정보-확인하는-법">CPU 정보 확인하는 법</h3>
<p><img src="https://user-images.githubusercontent.com/80192345/177258628-ced61d84-1530-445a-ab22-6078e314a1d5.png" alt=""></p>
<pre class=" language-shell"><code class="prism  language-shell">sar -P ALL 1
</code></pre>
<ul>
<li>명령어를 통해 cpu 의 모드를 확인할 수 있다.</li>
<li>idle
<ul>
<li>kernel, user mode가 아닌 상태</li>
</ul>
</li>
<li>user, nice
<ul>
<li>user mode인 상태</li>
</ul>
</li>
<li>system
<ul>
<li>kernel mode인 상태</li>
</ul>
</li>
</ul>
<h3 id="실험-1">실험 1</h3>
<hr>
<ul>
<li>loop를 걸어서 확인해보자.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span></span>
<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token keyword">for</span><span class="token punctuation">(</span><span class="token punctuation">;</span><span class="token punctuation">;</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p><img src="https://user-images.githubusercontent.com/80192345/177263301-bb68f193-e672-4f69-8b9a-feb5a50b86a4.png" alt=""></p>
<ul>
<li>user mode에서 계속 돌아가는 것을 볼 수 있다.
<ul>
<li>cpu 3번</li>
</ul>
</li>
</ul>
<h3 id="실험-2">실험 2</h3>
<hr>
<ul>
<li>이번에는 getppid( ) 시스템 콜을 이용해보자.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;sys/types.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;unistd.h&gt;</span></span>

<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
	<span class="token keyword">for</span><span class="token punctuation">(</span><span class="token punctuation">;</span><span class="token punctuation">;</span><span class="token punctuation">)</span>
		<span class="token function">getppid</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p><img src="https://user-images.githubusercontent.com/80192345/177264219-808ece62-af0f-436f-a5e5-786ef53c0c53.png" alt=""></p>
<ul>
<li>일정 비율로 kernel과 user mode가 실행되었음을 알 수 있다.
<ul>
<li>즉 사용자 모드 -&gt; 커널 모드 -&gt; 사용자 모드 …</li>
</ul>
</li>
</ul>
<h2 id="system-call의-wrapper-함수">system call의 wrapper 함수</h2>
<hr>
<ul>
<li>
<p>system call은 보통 C언어 같은 고급 언어에서 직접 호출이 불가능하다.</p>
</li>
<li>
<p>아키텍처에 의존하는 어셈블리 코드를 사용해 호출된다.</p>
</li>
<li>
<p>getppid( ) 시스템 콜은 x86_64 아키텍처에서 다음과 같이 호출된다.</p>
</li>
</ul>
<pre class=" language-assembly"><code class="prism  language-assembly">mov $0x6e, %eax // 시스템 콜 번호를 eax 레지스터에 대입
syscall
</code></pre>
<ul>
<li>
<p>만약 wrapper 함수가 없었다면 어셈블리 언어를 사용해서 system call을 호출했을 것이다.</p>
<ul>
<li>이는 이식성이 매우 낮으며 프로그램 작성 시 오래 걸린다.</li>
</ul>
</li>
<li>
<p>고급언어로 작성된 사용자 프로그램은 단순히 시스템 콜의 wrapper 함수를 호출하면 된다.</p>
</li>
<li>
<p>wrapper 함수는 단순하게 시스템 콜을 요청하는 함수이다.</p>
<ul>
<li>trap이 걸리면 eax레지스터를 참조해 커널 모드로 함수가 실행됨.</li>
</ul>
</li>
</ul>
<h2 id="표준-c-라이브러리">표준 C 라이브러리</h2>
<hr>
<ul>
<li>
<p>c 언어에는 표준 라이브러리가 있다.</p>
<ul>
<li><a href="https://ko.wikipedia.org/wiki/Glibc">glibc</a> 를 사용한다.</li>
</ul>
</li>
<li>
<p>glibc는 system call의 wrapper 함수를 포함한다.</p>
<ul>
<li><a href="https://ko.wikipedia.org/wiki/POSIX">posix</a> 규격에 정의된 함수도 제공</li>
</ul>
</li>
<li>
<p>ldd 명령을 통해 링크하는 라이브러리를 볼 수 있다.</p>
</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/177268168-3d5264ad-dc40-4ffc-b0f5-fa3043bdaf7e.png" alt=""><br>
<img src="https://user-images.githubusercontent.com/80192345/177268275-7c4424a6-852f-4931-9ac9-542b720f04f0.png" alt=""></p>
<ul>
<li>libc / 표준 C 라이브러리를 링크하고 있다.</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/177270378-2f3794ef-3a5f-451b-a537-2d519f4962f3.png" alt=""></p>
<ul>
<li>참고로 파이썬도 내부적으로는 C 라이브러리를 사용한다.</li>
</ul>
<p>이해가 안 간다면 <a href="https://www.inflearn.com/questions/9920">다음 글</a>을 참고해보자.</p>

