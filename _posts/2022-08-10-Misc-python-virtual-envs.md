---
layout: single
title: "Python virtual environments 관리"
categories:
  - Miscellaneous
tags:
  - venv
  - Anaconda
  - pip
---

# Anaconda virtual environments

Anaconda prompt에서는 `conda env ~` 명령어를 이용해서 원하는 python 버전으로 가상 환경을 생성할 수 있다.

같은 환경에서 여러 개의 프로젝트를 진행할 경우, python 버전이나 다른 패키지끼리의 의존성 문제로 conflict가 일어날 수 있다. 이런 충돌을 방지하기 위해 virtual environments를 생성해서, 프로젝트 각각에 호환되는 환경을 만들 수 있다.

아래는 가상 환경에 관련된 anaconda 명령어를 정리한 표이다.

| code                                        | description                                                                     |
| ------------------------------------------- | ------------------------------------------------------------------------------- |
| `conda env list`                            | 현재 존재하는 모든 환경 표시                                                    |
| `conda info`                                | 현재 실행중인 환경을 확인                                                       |
| `conda create[ -n envName pkgs]`            | -n envName 옵션을 이용하면 envName이라는 이름으로 가상환경을 생성               |
| `conda env remove -n envName`               | envName 가상환경 삭제                                                           |
| `conda activate envName`                    | envName 가상환경 실행                                                           |
| `conda deactivate`                          | 현재 실행중인 가상환경을 종료<br>(base환경으로 돌아옴)                          |
| `conda create --name envNew --clone envOld` | envOld를 복제해서 envNew 가상환경 생성<br>(envOld의 패키지들이 동일하게 설치됨) |

## `pip install`과 `conda install`의 차이

### `conda install`

- 현재 실행중인 환경이 `base`일 경우
  `C:\Users\user\anaconda3\Lib\site-packages`
- `base` 이외의 환경일 경우
  `C:\Users\user\anaconda3\envs\envName\Lib\site-packages`
  에 설치됨

### `pip install`

- `conda install`과 비슷함
- `--user` 옵션을 사용할 경우
  - `site.USER_SITE`에 설치됨
  - `USER_SITE`의 경로는 `python -m site`로 확인 가능
    윈도우 기준:
    `AppData\Roaming\Python\Python39\site-packages`

_어차피 아나콘다로 실행하는 가상환경이니까 웬만하면 `conda install`을 사용하자_

## 가상환경 복제하기

위 표에서 `--clone` 옵션을 이용해서 기존의 환경을 복제할 수도 있지만, 현재 환경에 설치된 패키지들을 공유해야 할 경우도 존재한다. 이럴 때는 `conda list`나 `pip freeze`로 현재 환경에 관련된 패키지들을 가져올 수 있다.

### `conda list`

- 현재 실행중인 환경이 `base`일 경우
  `C:\Users\user\anaconda3\pkgs` 내부의 패키지 나열
- `base` 이외의 환경일 경우
  `C:\Users\user\anaconda3\envs\envName` 내부의 패키지 나열
- 현재 환경에 설치된 패키지들이 나열됨
  → `conda list`에 나오는 패키지들은 현재 환경에서 사용 가능 - `USER_SITE`에 존재하는 패키지들은 나열되지 않음
- 아래 코드를 이용해서 현재 가상환경의 패키지를 export할 수 있다.

  ```bash
  conda list —export > requirements.txt

  conda install —-yes --file requirements.txt
  ```

  - `--yes` : 설치하면서 자동으로 `y` 입력

### `pip freeze`

- `pip`로 설치된 패키지들을 나열
  `USER_SITE` 포함
  → `pip freeze`에는 출력되지만, 현재 환경에서 사용하지 못하는 패키지가 존재할 수 있음

$e.g.$ `base`에서 `--user` 옵션을 이용해서 패키지 `A`를 설치하고, 동일 버전의 파이썬으로 가상환경 `venv1`을 생성
`venv1`에서 `pip freeze`를 실행하면 `USER_SITE`에 설치된 패키지 `A`가 출력됨
하지만 `venv1`에서 `A`를 import할 수 없음

