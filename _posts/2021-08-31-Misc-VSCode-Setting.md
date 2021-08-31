---
layout: single
title: "VS Code로 Mac, Windows에서 Jekyll 사용하기"
categories:
  - Miscellaneous
tags:
  - Blogging
  - Jekyll
---

## Mac
1. homebrew 설치
    - homebrew가 ruby 기반이기 때문에 homebrew를 설치하면 ruby도 같이 깔림
2. homebrew로 rbenv 설치
    - ruby 경로가 없다고 나오면 ~/.bash_profile에 환경변수 추가
    - `which ruby` 실행해서 나오는 경로를 아래 `<path>`에 넣어야 함  
        `export PATH="<path>"`
3. rbenv에서 ruby 다시 설치하고 bundler, jekyll 설치  
    - rbenv는 여러 버전의 ruby를 오가며 개발할 수 있도록 ruby의 버전을 설정해주는 도구임  
        따라서 rbenv를 설치하고 rbenv로 다시 특정 ruby 버전을 설치해줘야 함
    
    webrick 없다고 나오면 `gem install webrick`, `bundle add webrick`을 순서대로 실행하면 됨

## 윈도우
1. ruby 홈페이지에서 ruby installer 설치  
    마지막에 ridk 설치 체크하고 실행해야 함!
2. ridk는 1, 3 순서대로 설치함  
    1에서 잘 안되면 다시 실행하는데, 미리 start command prompt with ruby를 관리자 권한으로 열어놓아야 함(시작메뉴에 shortcut 만들기 체크 해제했다면 C드라이브 들어가서 찾아야 함)

## 설치한 후
`ruby -v`로 설치 확인하고 `gem install jekyll bundler` 실행

`bundle exec jekyll serve`로 실행해보고 하라는거 나오면 그대로 하면 됨  
(`bundle install` 등 dependency 관련)

> ※ `bundle exec jekyll serve`는 `_config.yml` 파일이 있는 위치에서 실행해야 함!