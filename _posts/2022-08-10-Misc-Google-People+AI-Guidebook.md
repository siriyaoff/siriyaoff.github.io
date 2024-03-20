---
layout: single
title: "Google People + AI Guidebook 간단 요약"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Miscellaneous
tags:
  - CT
---

# 문화지능

- AI가 도움이 되려면 Human-Centered AI가 되어야 함(+HCI)
- AI product:
  - 설계, 문제 정의, 사용할 알고리즘, 테스트, UI를 통해 상호작용
    ⇒ 사람이 수행함
- Challenge:
  - 어떻게 human-centered AI products를 설계할까?
  - 어떻게 윤리적 이슈와 같은 문제를 해결할 수 있을까?

# Ch 1. User Needs + Defining Success

- 단순히 알고리즘 성능을 측정하는 것을 넘어서 사용자의 만족감 또한 Human-Centered AI의 측정 대상이 됨
- 사실 AI product는 연산만 AI가 하고 문제 정의부터 데이터 준비, 결과 사용은 사람들이 함
- 모델 성능이 아무리 좋아도 user need를 충족시키지 못하면 실패함
- 사람이 맞춰지면 안되고, 사람 중심으로 ai를 적용해야 함

## Key Considerations

### 1. user needs와 AI strengths 간에 교차점 찾기

- ai가 해결할 수 있는 방법으로 실제 문제 풀기
- ai가 unique value를 제공할 수 있어야 함
- 문제 정의 → ai 사용할 곳 찾기 → worflow 그리기

### 2. automation / augmentation 평가하기

- automate task : 어려움, 지루함, 반복적, 규모가 큼
- augment task : 재밌음, 결과에 대한 신용 필요, 창의성 필요, 작업 하면서 성장 가능

### 3. reward function 설계하기

- reward function( = objective function, loss function) : AI가 성공과 실패를 정의하는 방법
  - true positive : 반응 해야하는 것을 반응
  - true negative : 반응하지 말아야 하는 것을 반응하지 않음
  - false positive : 반응하지 말아야 하는 것을 반응
  - false negative : 반응 해야하는 것을 반응하지 않음
- precision : true/false positive 중에 true positive의 비율(prediction이 positive임)
- recall : true positive, false negative 중에 true positive의 비율(reference가 positive임)

# Ch 2. Data Collection + Evaluation

- 모델 train에 사용되는 데이터셋 추출, 평가 방법

## Key Considerations

### 1. Plan to gather high-quality data from the start

- Data cascades : 데이터 질이 안좋으면 개발 전체에 영향을 미침

### 2. Translate user needs into data needs

- 데이터 전처리도 중요함
  - PII(Personally Identifiable Information)이 없어야 함
  - 레이블 해야하는 데이터가 적어야 함
  - under-fitting, over-fitting을 조심해야 함
  - 고정관념이나 편견과 같은 요소에 관한 처리도 필요함
    → 데이터에 있는 bias를 잘 제거해야 함

### 3. Source your data responsibly

- Use existing datasets
  - 데이터 전처리에 시간이 많이 소요될 수 있음
- Building your own dataset
  - 레이블링을 신경써야 함
- 데이터가 시간이나 환경이 변함에 따라 영향이 있는지 알아야 함

### 4. Prepare and document your data

- 잘못된 데이터 조절
- missing value 처리

### 5. Design for labelers & labeling

- labeler들을 통해 labeling을 완료할 수 있음
- labeler들은 end user와 비슷한 사람들이어야 함
  - 문화적 차이같은거 고려해야 함
- labeling tool을 설계하는게 좋음

### 6. Tune your model

- tunning : training 단계에서 hyperparameter이나 reward function을 조절하는 것 또는 training data 수정

# Ch 3. Mental Models

- ai model의 어떤 점을 user들에게 설명해야 할까?
- mental model : product의 동작에 대한 이해
  - Product가 무엇을 할 수 있는지 사람이 예상할 수 있게 만듦
    → 적절한 사용 가능

## Key Considerations

### 1. Set expectations for adaptation

- 사용자가 어떤 일에 product를 사용하는지 고려해서 step-by-step 과정을 생각해야 함

