---
description: 모든 팀 상태를 스캔해 data/status.json을 갱신 (픽셀 Office UI가 읽을 데이터)
---

너는 Research Office의 상태 집계 Agent다. `data/teams.json`에 등록된 모든 팀을 스캔해
`data/status.json`을 최신 상태로 다시 쓴다. 이 파일은 아직 UI가 없는 지금 단계에서는
"파이프라인이 실제로 얼마나 돌았는지"를 확인하는 용도로도 쓰인다.

## 절차

1. `data/teams.json`을 읽어 팀 목록을 얻는다 (각 항목의 `path`, `wiki_path` 포함).
2. 각 팀에 대해:
   - `{wiki_path}/papers/*.md`, `{wiki_path}/concepts/*.md`, `{wiki_path}/comparisons/*.md`
     개수를 센다.
   - `data/{path}/config.json`의 `last_run` 값을 가져온다.
   - `{wiki_path}/synthesis/latest.md`가 있으면 그 파일의 최종 수정 관련 정보(파일 안에
     적힌 작성일)를 가져온다. 없으면 `null`.
   - `pipeline`에 `"advisor"`가 포함되면 `{wiki_path}/advisor/latest.md`도 동일하게
     확인. 없으면 이 필드 자체를 생략.
3. 아래 형식으로 `data/status.json`을 전체 재작성한다:
   ```json
   {
     "generated_at": "{ISO 8601 timestamp}",
     "teams": [
       {
         "id": "...",
         "type": "journal-watch | research-team",
         "name": "...",
         "paper_count": 0,
         "concept_count": 0,
         "comparison_count": 0,
         "last_run": null,
         "synthesis_updated_at": null,
         "advisor_updated_at": null
       }
     ]
   }
   ```
   (`advisor_updated_at`은 `type: journal-watch` 팀 항목에는 아예 넣지 않는다.)
4. 팀별 요약(논문 수, 마지막 실행일)을 표로 사용자에게 보고한다.
