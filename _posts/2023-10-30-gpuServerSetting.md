---
title: gpu 서버 세팅
author: highcloud100
date: 2023-10-30 22:38:00 +0900
categories:
  - life
  - AI
tags: 
math: true
mermaid: true
render_with_liquid: false
---
---

- tmux로 gpu 세션 받아 유지함
- 아나콘다 설치
```bash
# 설치 파일을 다운로드할 경로로 이동(to home directory)
cd ~

# Anaconda 설치 파일 다운로드
wget https://repo.anaconda.com/archive/Anaconda3-2023.03-1-Linux-x86_64.sh

# 설치 파일 실행
bash Anaconda3-2020.11-Linux-x86_64.sh

# Anaconda 설치 후 재접속
exit

# conda 환경 생성 예시
conda create -n <환경 이름> python=<파이썬 버전>

# conda 환경 실행
conda activate <환경 이름>
```

- 파이썬 버전 붙여서 환경 만들면 pip 도 설치돼서, pip 경로가 가상환경 하위 디렉토리로 잡힌다. 
	- 격리된다. 

- 주피터에서 conda 환 쓰기 위해서 다음을 작업

```bash
pip install jupyter notebook
python -m ipykernel install --user 이름 --display-name "주피터에 보일 이름"
```

이후 실행
```
jupyter-notebook --no-browser --ip=<165.246.***.***> --port=<port number>
```

or jupyter-lab

---
- python 12버전은 torch가 아직 지원 안 한다.
	- 낮춰서 설치