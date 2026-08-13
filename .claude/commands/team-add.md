---
description: 새 Research Team 생성 (이름/주제/키워드/검색조건 확인 후 스캐폴딩 + 등록)
---

너는 Research Office의 팀 관리 Agent다. 새로운 **Research Team**을 하나 만든다. 팀 데이터는
두 곳에 나뉘어 생긴다 - `data/teams/{slug}/`(config.json, raw/, git 저장소)와
`<WIKI_VAULT_ROOT>/{slug}/`(위키 콘텐츠, Obsidian Vault, git 아님). 자세한 근거는
`schema/schema.md`의 "저장 위치" 섹션 참고.

## 절차

1. 사용자에게 다음을 확인한다 (한 번에 물어봐도 되고, 이미 메시지에 포함돼 있으면
   다시 묻지 않는다):
   - 팀 이름 (예: "도시 활력 연구팀")
   - 연구 주제 (한두 문장)
   - 키워드 목록 (검색에 쓸 핵심어들)
   - 검색 조건/소스 (기본값: arxiv, semantic-scholar, openalex - 특별한 요청 없으면
     기본값 그대로 진행해도 됨)
   - 하루 수집 목표 편수 (기본 3~4편, 특별한 요청 없으면 기본값 사용)
2. 팀 이름을 소문자/하이픈 slug로 변환한다 (예: "도시 활력 연구팀" → 사용자에게 영문
   slug를 제안하고 확인받는다, 예: `example-team`). `data/teams.json`, `data/teams/`,
   `<WIKI_VAULT_ROOT>/`에 이미 같은 slug가 있으면 다른 slug를 제안한다.
3. `data/teams/_template/`을 `data/teams/{slug}/`로 복사한다 (`config.json`,
   `my-research.md`(빈 템플릿), `raw/.gitkeep`이 있음 - `.gitkeep`은 그대로 유지,
   빈 디렉토리 보존용).
4. `data/teams/{slug}/config.json`을 실제 값으로 채운다 (`id`, `name`, `topic`,
   `keywords`, `search_conditions`, `papers_per_day`, `created`: 오늘 날짜,
   `last_run`: null).
5. `<WIKI_VAULT_ROOT>/{slug}/`를 새로 만들고 그 아래에 `papers/`, `concepts/`,
   `comparisons/`, `synthesis/history/`, `advisor/history/` 폴더를 만든다 (git
   저장소가 아니므로 `.gitkeep` 불필요, 그냥 빈 폴더로 둔다). 같은 위치에
   `index.md`를 아래 형식으로 생성한다:
   ```markdown
   # {팀명} 위키 지도

   이 위키의 전체 목차. `/paper-collect {slug}`, `/synthesize {slug}`,
   `/advise {slug}` 실행 시 Agent가 갱신한다.

   ## 논문 (Papers)
   (아직 없음)

   ## 개념 (Concepts)
   (아직 없음)

   ## 비교 (Comparisons)
   (아직 없음)

   ## 관심 논문 (Wanted)
   전문 접근 불가로 raw/papers에는 반영되지 않은 후보 목록: [[wanted.md]]

   ## Synthesis
   (아직 없음)

   ## Advisor
   (아직 없음)

   ## 최근 갱신
   (아직 없음)
   ```
   같은 위치에 `schema/templates/wanted.md`를 근거로 `wanted.md`도 생성한다
   (`{팀명}` 자리를 실제 팀 이름으로 치환).
6. `data/teams.json`에 다음 항목을 추가한다:
   ```json
   {"id": "{slug}", "type": "research-team", "name": "{팀명}", "created": "{오늘}",
    "path": "teams/{slug}",
    "wiki_path": "<WIKI_VAULT_ROOT>/{slug}",
    "pipeline": ["collector", "synthesizer", "advisor"]}
   ```
7. 방금 만든 `data/teams/{slug}/my-research.md`는 빈 템플릿 상태다. 지금 바로
   `/my-research-setup {slug}`로 채울지 사용자에게 묻는다 - 이 문서가 채워져야
   Research Advisor뿐 아니라 Paper Collector도 이 팀에 필요한 논문을 더 정확히
   찾는다.
8. 생성 완료를 보고하고, 다음 단계로 `/paper-collect {slug}`를 실행하면 논문 수집이
   시작된다고 안내한다.

## 지켜야 할 것

- `_template/`은 원본 그대로 남겨둔다 (복사만 하고 수정하지 않음).
- slug 충돌은 `data/teams/`뿐 아니라 `<WIKI_VAULT_ROOT>/`에서도 반드시 사전에 확인한다.
