---
layout: single
title: "AM 프로젝트 및 Agile 개발"
categories:
  - Dev
tags:
  - Agile
  - Scrum
  - Sprint
  - Userstory
---

## Agile 개발

- Agile : 점진적으로 제품을 개발하는 기법들
- Agile Manifesto
  - 의사소통(>프로세스, 도구), 동작하는 소프트웨어(>문서), 고객과의 협력(>계약 협상), 변화에 대응

![daily-scrum](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/am-daily-scrum.png)

- Backlog Refinement : 스프린트 진행하는 동안에도 상시 이루어짐
- Sprint Review : PO와 함께 개발 완료된 내용의 데모 시연
- Sprint Retrospective : 회고
- Sprint Planning : 다음 sprint에서 수행할 작업 계획, user story estimation 측정

### User Story

![userstory](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/am-userstory.png)

- Theme : 추상화된 개발 대상
- Epic : Theme 내 분리 가능한 내용
- User Story : 사용 시나리오를 동반하는 개발 단위
  - 사용자 관점에서 제품이 전달하는 기능에 대한 비공식적, 일반적인 설명

### User story 나누기

- 초기 백로그는 범위가 넓고 세부 사항이 적음
- PO, SE 등의 대화를 거쳐 더 작고 자세한 백로그로 refine됨
  - 자세한 요구사항을 미리 다듬는데 많은 시간과 비용을 들이면 안됨
- 백로그에 user story를 많이 사용함
- INVEST 원칙 : User story를 나누는 기준
  - Independent : 실용적인 범위 내에서 독립적 또는 느슨한 정도로 연결
  - Negotiable : 스토리 세부 사항은 협상이 가능
  - Valuable : 고객 또는 사용자에게 가치가 있어야 함
  - Estimatable : 제품을 설계, 제작, 테스트할 팀에 의해 추정 가능
  - Small(사이즈 적합성) : 알맞은 크기
  - Testable : 관련된 테스트를 T/F 형식으로 테스트 가능
- 비기능적 요구사항
  - ‘웹브라우저는 chrome, edge, safari를 지원해야 한다’ 등의 제품 전반에 걸쳐 적용되어야 하는 비기능적 요구사항은 user story로 도출하는 것보다 Definition of Done에 정의하는게 좋음
- User story Template
  ```
  As a 상품기획자,
  I want 검색 결과 상세에서 내가 저장한 북마크 정보를 북마크 바로가기를 \
  통해 보여지는 메뉴에서 동일하게 확인 가능하다.
  So that 내가 저장한 북마크 정보가 정확히 저장되어 있는지 보기 위해서,
  Note:
  Out of Scope:
  Acceptance Criteria:
  ```
  - `As a` : 기능을 사용할 최종 사용자(role) 명시
  - `I want` : 개발할 기능(goal) 명시
  - `So that` : 개발할 기능을 통해 최종 사용자가 얻고자 하는 가치(benefit) 명시
  - `Note` : 개발에 필요한 사전 조건, 알아야 할 사항 등의 내용을 명시
  - `Out of Scope` : 해당 User Story가 개발 시 고려하지 않아도 되는 사항을 명시
  - `Acceptance Criteria` : 해당 User Story가 개발 완료되었다는 것을 확인 시켜줄 수 있는 기준을 명시

### Story Ceremony

- Backlog Refinement
- 회고 : EasyRetro
- 스토리 포인트 : planitpoker