### 2. Onboard in stages

- product가 어떤 것을 할 수 있는지, 개선을 위해 피드백이 필요한 점 등을 설명해야 함
- 첫 실험에서 ‘추천을 위해 데이터를 측정할 것이다’와 같이 초기 경험에 관해서 설명하는게 좋음

### 3. Plan for co-learning

- product가 사용되면서 training을 계속 해서 더 나아질 수 있음
  - 사용자 경험이 개선됨
  - mental model을 수정해야 할 수도 있음
- Fail gracefully
  - 데이터를 모두 수집하지 못한 경우 수동으로 채울 수 있도록 설정

### 4. Account for user expectations of human-like interaction

- ai model을 의인화해서 ux 개선 가능
  - 이때 product의 성능과 한계를 밝혀야 사용자들의 신뢰도 올라감

# Ch 4. Explainability + Trust

- explainable ai : prediction에 대한 근거를 설명할 수 있는 ai
  - ai confidence를 보여주는 방법을 고려해야 함
  - 상위 모델로 responsible ai가 있음
- user trust를 위해선 아래와 같은 항목이 필요함
  - ability : ai의 숙련도
  - reliability : 결과의 일관성
  - benevolence : ai에 대한 긍정적 이미지

## Key Considerations

### 1. Help users calibrate their trust

- 사용자들은 언제 product를 믿고 언제 직접 판단해야 하는지 알아야 함
- 판단 근거인 data source에 관해 분명히 말해야 함

### 2. Plan for trust calibration throughout the product experience

- product를 사용하면서 순간순간에 설명을 해줘서 user trust를 조절해야 함

### 3. Optimize for understanding

- 시스템에 관한 설명이나 결과에 관한 설명 필요
  - data source(판단 근거 표시)
  - model confidence(옆에 확률 같이 표시)
  - example-based explanations(avocado 그리기)
  - explanation via interaction(IDE tooltip)

### 4. Manage influence on user decisions

- model confidence을 보여줄지 결정해야 함(confidence level이 강렬하지 않을 경우)
- categorical, N-best alternatives, numeric 등으로 model confidence를 나타낼 수 있음
  - categorical : 질문의 주제에 관해 다시 물어봄
  - n-best : 연관있는거 몇개 제시
  - numeric : 유사도와 같이 표시

# Ch 5. Feedback + Control

## Key Considerations

### 1. Align feedback with model improvement

- explicit feedback : 모델 개선에 도움을 준다하고 사용자로부터 의견 받음
- implicit feedback : 사용자가 실제로 행동한 데이터 받음

### 2. Communicate value & time to impact

- 사용자는 왜 피드백을 하는가
  - 물질적 보상
  - 뱃지 등의 상징적 보상
  - 성능 개선
  - 커뮤니티 만들어서 서로 도와주도록
  - 자기만족감

### 3. Balance control & automation

- task가 간단하고 흥미가 있을 때 피드백을 주기 때문에 피드백 얻는 과정을 잘 조절해야 함

# Ch 6. Errors + Graceful Failure

- 복잡한 모델에서 어떻게 에러를 유발한 원인을 찾을 수 있을까?
- 에러가 발생한 다음 사용자들이 계속 사용하도록 놔둬도 괜찮을까?

## Key Considerations

### 1. Define “errors” & “failure”

- errors
  - user errors, system errors
  - context errors : user나 system 상의 에러는 없지만 잘못된 설명으로 발생하는 에러
    사용자의 mental model이 잘못 설계된 경우임
- 사용자의 mental model에 따라서 60%의 정확도를 실패로 받아들일 수도 있음

### 2. Identify error sources

- 데이터 문제
- 모델의 설계
- background error(시스템이 제대로 동작하지 않았지만, 사용자나 시스템이 에러를 인지하지 못한 경우)

### 3. Provide paths forward from failure

- 피드백을 줄 수 있는 기회 부여
  ready to run? (ok/not now) : not now를 붙여서 거절하는 option 주면서 mental model 형성을 도움
- 필요한 피드백만 하기
  - 오류에 대한 피드백은 pain point를 명확히 알 수 있도록 해야 함

visualdata.io
