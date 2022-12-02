---
layout: single
title: "Anaconda on EC2"
categories:
  - Dev
tags:
  - AWS
  - EC2
  - Anaconda
  - AMI
  - PyTorch
---

## 리눅스 AMI에서 설치

```bash
# 아나콘다 설치
sudo wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh

# 아나콘다 실행
source ~/.bashrc
```

- 터미널 프롬프트에 `(base)` 뜨는거 확인
- linux에서는 windows와 다르게 가상환경에서 `pip install` 사용해도 가상환경 안에 깔리는 것 확인
  - 인스턴스에 python을 깔지 않아서 그런 것일 수도 있음
- 모듈은 pip로 깔았는데 jupyter notebook으로 `{dns}:8888/?token` 접속해도 connection refused 에러남
  - 아래처럼 `/home/ec2-user/.jupyter/jupyter_notebook_config.py`에 jupyter 설정을 추가하면 됨
  ```python
  c.NotebookApp.ip='0.0.0.0'
  c.NotebookApp.allow_origin='*'
  ```
- conda 설치 후 채널을 추가해줘야 패키지를 찾음
  ```bash
  conda config --add channels conda-forge
  conda config --add channels bioconda
  ```
- torch는 실행할 때 gcc가 필요하기 때문에 따로 설치해줘야 함
  ```bash
  sudo yum install gcc
  ```
- opencv import할 때 `ImportError: libGL.so.1: cannot open shared object file: No such file or directory` 에러

  ```bash
  yum install mesa-libGL
  yum install mesa-libGL-devel
  ```

  - 관련 라이브러리 설치

- torch 설치하는 것보다 아예 Deep Learning AMI GPU PyTorch 1.12.1 (Amazon Linux 2) 20220928 같은 AMI를 사용하는게 효율적임
  - jupyter까지 설치되어 있는 conda 가상환경 제공

## EC2 instance 접속

### 맥에서 접속

1. pem키를 `~/.ssh/`로 복사
2. 권한 변경(600)
3. `~/.ssh/config` 생성

   ```bash
   # instance name
   HOST {instance name}
   	HostName {eip}
   	User ec2-user
   	IdentityFile {pem key path}
   ```

4. `config` 파일 권한 변경(700)
5. `ssh {instance name}`으로 실행

### 윈도우에서 접속

1. PUTTYgen에 pem 넣어서 ppk 생성
2. PUTTY에 Host : eip, Auth : ppk 파일 각각 넣고 프로파일 저장
3. 프로파일 로드 후 오픈

### 초기 설정

- user name은 ec2-user 등으로 설정
- 초기 비밀번호는 없기 때문에 `sudo passwd root`으로 설정해줘야 함

> 현재 cuda를 찾지 못함
> → cpu로 진행

- 서버로 파일 업로드/다운로드
  - 파일 업로드
    ```bash
    scp -i MyKeyFile.pem FileToUpload.pdf ubuntu@ec2-123-123-123-123.compute-1.amazonaws.com:FileToUpload.pdf
    ```
  - 파일 다운로드
    ```bash
    scp -i MyKeyFile.pem ubuntu@ec2-123-123-123-123.compute-1.amazonaws.com:FileToUpload.pdf FileToUpload.pdf
    ```
- opencv-python은 conda 모듈 저장소에 없어서 pip로 설치해야 함

## Anime-face-detector

- 샘플 데이터 인퍼런스
  - faster-rcnn : 31초
  - yolov3 : 33초

## Pupilganinf

- 샘플 데이터 인퍼런스(5개)
  - 8초
