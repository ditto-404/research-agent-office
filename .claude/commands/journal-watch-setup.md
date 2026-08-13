---
description: Journal Watch 타입 팀의 저널 목록/수집 조건을 설정·수정
---

너는 Research Office의 팀 관리 Agent다. 고정 Journal Watch 타입 팀의 `config.json`을
편집한다.

## 절차

1. `data/teams.json`에서 `type`이 `journal-watch`인 팀을 찾는다 (항상 정확히 1개).
   그 항목의 `id`와 `path`를 기억해둔다 - id를 하드코딩하지 말고 항상
   `teams.json`에서 찾은 값을 쓴다.
2. `data/{path}/config.json`을 읽어 현재 `field`와 저널 목록을 보여준다.
3. 사용자에게 다음을 확인한다:
   - `field`를 바꾸고 싶은지 (특정 세부 주제 없이 폭넓게 훑을 분야)
   - 추가/삭제/수정할 저널이 있는지 (있으면: 이름, 홈페이지 URL 또는 RSS 피드 URL,
     비고. **없어도 된다** - 저널을 지정하지 않으면 `/paper-collect`가 저널에
     국한하지 않고 전체 검색하되, `schema/schema.md`의 "저널 우선순위" 규칙대로
     `field` 기준으로 impact factor가 높거나 저명한 저널 위주로 후보를 추린다.)
   - 하루 수집 목표 편수를 바꾸고 싶은지 (기본 3~4편)
4. `journals` 배열을 갱신한다 (비워둬도 유효한 상태). 각 항목은
   `{"name": "...", "url_or_feed": "...", "notes": "..."}` 형식.
5. 변경 후 목록을 다시 보여주고, `/paper-collect {id}`로 바로 수집을 시작할지 물어본다.

## 참고

- 저널 웹사이트에 RSS가 있으면 `url_or_feed`에 RSS URL을 쓰는 게 수집 정확도가 높다.
  없으면 저널 최신호 페이지 URL을 쓰고 `notes`에 "RSS 없음, 웹페이지 직접 확인"이라고
  적어둔다.
- `journals`를 비워둔 채로 `/paper-collect {id}`를 실행하면, 어떤 분야를 기준으로
  저명한 저널을 판단할지가 필요하다 - `config.json`의 `field`가 있으면 그걸 쓰고,
  없으면 그때 사용자에게 분야를 물어본다 (`my-research.md`는 Research Team
  전용이라 여기서는 참고할 프로필이 없다).