- 아래 코드를 이용해서 현재 가상환경의 패키지를 export할 수 있다.

  ```bash
  pip freeze > requirements.txt

  pip install -r requirements.txt
  ```

### 비교

1. `conda list`는 `python`의 버전과 같은 기본적인 내용도 출력됨
   `pip freeze`는 package management를 위한 패키지( `pip`, `wheel`, `setuptools` 등)은 포함하지 않음(`pip list`는 포함함)
2. `pip freeze`는 아예 다른 곳에서 패키지 목록을 읽어들이는 것 같음
   1. `USER_SITE`에는 패키지 5개 정도 설치됨
   2. `base`에서는 `conda list`가 `pip freeze`보다 100개 정도 더 많음
      `conda list`는 380, `pip freeze`는 260 정도
   3. python 버전만 base와 동일하게 맞추고, 아무 패키지를 설치하지 않은 가상환경에서
      `conda list`는 package management 패키지 포함 10개 정도
      `pip freeze`는 `USER_SITE`에 설치된 패키지(5개)만 출력됨
   4. 환경마다 python의 위치가 다른 것은 당연한데, `base`에서 `conda list`와 `pip freeze`의 차이가 왜 이렇게 큰지 모르겠다.
3. 확실하게 동일한 가상환경을 만드려면 `conda list`를 사용하는게 나을 것 같다.
   `USER_SITE`에 설치된 패키지는 `venv1`에서 왜 안되는지도 의문이다.
   내 환경이 python과 anaconda를 모두 설치한 상태라서 뭔가 꼬인 것 같다.

# `jupyter notebook`이 정상적으로 실행되지 않을 때

- anaconda prompt에서 `jupyter notebook`를 실행할 때 아래와 같은 에러가 발생할 수 있다.
  `Bad config encountered during initialization: No such notebook dir: ‘C:/Users/user/...’`
- jupyter config 파일(`C:\user\usr\.jupyter\jupyter_notebook-config.py`)에서 주소를 고치거나 주석처리하면 된다.

---

# Python venv

venv를 사용해서 가상환경을 만들 때는 아나콘다와 같이 가상환경을 생성하는 명령에서 파이썬 버전을 지정하지 못한다. venv가 실행되는 파이썬 버전이 자동으로 설치되기 때문에, 원하는 버전의 파이썬으로 가상환경을 구축하기 위해선 그 버전의 파이썬을 설치해야 한다.

이때문에 여러 버전의 파이썬을 컴퓨터에 설치해야 하는 부담이 있지만, 여러 장점이 존재한다.

- conda는 jupyter notebook을 매번 실행시켜야 함(venv도 매번 activate해주는 건 마찬가지이다)
- VSCode에서 작업 가능
  - matlab처럼 variables를 볼 수 있음
  - ipynb파일을 실행시킬 커널을 편리하게 선택할 수 있음(경로 내 가상환경을 생성했을 경우 자동으로 커널 옵션에 추가됨)

## python 버전 관리

기본으로 실행되는 파이썬 버전은 환경 변수 설정을 통해 바꿀 수 있다.  
환경 변수 창에서 사용자 변수는 건드리지 않고 시스템 변수만 아래와 같이 추가하면 된다

- 변수 : PYTHON  
  값 : `C:\Users\user\AppData\Local\Programs\Python\Python39`
- 변수 : Path  
  값 : `%PYTHON%` 추가

> 아나콘다를 설치하면서 빨간 글씨로 <span style="color:red">Not recommended</span>라고 뜨는 옵션과 그 밑에 있는 옵션을 체크했을 경우 `C:/ProgramData/Anaconda3`에 설치된 파이썬이 기본 버전으로 설정되는데, 아나콘다를 지우면 해결 가능하다.  
> 아나콘다를 지울 수 없는 경우 시스템 변수에서 아나콘다의 경로를 지우면 된다.

## VSCode에서의 ipynb 파일 실행시 주의할 점

- `!pip~`와 같은 명령어들은 ipynb 파일을 실행하고 있는 커널과 관계없이 시스템에서 사용하는 파이썬 기본 버전을 사용한다. 따라서 가상환경에서 돌리고 있는 경우 패키지 설치는 따로 cmd 창을 켜서 설치해야 한다.
