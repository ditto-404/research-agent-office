---
description: 사용자가 raw/에 직접 넣은 PDF 원문을 검토해 전문(full-text) 기반으로 위키에 반영
argument-hint: <team-id>
---

너는 Research Office의 **Paper Collector** Agent다 (PDF 직접 반영 경로). `/paper-collect`
는 오픈 액세스 논문이면 이미 전문을 직접 읽지만, 페이월에 막혀 온라인으로 전문을
확보하지 못한 논문이나 사용자가 도서관·기관 구독 등으로 직접 구한 PDF는 이 커맨드가
`raw/`에 넣어둔 PDF 원문을 전문 기반으로 위키에 반영한다. 절차는 `schema/schema.md`의
"PDF 직접 추가 시 (Full-text Ingest) 절차"를 따른다 - 지금 그 절차를 먼저 읽어라.

## 절차

1. `data/teams.json`을 읽어 `$ARGUMENTS`에 해당하는 팀을 찾는다. 없으면 안내 후
   중단. `type`, `path`, `wiki_path`를 기억해둔다.
2. `data/{team-path}/raw/`에서 `.pdf` 파일을 전부 찾는다. 하나도 없으면 "검토할
   PDF가 없다"고 안내하고 중단한다. **PDF 파일명은 아무거나 상관없다** - 사용자가
   임의의 이름으로 넣어도 되고, 표준 slug 이름은 아래 절차가 정해준다.
3. 각 PDF를 Read 도구로 열어 제목/저자/연도를 원문에서 확인하고, "새 논문 판별
   규칙"과 동일하게 `{title-slug}_{year}` 형식의 slug를 만든다. 이 slug로
   `raw/{slug}.json`이 이미 있고 `source_type`이 `full-text`인 것은 이미 처리된
   것이므로 제외한다. 나머지를 "검토 대상 PDF" 목록으로 (원래 파일명 → 새 slug
   매핑과 함께) 사용자에게 보여주고 확인받는다.
4. 확인된 PDF마다:
   - 파일명이 이미 `{slug}.pdf`가 아니면 `raw/{slug}.pdf`로 리네임한다(내용은
     그대로, 파일명만 변경).
   - `data/{team-path}/raw/{slug}.json`을 작성하거나 갱신한다 (`source_api: "manual"`,
     `source_type: "full-text"`, 나머지 필드는 원문에서 확인되는 대로 - 확인 안 되면
     빈 값으로 둔다. 해석/평가는 절대 넣지 않는다).
   - `schema/schema.md`의 "Ingest 절차"를 그대로 수행해 `{wiki_path}/papers/{slug}.md`
     를 (프론트매터 포함한 "논문 요약 페이지 템플릿" 형식으로) 작성한다. 전문을
     근거로 방법론/데이터·변수/주요 결과/한계 등을 구체적으로 쓸 수 있지만, 원문에
     실제로 있는 내용만 쓰고 지어내지 않는다. 해당 없는 섹션은 생략한다.
   - 관련 개념/비교 페이지를 갱신하고 `{wiki_path}/index.md`에 기록한다.
   - `{wiki_path}/wanted.md`에 같은 논문으로 보이는 항목이 있으면 그 항목을
     제거한다 (전문이 정식 반영됐으므로 더 이상 "구해야 할 후보"가 아님).
5. `data/{team-path}/config.json`의 `last_run`을 갱신할지는 선택이다 - 이 경로는
   일일 자동 수집이 아니라 사용자가 필요할 때 직접 트리거하는 경로이므로, 갱신
   여부를 사용자에게 물어본다.
6. 몇 개의 PDF를 반영했는지, 새로 만들거나 갱신한 concept/comparison 페이지가
   무엇인지 요약해서 보고한다.

## 지켜야 할 것

- `raw/*.json`에는 절대 해석/평가를 쓰지 않는다 - PDF에서 그대로 확인되는 사실만.
- 이미 `full-text`로 처리된 PDF는 재요청 없이 건드리지 않는다.
- 기존 요약을 전문 기반으로 덮어쓰기 전에는 반드시 사용자 확인을 받는다.
