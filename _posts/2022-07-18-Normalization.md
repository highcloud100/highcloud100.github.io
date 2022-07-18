---
title: Normalization
author: highcloud
date: 2022-07-18 23:10:00 +0900
categories: [CS, AI]
tags: [cs, AI, ML, summer]
render_with_liquid: false
---

<h1 id="normalization">Normalization</h1>
<hr>
<h2 id="종류">종류</h2>
<hr>
<ol>
<li>Batch normalization
<ul>
<li><a href="https://arxiv.org/abs/1502.03167">https://arxiv.org/abs/1502.03167</a></li>
<li>CNN</li>
</ul>
</li>
<li>Layer normalization
<ul>
<li><a href="https://arxiv.org/pdf/1607.06450.pdf">https://arxiv.org/pdf/1607.06450.pdf</a></li>
<li>RNN, Transformer</li>
</ul>
</li>
<li>Instance normalization</li>
<li>Group normalization</li>
</ol>
<h2 id="normalization-1">Normalization</h2>
<hr>
<ul>
<li>Standardization 표준화</li>
<li>왜 할까?
<ul>
<li>범위가 너무 넓어 step size를 정하기 어렵다.
<ul>
<li>보수적으로 잡게 됨</li>
</ul>
</li>
</ul>
</li>
<li>방법
<ul>
<li>평균이 0, 분산이 1이도록 만든다.</li>
</ul>
</li>
</ul>
<h1 id="batch-normalization">Batch Normalization</h1>
<hr>
<ul>
<li>data들이 whitened하면 Training converges 가 빠르게 진행된다.</li>
</ul>
<blockquote>
<p>whitened :  zero mean, unit variance, decorrelated</p>
</blockquote>
<ul>
<li>Deep Neural Networks 에 적용을 해보자!</li>
</ul>
<h2 id="internal-covariate-shift">Internal Covariate Shift</h2>
<hr>
<p><img src="https://user-images.githubusercontent.com/80192345/179454886-a2df93f1-962e-41b1-8120-44b34d2abec4.png" alt="" width="500"></p>
<ul>
<li>처음 input은 표준화 되어있다.
<ul>
<li>그러나 다음 layer에서는 표준화 되어 있지 않다.</li>
</ul>
</li>
</ul>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mi>h</mi><mo>=</mo><mo>∑</mo><mi>w</mi><mi>x</mi><mo>+</mo><mi>b</mi><mspace linebreak="newline"></mspace><msub><mi>h</mi><mn>1</mn></msub><mo>=</mo><mi>ϕ</mi><mo stretchy="false">(</mo><mi>h</mi><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">
h = \sum wx + b \\ h_1 = \phi(h)
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord mathnormal">h</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.60001em; vertical-align: -0.55001em;"></span><span class="mop op-symbol large-op" style="position: relative; top: -5e-06em;">∑</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord mathnormal" style="margin-right: 0.02691em;">w</span><span class="mord mathnormal">x</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord mathnormal">b</span></span><span class="mspace newline"></span><span class="base"><span class="strut" style="height: 0.84444em; vertical-align: -0.15em;"></span><span class="mord"><span class="mord mathnormal">h</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.301108em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">1</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathnormal">ϕ</span><span class="mopen">(</span><span class="mord mathnormal">h</span><span class="mclose">)</span></span></span></span></span></span></p>
<ul>
<li>보통 input distrubution 이 input과 output에서 같다고 생각한다.
<ul>
<li>만약 다른 상황이라면 이를 Covariate shift라고 부른다.</li>
<li>지금은 layer에서 벌어진 상황이기에 Internal Covariate shift라고 부른다.</li>
</ul>
</li>
</ul>
<h2 id="batch-normalization-1">Batch Normalization</h2>
<hr>
<ul>
<li>이제 batch normalization에 대해 알아보자.</li>
<li>우선 하나의 노드를 기준으로 생각한다.</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/179466661-f362ae19-b00d-488a-959b-44ec0b030ec1.png" alt="" width="500"></p>
<ul>
<li>
<p>batch는 입력 데이터의 개수이다.</p>
<ul>
<li>입력 노드의 수가 아니다.</li>
</ul>
</li>
<li>
<p>batch가 10이라면 10번 feed foward가 발생한다.</p>
<ul>
<li>이때 h1이 된 값은 10개가 존재한다.</li>
<li>이 값들을 가지고 normalization을 진행한다.</li>
</ul>
</li>
<li>
<p><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>M</mi><mo>=</mo><mtext>size&nbsp;of&nbsp;mini-batch</mtext></mrow><annotation encoding="application/x-tex">M = \text{size of mini-batch}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">M</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">size&nbsp;of&nbsp;mini-batch</span></span></span></span></span></span> 일때 수식으로 정리해보면 다음과 같다.<br>
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><msub><mover accent="true"><mi>z</mi><mo>~</mo></mover><mi>i</mi></msub><mo>=</mo><mfrac><mrow><msub><mi>h</mi><mi>i</mi></msub><mo>−</mo><msub><mi>μ</mi><mi>i</mi></msub></mrow><msqrt><mrow><msubsup><mi>σ</mi><mi>i</mi><mn>2</mn></msubsup><mo>+</mo><mi>ϵ</mi></mrow></msqrt></mfrac></mrow><annotation encoding="application/x-tex">
\tilde z_i = {h_i - \mu _i \over \sqrt{\sigma_i^2+\epsilon}}
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.81786em; vertical-align: -0.15em;"></span><span class="mord"><span class="mord accent"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.66786em;"><span class="" style="top: -3em;"><span class="pstrut" style="height: 3em;"></span><span class="mord mathnormal" style="margin-right: 0.04398em;">z</span></span><span class="" style="top: -3.35em;"><span class="pstrut" style="height: 3em;"></span><span class="accent-body" style="left: -0.19444em;"><span class="mord">~</span></span></span></span></span></span></span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.04398em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 2.50144em; vertical-align: -1.13em;"></span><span class="mord"><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.37144em;"><span class="" style="top: -2.16548em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord sqrt"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.944522em;"><span class="svg-align" style="top: -3.2em;"><span class="pstrut" style="height: 3.2em;"></span><span class="mord" style="padding-left: 1em;"><span class="mord"><span class="mord mathnormal" style="margin-right: 0.03588em;">σ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.795908em;"><span class="" style="top: -2.42314em; margin-left: -0.03588em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span><span class="" style="top: -3.0448em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.276864em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mord mathnormal">ϵ</span></span></span><span class="" style="top: -2.90452em;"><span class="pstrut" style="height: 3.2em;"></span><span class="hide-tail" style="min-width: 1.02em; height: 1.28em;"><svg width="400em" height="1.28em" viewBox="0 0 400000 1296" preserveAspectRatio="xMinYMin slice"><path d="M263,681c0.7,0,18,39.7,52,119
c34,79.3,68.167,158.7,102.5,238c34.3,79.3,51.8,119.3,52.5,120
c340,-704.7,510.7,-1060.3,512,-1067
l0 -0
c4.7,-7.3,11,-11,19,-11
H40000v40H1012.3
s-271.3,567,-271.3,567c-38.7,80.7,-84,175,-136,283c-52,108,-89.167,185.3,-111.5,232
c-22.3,46.7,-33.8,70.3,-34.5,71c-4.7,4.7,-12.3,7,-23,7s-12,-1,-12,-1
s-109,-253,-109,-253c-72.7,-168,-109.3,-252,-110,-252c-10.7,8,-22,16.7,-34,26
c-22,17.3,-33.3,26,-34,26s-26,-26,-26,-26s76,-59,76,-59s76,-60,76,-60z
M1001 80h400000v40h-400000z"></path></svg></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.295478em;"><span class=""></span></span></span></span></span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.677em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord"><span class="mord mathnormal">h</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mord"><span class="mord mathnormal">μ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 1.13em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span></span></span></span></span></span></p>
</li>
</ul>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><msub><mi>μ</mi><mi>i</mi></msub><mo>=</mo><mfrac><mn>1</mn><mi>M</mi></mfrac><munderover><mo>∑</mo><mrow><mi>m</mi><mo>=</mo><mn>1</mn></mrow><mi>M</mi></munderover><msub><mi>a</mi><mrow><mi>i</mi><mo separator="true">,</mo><mi>m</mi></mrow></msub><mtext>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</mtext><msubsup><mi>σ</mi><mi>i</mi><mn>2</mn></msubsup><mo>=</mo><mfrac><mn>1</mn><mi>M</mi></mfrac><munderover><mo>∑</mo><mrow><mi>m</mi><mo>=</mo><mn>1</mn></mrow><mi>M</mi></munderover><mo stretchy="false">(</mo><msub><mi>a</mi><mrow><mi>i</mi><mo separator="true">,</mo><mi>m</mi></mrow></msub><mo>−</mo><msub><mi>μ</mi><mi>i</mi></msub><msup><mo stretchy="false">)</mo><mn>2</mn></msup></mrow><annotation encoding="application/x-tex">
\mu _i = {1\over M}\sum_{m=1}^M a_{i,m} \ \ \ \ \  \sigma_i^2 = {1\over M}\sum_{m=1}^M(a_{i,m}-\mu_i)^2
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.625em; vertical-align: -0.19444em;"></span><span class="mord"><span class="mord mathnormal">μ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 3.09545em; vertical-align: -1.26711em;"></span><span class="mord"><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.32144em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord mathnormal" style="margin-right: 0.10903em;">M</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.677em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">1</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mop op-limits"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.82834em;"><span class="" style="top: -1.88289em; margin-left: 0em;"><span class="pstrut" style="height: 3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">m</span><span class="mrel mtight">=</span><span class="mord mtight">1</span></span></span></span><span class="" style="top: -3.05001em;"><span class="pstrut" style="height: 3.05em;"></span><span class=""><span class="mop op-symbol large-op">∑</span></span></span><span class="" style="top: -4.30001em; margin-left: 0em;"><span class="pstrut" style="height: 3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight" style="margin-right: 0.10903em;">M</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 1.26711em;"><span class=""></span></span></span></span></span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathnormal">a</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">i</span><span class="mpunct mtight">,</span><span class="mord mathnormal mtight">m</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mspace">&nbsp;</span><span class="mspace">&nbsp;</span><span class="mspace">&nbsp;</span><span class="mspace">&nbsp;</span><span class="mspace">&nbsp;</span><span class="mord"><span class="mord mathnormal" style="margin-right: 0.03588em;">σ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.864108em;"><span class="" style="top: -2.453em; margin-left: -0.03588em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.247em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 3.09545em; vertical-align: -1.26711em;"></span><span class="mord"><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.32144em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord mathnormal" style="margin-right: 0.10903em;">M</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.677em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">1</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mop op-limits"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.82834em;"><span class="" style="top: -1.88289em; margin-left: 0em;"><span class="pstrut" style="height: 3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">m</span><span class="mrel mtight">=</span><span class="mord mtight">1</span></span></span></span><span class="" style="top: -3.05001em;"><span class="pstrut" style="height: 3.05em;"></span><span class=""><span class="mop op-symbol large-op">∑</span></span></span><span class="" style="top: -4.30001em; margin-left: 0em;"><span class="pstrut" style="height: 3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight" style="margin-right: 0.10903em;">M</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 1.26711em;"><span class=""></span></span></span></span></span><span class="mopen">(</span><span class="mord"><span class="mord mathnormal">a</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">i</span><span class="mpunct mtight">,</span><span class="mord mathnormal mtight">m</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.286108em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1.11411em; vertical-align: -0.25em;"></span><span class="mord"><span class="mord mathnormal">μ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mclose"><span class="mclose">)</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.864108em;"><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span></span></span></span></span></span></span></span></span></span></p>
<ul>
<li>
<p>이후에는 Scale 과 Shift 과정을 지난다.</p>
<ul>
<li>이는 하나의 layer가 추가된 것으로 볼 수 있다.<br>
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><msub><mi mathvariant="bold">z</mi><mi>i</mi></msub><mo>=</mo><msub><mi>γ</mi><mi>i</mi></msub><msub><mover accent="true"><mi>z</mi><mo>~</mo></mover><mi>i</mi></msub><mo>+</mo><msub><mi>β</mi><mi>i</mi></msub></mrow><annotation encoding="application/x-tex">
\bold z_i = \gamma_i \tilde z_i+\beta_i
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.59444em; vertical-align: -0.15em;"></span><span class="mord"><span class="mord mathbf">z</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.8623em; vertical-align: -0.19444em;"></span><span class="mord"><span class="mord mathnormal" style="margin-right: 0.05556em;">γ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.05556em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mord"><span class="mord accent"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.66786em;"><span class="" style="top: -3em;"><span class="pstrut" style="height: 3em;"></span><span class="mord mathnormal" style="margin-right: 0.04398em;">z</span></span><span class="" style="top: -3.35em;"><span class="pstrut" style="height: 3em;"></span><span class="accent-body" style="left: -0.19444em;"><span class="mord">~</span></span></span></span></span></span></span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.04398em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.88888em; vertical-align: -0.19444em;"></span><span class="mord"><span class="mord mathnormal" style="margin-right: 0.05278em;">β</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.311664em;"><span class="" style="top: -2.55em; margin-left: -0.05278em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.15em;"><span class=""></span></span></span></span></span></span></span></span></span></span></span></li>
</ul>
</li>
<li>
<p>왜 scale과 shift 과정이 필요할까?</p>
<ul>
<li>다음 그림을 보며 이해해보자.</li>
</ul>
</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/179464592-0ba67664-62ab-44f1-b5e6-9bcbc81810d6.png" alt=""></p>
<ul>
<li>
<p>우선 normalization 과정은 빨간 분포를 중앙의 파란 분포로 바꾸는 과정이다.</p>
<ul>
<li>이후 우리는 sigmoid 를 취하게 되는데 빨간색에 위치한 sigmoid의 경사는 파란색 분포보다 완만하다.</li>
<li>만약 빨간색에서 계속 기울기가 계산된다면 매우 느리게 converge 할 것이다.
<ul>
<li>gradient vanishing 문제</li>
</ul>
</li>
</ul>
</li>
<li>
<p>이때 표준화 과정에서 항상 평균을 0으로 분산을 1로 하는 것이 의미가 있을까?</p>
<ul>
<li>우선 표준화 과정의 장점은 빠른 converge이다.</li>
<li>gradient vanishing 해결</li>
<li>하지만 모두 같은 기울기를 가지게 되고 , 이는 활성화 함수가 비선형 함수로써 의미가 없어지게 만든다.</li>
</ul>
</li>
<li>
<p>그래서 두 개의 변수를 추가해서 scale과 shift 과정을 만든다.</p>
<ul>
<li>랜덤성을 부여하는 것이다.
<ul>
<li>중앙이 답이 아닐 수도 있으니까</li>
</ul>
</li>
<li>두 변수는 역전파 과정에서 학습된다.  즉 데이터가 정한다.</li>
</ul>
</li>
</ul>
<h2 id="bn-in-inference-phase">BN in Inference phase</h2>
<hr>
<ul>
<li>
<p>학습 시에는 배치 크기 기준으로 학습을 했다.</p>
</li>
<li>
<p>그렇다면 테스트 시에는 무엇을 기준으로 해야 될까?</p>
<ul>
<li>이때는 단순히 마지막 결과를 사용하는 것이 아닌 평균을 이용한다.</li>
</ul>
</li>
<li>
<p>즉 한번 배치를 돌 때마다 평균 값과 분산 값이 나오게 됨</p>
<ul>
<li>이 값들을 다시 평균 내서 입력 값을 표준화 하는데 사용한다.</li>
<li>이후 학습된 <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>γ</mi><mtext>와</mtext><mi>β</mi></mrow><annotation encoding="application/x-tex">\gamma \text{와} \beta</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.88888em; vertical-align: -0.19444em;"></span><span class="mord mathnormal" style="margin-right: 0.05556em;">γ</span><span class="mord text"><span class="mord hangul_fallback">와</span></span><span class="mord mathnormal" style="margin-right: 0.05278em;">β</span></span></span></span></span> 값을 이용해 scale, shift 해준다.</li>
</ul>
</li>
</ul>
<blockquote>
<p>분산을 평균 낼 때는 <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mfrac><mi>M</mi><mrow><mi>M</mi><mo>−</mo><mn>1</mn></mrow></mfrac></mrow><annotation encoding="application/x-tex">M\over M-1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.27566em; vertical-align: -0.403331em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.872331em;"><span class="" style="top: -2.655em;"><span class="pstrut" style="height: 3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight" style="margin-right: 0.10903em;">M</span><span class="mbin mtight">−</span><span class="mord mtight">1</span></span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.394em;"><span class="pstrut" style="height: 3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight" style="margin-right: 0.10903em;">M</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.403331em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span></span></span></span>을 곱한다.  (편향)</p>
</blockquote>
<blockquote>
<p>moving avg를 구한다.</p>
</blockquote>
<h2 id="장점">장점</h2>
<hr>
<ul>
<li>step size 를 증가시킬 수 있다.
<ul>
<li>speed up</li>
</ul>
</li>
<li>remove dropout</li>
<li>reduce L2 weight regularization</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/179531135-86ba13d6-69e6-4957-8a4b-d2b581f20aaa.png" alt=""></p>
<h1 id="layer-normalization">Layer Normalization</h1>
<hr>
<ul>
<li>rnn에는 BN이 잘 안 맞음
<ul>
<li>시간의 흐름이 중요한데 배치마다 평균 내버림</li>
</ul>
</li>
<li>배치 사이즈가 작으면 BN의 문제가 있다.
<ul>
<li>( 이유는 찾아보자 )</li>
</ul>
</li>
</ul>
<h2 id="ln">LN</h2>
<hr>
<ul>
<li>average of node</li>
<li>말 그대로 레이어 값들의 통계량을 이용한다.</li>
<li>BN은 Batch를 기준으로 통계량을 만들어 이용했다면</li>
<li>LN은 한 input에 대하여 작동하며, 같은 층의 노드들의 통계량을 사용한다.</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/179529794-eb06e753-e9af-4f41-bfa5-13f06761be00.png" alt="" width="500"></p>
<ul>
<li>다음과 같은 결과가 나온다.</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/80192345/179530862-2a262354-f28e-46c2-862a-888963b843d0.png" alt=""></p>
<h1 id="이후-내용">이후 내용</h1>
<hr>
<ul>
<li>CNN을 배우고 오자!</li>
</ul>

