---
layout: single
title: "블로그 업데이트(mmistakes pull 완료)"
categories:
  - Miscellaneous
tags:
  - Blogging
  - Git
---
- 처음 mmistakes를 `fork`해서 블로그를 만든 뒤 fetch를 하지않고 있다가 오늘 한꺼번에 모두 받았다. mmistakes의 master를 upstream으로 설정하고 다 받았는데 처음에 `_config.yml`이 다른 곳에 생겨서 위치가 바뀌었나 싶었다. 근데 예시를 보여주기 위해 docs안에 추가한 것이었다. 이외에 글자 크기 같은 버그 수정이나 docs를 추가한 것 같다.

- 받아올 저장소를 upstream으로 설정하고 master브랜치를 `fetch` 후 `git status`를 실행하면 파일 변화를 알려준다. 이중에 초록색으로 표시되는건 자동으로 커밋이 가능한 것들이고 맨 밑에 빨간 색이 conflict가 일어난 파일들이다. 앞에 `deleted by us: `이런 식으로 conflict가 일어난 이유를 알려준다. 저건 내 로컬에는 있지만 upstream에는 파일이 없다는 것이다. 파일을 적절하게 수정한 후 `add`해주면 된다. 다 해결한 다음에 `push`하면 끝난다.

- `fetch`하고 `rebase`로 커밋 하나하나 수정하는 방법도 있는데 다음번에 커밋이 작게 쌓여있을 때 이 방법을 시도해봐야겠다.

- `_sass`폴더 안에서 본문 폭이나 여백 조정같은게 가능하다.