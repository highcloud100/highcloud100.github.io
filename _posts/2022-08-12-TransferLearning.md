---
title: Few shot, Transfer, Domain adaptation
author: highcloud
date: 2022-08-12 19:30:00 +0900
categories: [CS, AI]
tags: [cs, AI, ML, summer]
render_with_liquid: false
---

<h1 id="few-shot-learning">Few shot learning</h1>
<hr>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/80192345/184333963-c2ea512c-c21d-4f68-8364-3a454da9e7b5.png">
<h2 id="def">Def</h2>
<hr>
<ul>
<li>여러 해석 중 가장 인정되고 있는 정의를 소개한다.</li>
<li>2020년 다음 survey에서 FSL의 디테일한 정의가 언급된다.
<ul>
<li>Yaqing Wang, Quanming Yao, James T Kwok, and Lionel M Ni. Generalizing from a few examples: A survey on few-shot learning. ACM Computing Surveys (CSUR), 53(3):1–34, 2020.</li>
</ul>
</li>
</ul>
<img width="790" alt="image" src="https://user-images.githubusercontent.com/80192345/185036005-7b7f963e-117d-4aa7-9132-a12c79928d0f.png">
<ul>
<li>
<p>어떤 데이터 E를 통해 사용하여 예측 결과를 만들었을 때 E가 매우 작다면 FSL이라 부른다.</p>
</li>
<li>
<p>FSL을 잘 이해하기 위해서는 두 가지 컨셉이 필요하다.</p>
<ol>
<li>N-way-K-shot problem</li>
<li>cross-domain FSL</li>
</ol>
</li>
</ul>
<h3 id="n-way-k-shot-problem">N-way-K-shot problem</h3>
<ul>
<li>FSL 에서 발생하는 특정 문제를 설명하는데 사용된다.</li>
<li>support set 은 학습하는데 사용되는 작은 dataset을 말한다.
<ul>
<li>N은 support set의 카테고리 종류 수이다.</li>
<li>K는 support set의 각 카테고리의 샘플 수이다.</li>
</ul>
</li>
<li>query set은 모델이 예측할 데이터들을 의미한다.</li>
<li>모델은 전체적으로 N * K 개의 샘플을 가지게 된다.</li>
</ul>
<blockquote>
<p>N categories and K samples per category</p>
</blockquote>
<h3 id="cross-domain-fsl">Cross-domain FSL</h3>
<ul>
<li>cross-domain originates from transfer learning
<ul>
<li>source domain에서 target domain으로 knowledge가 transfer 되는 것
<ul>
<li>이때 두 domain에는 차이가 있다.</li>
</ul>
</li>
<li>cross-domain FSL은 cross-domain과 FSL을 합친 것이다.
<ul>
<li>최근의 발생한 도전적인 방향이다.</li>
</ul>
</li>
</ul>
</li>
</ul>
<h2 id="different-from-machine-learning">Different from Machine learning</h2>
<hr>
<ul>
<li>
<p>전통적 ML은 큰 데이터 셋에 의존적이다.</p>
</li>
<li>
<p>적은 샘플로 수행되기 어렵다.</p>
</li>
<li>
<p>반면 FSL은  data scarcity scenario를 처리할 방법을 제공한다.</p>
</li>
<li>
<p>중요한 차이점은 support set과 query set이 disjoint 하다는 것이다.</p>
<ul>
<li>즉 분리되어 있다.</li>
<li>기존 ML은 훈련 데이터에 예측할 클래스가 포함되어 있다.</li>
</ul>
</li>
<li>
<p>unseen task를 마주하면 기존 ML은 큰 데이터를 필요로 하지만 FSL은 적은 훈련 데이터와 자원을 통해 튜닝 가능하다.</p>
</li>
</ul>
<h2 id="different-from-transfer-learning">Different from Transfer Learning</h2>
<hr>
<ul>
<li>만약 다른 task나 domain에서의 사전 훈련을 통해 사전 지식을 얻은 경우 FSL은 전이 학습에 속할 수 있다.
<ul>
<li>즉 전이 학습을 거치고 downstream task 에서 few 데이터를 사용하는 경우다.</li>
</ul>
</li>
<li>FSL을 위한 방법으로 Transfer Learning이 사용된다.</li>
</ul>
<h2 id="different-from-meta-learning">Different from Meta-Learning</h2>
<hr>
<ul>
<li>특정 작업에서 학습하는 방법을 모델에 가르치기 위해 사전 지식을 사용하는 경우 메타 학습은 FSL의 변종으로 간주 할 수 있다.</li>
<li>하지만 메타 학습은 FSL과 동일하지 않다.
<ul>
<li>FSL은 궁극적인 목표로 봐야 한다.</li>
</ul>
</li>
</ul>
<h1 id="transfer-learning">Transfer learning</h1>
<hr>
<ul>
<li>
<p>특정 태스크를 학습한 모델을 다른 태스크 수행에 재사용하는 방법이다.</p>
</li>
<li>
<p>학습 데이터가 적어도 효과적, 학습 속도도 빠르다.</p>
</li>
<li>
<p>upstream task</p>
<ul>
<li>pretrain : upstream task를 학습하는 과정</li>
</ul>
</li>
<li>
<p>downstream task</p>
<ul>
<li>풀어야 하는 구체적인 과제</li>
<li>이를 학습하는 방법은 다음과 같다.
<ul>
<li>fine-tuning
<ul>
<li>다운스트림 태스크 데이터 전체 사용</li>
<li>모델 전체를 업데이트</li>
</ul>
</li>
<li>prompt tuning
<ul>
<li>다운스트림 태스크 데이터 전체 사용</li>
<li>다운스트림 데이터에 맞게 모델 일부만 업데이트</li>
</ul>
</li>
<li>in-context learning<br>
-	 다운스트림 태스크 데이터 일부 사용<br>
-	모델을 업데이트 하지 않는다.</li>
</ul>
</li>
</ul>
</li>
<li>
<p>In-context learning 의 3가지 방식</p>
<ul>
<li>zero shot learning
<ul>
<li>다운 스트림 태스크 데이터를 전혀 사용하지 않음</li>
</ul>
</li>
<li>one shot learning
<ul>
<li>다운 스트림 태스크 데이터를 1 건만 사용</li>
</ul>
</li>
<li>few shot learning
<ul>
<li>다운스트림 태스크 데이터 일부 사용</li>
</ul>
</li>
</ul>
</li>
<li>
<p>네트워크가 보편적 특징을 학습했기 때문에 전이 학습이 가능하다.</p>
</li>
</ul>
<p><img src="https://blog.kakaocdn.net/dn/OzIUv/btrs8AsKuH8/OHHLKL2orZNqyaoCzuckJ0/img.png" alt=""><br>
<img src="https://blog.kakaocdn.net/dn/bnvwit/btrtb2B5nT1/VKBOjfy45Ssc2BcydDd8s1/img.png" alt=""></p>
<ul>
<li><a href="https://arxiv.org/pdf/1811.08883.pdf">논문</a>을 통해 전이학습이 학습 속도에서 효과 있음을 볼 수 있다.</li>
</ul>
<h1 id="domain-adaptation">Domain adaptation</h1>
<hr>
<ul>
<li>
<p>source, target domain 이 다른 경우 문제가 생긴다.</p>
<ul>
<li>domain shift가 발생한다.</li>
<li>이를 해결하기 위한 robust한 모델을 만드는 것</li>
</ul>
</li>
<li>
<p>한 쪽을 다른 쪽으로 조정하거나 맞추려는 것이 목적이다.</p>
</li>
</ul>
<h3 id="가정">가정</h3>
<ul>
<li>source domain은 class label이 있다고 가정</li>
<li>target domain은 class label이 없어도 됨</li>
</ul>
<h2 id="dann"><a href="https://arxiv.org/abs/1505.07818">DANN</a></h2>
<hr>
<p><a href="https://jaejunyoo.blogspot.com/2017/01/domain-adversarial-training-of-neural.html">참고</a></p>
<p><img src="https://3.bp.blogspot.com/-TmE6RmQRDz4/WIBTgu-aD6I/AAAAAAAABME/MMllHZmQAuoILywAX9JSdCI8Hd1-VGgbwCK4B/s400/DANN-shallow-model.png" alt=""></p>
<ul>
<li>Domain Discriminator가 구별하지 못하게 만듬</li>
</ul>
<img width="795" alt="image" src="https://user-images.githubusercontent.com/80192345/184464021-0fac6282-e71e-4b73-8e3a-1e7975df4412.png">
<p><a href="https://stats.stackexchange.com/questions/260272/what-is-difference-between-transfer-learning-and-domain-adaptation">https://stats.stackexchange.com/questions/260272/what-is-difference-between-transfer-learning-and-domain-adaptation</a></p>
<p><a href="https://lhw0772.medium.com/study-da-domain-adaptation-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-%EA%B8%B0%EB%B3%B8%ED%8E%B8-4af4ab63f871">https://lhw0772.medium.com/study-da-domain-adaptation-알아보기-기본편-4af4ab63f871</a></p>

