---
description: Research Team을 제거 (기본은 등록만 해제, 데이터 폴더는 보존)
argument-hint: <team-id>
---

너는 Research Office의 팀 관리 Agent다. 팀 id `$ARGUMENTS`를 제거한다. 이 팀의 데이터는
두 곳에 있다는 점에 유의한다 - `data/teams/{id}/`(config/raw/my-research.md)와
`wiki_path`가 가리키는 `<WIKI_VAULT_ROOT>/{id}/`(위키 콘텐츠).

## 절차

1. `data/teams.json`에서 `$ARGUMENTS`를 찾는다. 없으면 안내 후 중단. 찾은 항목의
   `type`이 `journal-watch`면 "Journal Watch 타입 팀은 고정 팀이라 제거할 수 없다"고
   안내하고 중단한다 (id로 비교하지 않는다). `wiki_path` 값을 기억해둔다.
2. 해당 팀의 논문 수(`{wiki_path}/papers/` 개수)와 마지막 수집일을 보여주고, 정말
   제거할지 확인받는다.
3. 기본 동작: `data/teams.json`에서 이 팀 항목만 **삭제**(등록 해제)한다.
   `data/teams/{id}/` 폴더와 `{wiki_path}` 폴더는 **둘 다 그대로 둔다** (데이터 보존).
4. 완전 삭제(폴더까지 rm)는 사용자가 명시적으로 "폴더도 지워줘" 같이 요청했을 때만
   수행한다 - 이 경우 `data/teams/{id}/`와 `{wiki_path}` 둘 다 지울지 사용자에게 각각
   확인받는다 (하나만 지우고 싶어할 수도 있음). 되돌릴 수 없다는 점을 다시 한번
   확인받은 뒤 삭제한다.
5. 처리 결과를 보고한다 (등록만 해제했는지, 어느 폴더까지 삭제했는지 명확히). 폴더를
   보존했다면 그 팀의 `my-research.md`도 그대로 남아있다는 점을 함께 알린다.

## 지켜야 할 것

- 기본값은 항상 "보존" - 사용자가 명시적으로 폴더 삭제를 요청하지 않는 한 두 곳의
  데이터 모두 건드리지 않는다.
