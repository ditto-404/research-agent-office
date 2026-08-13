---
description: 기존 Research Team의 설정(이름/주제/키워드/검색조건 등)을 수정
argument-hint: <team-id>
---

너는 Research Office의 팀 관리 Agent다. 팀 id `$ARGUMENTS`의 설정을 수정한다.

## 절차

1. `data/teams.json`에서 `$ARGUMENTS`를 찾는다. 없으면 안내 후 중단. 찾은 항목의
   `type`이 `journal-watch`면 "저널/분야 설정은 `/journal-watch-setup`을 쓰라"고
   안내하고 중단한다 (id로 비교하지 않는다 - 고정 Journal Watch 팀의 id는 상황에
   따라 `journal-watch`가 아닐 수 있다, 예: `<journal-watch-id>`).
2. `data/{path}/config.json`을 읽어 현재 값을 사용자에게 보여준다.
3. 무엇을 바꾸고 싶은지 확인한다 (이름/주제/키워드/검색조건/papers_per_day 중 사용자가
   요청한 항목만 바꾼다. 요청하지 않은 항목은 그대로 둔다).
4. `config.json`을 갱신한다. `id`는 바꾸지 않는다 (slug 변경은 폴더 이름/teams.json
   path와도 얽혀 위험하므로, slug 자체를 바꾸고 싶다면 `/team-remove` 후 `/team-add`로
   새로 만들 것을 권한다).
5. 이름이 바뀌었으면 `data/teams.json`의 해당 항목 `name`도 함께 갱신하고, 그 팀의
   `wiki_path` (teams.json에서 확인) `/index.md` 제목도 갱신한다.
6. 변경 내역을 요약해 보고한다.
