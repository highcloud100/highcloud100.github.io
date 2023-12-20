---
title: SLS sat solver 서칭
author: highcloud100
date: 2023-12-18 10:46:00 +0900
categories:
  - CS
  - algortihm
tags:
  - sls
  - 연구실
  - sat
math: true
mermaid: true
render_with_liquid: false
---
---

## Improving two-mode algorithm via probabilistic selection for solving satisfiability problem

![[assets/img/Pasted image 20231218133813.png]]

- **Sattime** : i C, Li Y. Satisfying versus falsifying in local search for satisfiability. In: Proceedings of the 15th International Conference on Theory and Applications of Satisfiability Testing–SAT 2012, Trento, Italy, 2012, 477-478.
- **dimetheus** : https://scholar.google.com/scholar_lookup?title=Sat%20solving%20with%20message%20passing&author=O.%20Gableske&publication_year=2016
- **CCAnr** : S . Cai, C. Luo, K. Su, CCAnr: A configuration checking based local search solver for non-random satisfiability. InProceedings of the 18th International Conference on Theory and Applications of Satisfiability Testing–SAT 2015, USA, 2015, 1-8.
- **CCAnr  + CNC** : . Cai, C. Luo, X. Zhang, J. Zhang, Improving local search for structured SAT formulas via unit propagation based construct and cut initialization (short paper). In: Proceedings of the 27th International Conference on Principles and Practice of Constraint Programming (CP 2021), Saarland, 2021, 5:1-5:10.


SLS algorithm은 두 가지로 나뉜다. 
- two-mode SLS solvers : greedy mode and diversification mode 
- focused local search

#### focused local search
- WalkSAT, WalkSATlm, ProbSAT
- walkSATlm : S. Cai, C. Luo, K. Su, Improving walksat by effective tie-breaking and efficient implementation, Comput. J. 58 (11) (2015) 2864 –2875.

#### Two mode
- greedy mode : select the flipping variable in a global local search 
- diversification : explores the search space, evade local optima

- typically combination of deterministic, stochastic


## A survey of SAT solver : Weiwei Gong and Xu Zhou

- sparrow 
- probSAT

#### Incomplete Algorithms parallelism
PGSAT 
- is a parallel version of GSAT. In this algorithm, the variables are divided into subsets, which are assigned to different processors.
- "A. Roli, “Criticality and parallelism in structured sat instances,” in International Conference on Principles and Practice of Constraint Programming (Springer, 2002), pp. 714–719."

PGWSAT 
- improves PGSAT, which randomly selects a variable to be flipped in an unsatisfiable clause with a certain probability during PGSAT execution. 
- "A. Roli, M. Blesa, and C. Blum, (2005)."

## Proceedings of SAT Competition 2023 : Solver, Benchmark and Proof Checker Descriptions

Yallin
- A Linear Weight Transfer Rule for Local Search ,  Md Solimul Chowdhury , Cayden R. Codel , and Marijn J.H. Heule

- “We implemented our modifications to ddfw on top of the solver yalsat.” 
- “Dynamic local search (DLS) algorithms are SLS algorithms that assign a weight to each clause.” 



---
## 정리 
- Improving stochastic local search for uniform k-SAT by generating appropriate initial assignment, Huimin Fu, Wuyang Zhang, Guanfeng Wu, Yang Xu, Jun Liu

random track of SAT competitions

- Sparrow : 2011 // 19
	- Balint A, Fröhlich A. Improving stochastic local search for SAT with a new probability distribution. Paper presented at: Proceedings of the SAT-10,Edinburgh, Scotland; 2010:10-15.
- ProbSat : 2013 // 20, 21
- CCASat : 2012 // 6
	- Cai S, Su K. Local search for boolean satisfiability with configuration checking and subscore. Artif Intell. 2013;204:75-98.
- FrwCB // 15
	- Luo C, Cai S, Wu W, Su K. Focused random walk with configuration checking and break minimum for satisfiability. Paper presented at: Proceedings of the International Conference on Principles and Practice of Constraint Programming,Berlin, Heidelberg; 2013:481-496.
- WalkSATlm // 22
	- Cai S, Luo C, Su K. Improving WalkSAT by effective tie-breaking and efficient implementation. Comput J. 2015;58:2864-2875.

- DCCASat // 16
	- Luo C, Cai S, Wu W, Su K. Double configuration checking in stochastic local search for satisfiability. Paper presented at: Proceedings of the AAAI-14,Québec City, Québec, Canada; 2014:2703-2709.
- CSCCSat : silver 2016 // 23
	- Luo C, Cai S, Wu W, Su K. CSCCSat2014: Solver description. Paper presented at: Proceedings of the SAT-2014,Vienna, Austria; 2014:25-26.
- dimetheus : 2016 // 10
	- Gableske O. Dimetheus: solver description. Paper presented at: Proceedings of the SAT- 2016,Bordeaux, France; 2016:37-38.
- Score_2 SAT : bronze 2017 // 24
		- Cai S, Luo C. Score2SAT: solver description. Paper presented at: Proceedings of the SAT-2017,Melbourne, Australia; 2017:34.
- yalsat : 2017 // 25

 
- CSCCSat
	- combination of two SLS solvers 
		 - FrwCB
		 - DCCASat
		 - Based on the clause-to-variable ratio of the instance, pick from 2 to solve
	- It won the “3rd Place Award” in the random SAT track of SAT Competition 2014
	- the “2nd Place Award” in the random SAT track of SAT Competition 2016.

- Score_2 SAT
	- combination of two SLS solvers
		- DCCASat
		- WalkSATlm
	- won the “3rd Place Award” in the random SAT track of SAT Competition 2017.

- Sparrow 
	- winner of the SAT Competition 2011 category random SAT.
	- based on gNovelty+



## SLS on H/w

- A. McDonald, Parallel walksat with clause learning. data analysis project papers, 2009
	- WalkSAT을 스레드로 병렬화한 논문, 각 스레드는 다른 초기 값을 가진다.
	

- Accelerating an FPGA-Based SAT Solver by Software and Hardware Co-design
	- https://ietresearch.onlinelibrary.wiley.com/doi/pdfdirect/10.1049/cje.2019.06015

- H. Deleau, C. Jaillet, and M. Krajecki, “Gpu4sat: solving the sat problem on gpu,” in PARA 2008 9th International Workshop on State–of–the–Art in Scientific and Parallel Computing, Trondheim, Norway (2008).
	- https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=807f4b8069782bba68ca2bfb4ad77ef56f8cf30f
	- SAT problem instance를 행렬로 표현한 뒤 incomplete methods를 이용하여 푼 논문