---
layout: single
title: "\"사진 기반 여행지 추천\"을 플젝 주제로 선정하지 않은 이유"
categories:
  - Dev
tags:
  - Rekognition
  - Open API
  - Ideation
---

## 사진 기반 여행지 추천
- 주제 설명 : 사용자가 가고 싶은 여행지 사진을 업로드하면 비슷한 특징을 가진 여행지 몇 곳을 추천해준다.
- 사용 기술 : Amazon Rekognition
- 기능
  - 사용자가 업로드한 여행지 사진을 분석하여 특징 추출
  - 특징 기반 여행지 검색
- 준비 사항
  - 여행지 특징 기반 (태그, 유사도) 데이터

### 기각 이유
- (태그, 유사도) 데이터를 구할 수 없음
  - 원했던 태그는 도시 종류, 액티비티, 교통 등이 포함되어 광범위했음
  - 여행사들의 open API에서 제공하는 정보는 정제된 데이터였음
  - 여행지들에 대한 리뷰와 sentence transformer, rekognition을 사용하여 직접 만드는 것은 out of scope임
  - 조사해본 여행사 API들
    - Trip advisor api
      - location details(name, addr, rating, url)
      - location photos
      - location reviews
      - location search : 카테고리가 너무 제한적("hotels", "attractions", "restaurants", "geos")
      - nearby location search
    - Lonely planet api : 사용 중지됨
    - SK open API : 국내 여행지에 대하여 실제 이용객 경로 기반 통계 제공
    - Tour API : 지역별 관광지들의 관광 정보(행사, 숙소 등) 제공
- 아래 조건들이 만족되면 해볼만한 주제라고 생각함
  - 여행지들의 리뷰를 제공받거나 크롤링
  - 리뷰에서부터 도시 종류, 액티비티, 포함하는 자연 경관 등에 대한 정보를 키워드로 추출할 수 있도록 훈련된 NLP
  - 키워드에서부터 여행 관련 정보만 classification하여 위 데이터 생성