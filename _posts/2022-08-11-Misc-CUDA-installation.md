---
layout: single
title: "CUDA 환경 구축"
categories:
  - Miscellaneous
tags:
  - CUDA
  - Tensorflow
  - PyTorch
---

- CUDA, cuDNN, 가상환경, ML framework 순으로 설치
- 아래는 tiny-cuda-nn을 설치하기 위해서 vscode, choco, cmake가 포함되어 있음

## 1. CUDA, cuDNN 설치

| GPU   | 3070 laptop |
| ----- | ----------- |
| CUDA  | 11.5        |
| cuDNN | 8.3.3       |

- cmd에 `nvcc -V` 실행하면 적절한 CUDA 버전 알려줌
- `nvidia-smi` 실행하면 드라이버 버전 확인 가능
- [GPU-CUDA 호환성](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)
- [Cuda 다운로드](https://developer.nvidia.com/cuda-toolkit-archive)
- [cuDNN 다운로드](https://developer.nvidia.com/rdp/cudnn-archive)

## 2. 가상환경 구축

[Python virtual environments 관리](https://siriyaoff.github.io/miscellaneous/Misc-python-virtual-envs.md)

- git bash에서 venv 실행하려면 `Scripts/activate` 실행
  CMD에서 실행하려면 `Scripts/activate.bat` 실행
- choco로 cmake 설치하면 일부 인식문제 생길 수 있음

## 3. vscode, choco, cmake 설치

[Visual Studio Code에서 CMake 환경 설정하기](https://evandde.github.io/vscode-cmake/)

## 4. pytorch, tiny-cuda-nn 설치

- [CUDA-cuDNN-python-tensorflow-gpu 호환성](https://www.tensorflow.org/install/source_windows#tested_build_configurations)
- [tensorflow-gpu 패키지](https://anaconda.org/search?q=platform%3Awin-64+tensorflow-gpu)가 없을 수 있기 때문에 먼저 확인하는게 나음
- [버전 정리된 블로그](https://velog.io/@boom109/Conda-Env-Tensorflow-Pytorch-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%84%B1) 참고하면 좋음

- torch  
  [PyTorch](https://pytorch.org/get-started/locally/)
  - [https://download.pytorch.org/whl/cu115/torch_stable.html](https://download.pytorch.org/whl/cu115/torch_stable.html)에서 whl파일 받은 후 설치(cu다음이 쿠다 버전으로, 변경 가능)
  ```python
  !pip install torch-1.11.0+cu115-cp39-cp39-win_amd64.whl
  !pip install torchvision-0.12.0+cu115-cp39-cp39-win_amd64.whl
  ```
  - CUDA 버전이 10 이하면 [https://pytorch.org/get-started/previous-versions/](https://pytorch.org/get-started/previous-versions/) 참조
- tiny-cuda-nn
  [https://github.com/NVlabs/tiny-cuda-nn](https://github.com/NVlabs/tiny-cuda-nn)

## Hair segmentation 모델 테스트

github repo가 공개된 논문 + github에서 hair segmentation 키워드로 검색한 레포들 중 실행 가능한 것들만 테스트함

- [pytorch-hair-segmentation](https://github.com/YBIGTA/pytorch-hair-segmentation)
  - 사람 머리카락과 비슷하게 머릿결이 표현된 경우 인식되지만, 단색으로 칠해졌을 때는 인식이 아예 안됨
  - 데이터를 넣어줄 때 마스크도 같이 넣어줘야 함  
    Anime-face-detector의 예시 사진으로 테스트한 결과  
    ![hair segmentation1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/hair-segmentation1.png)
- [fm.PyTorch](https://github.com/ash11sh/fm.PyTorch)
  - 사람 얼굴 넣으면 readme에 나오는 정도로 결과가 나오지만 컨셉아트나 애니메이션 이미지 넣으면 no face detected 뜨면서 아예 인식이 안됨
- Two-stage human hair segmentation in the wild using deep shape prior
  - 깃헙 없음
- Hair Segmentation on Time-of-Flight RGBD Images
  - ToF라는 특수한 데이터 기반으로 진행됨
- [Hair-Detection](https://github.com/Papich23691/Hair-Detection)
  - requirements.txt 설치할 때 버전 에러
- [https://downloads.hindawi.com/journals/tswj/2014/748634.pdf](https://downloads.hindawi.com/journals/tswj/2014/748634.pdf)
  - 사람 머리카락 → caricature용 머리
  - 알고리즘(머신러닝 x)
- [hair-segmentation](https://github.com/thangtran480/hair-segmentation)
  - fm.PyTorch와 마찬가지로 face detect 과정 포함
  - face detection 과정 없애고 진행하도록 수정하면 결과가 pytorch-hair-segmentation과 비슷한 수준으로 나옴  
    Anime-face-detector의 예시 사진으로 테스트한 결과  
    ![hair segmentation2](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/hair-segmentation2.png)
