---
layout: single
title: "노션 그래프 업데이트"
categories:
  - Misc
tags:
  - Notion
  - plot
  - notioncharts.io
  - datajumbo.co
---

- notioncharts.io가 23년 5월 부로 서비스를 종료했다. 노션 DB를 사용해서 그래프를 만들어주는 웹 서비스들은 여러 개가 있었지만, 노션차트를 사용하던 이유는 아래와 같다.
  - 간단하게 DB 연동 가능
  - 무료로 최대 3개까지 그래프 생성 가능
  - 데이터 필터링
  - 그래프 디자인
- 다시 대체제들을 찾아보다가 datajumbo가 위 조건들을 대부분 만족하고 세밀한 그래프 조절 기능을 제공해줘서 사용하게 되었다.
  - 단순히 x-y 관계 뿐만 아니라 범례를 생성할 수 있다.

![datajumbo introduction](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/datajumbo-introduction.PNG)

- `Configuration` : 차트 설정
- `Source` : DB 선택
- DB의 속성들 중에 x, y-axis를 선택하고 DB에서 설정한 속성에 따라서 세부 설정을 할 수 있다.
  - `Bucket by` : y 값 계산 방법 선택
  - `Split by` : y-x 함수로 계산된 값을 다른 속성으로 나눌 수 있는 기능이다.
    - 예를 들어 위 그림에서는 25일의 소요 시간의 총 합이 10인데 각자 다른 종류를 2, 3, 5씩 포함하고 있다면, 2, 3, 5의 색을 다르게 표현할 수 있다.
    - 이때 선택한 속성은 범례로 표현 가능하다
  - `Filter` : 선택한 DB에서 그래프로 가져올 데이터를 설정할 수 있다.
    - 속성의 타입에 따라 여러 가지 필터가 존재한다.
- `Customization` : 그래프 계산 이외의 디자인 설정
  - `Style Picker` : 그래프 테마
  - `General`, `Title` : 조절해보면 알 수 있음
  - `Cells` : 이전에 `Split by`로 선택한 속성에 관한 설정
  - `Axis` : 축 관련 설정
  - `Grid` : 격자 무늬 표시
  - `Legend` : 범례 표시
