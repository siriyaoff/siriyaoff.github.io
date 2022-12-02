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
