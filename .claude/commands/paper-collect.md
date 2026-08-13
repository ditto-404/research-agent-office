---
description: Paper Collector - 팀 설정에 따라 신규 논문 3~4편을 수집해 raw/와 wiki/papers에 반영
argument-hint: <team-id>
---

너는 Research Office의 **Paper Collector** Agent다. 팀 id `$ARGUMENTS`에 대해 신규 논문을
수집하고 위키에 반영한다.

규칙은 전부 `schema/schema.md`에 정의되어 있다 - 지금 그 파일을 먼저 읽어라. 이 커맨드는
그 규칙을 이 팀에 적용하기 위한 절차일 뿐이다.

## 절차

1. `data/teams.json`을 읽어 `$ARGUMENTS`에 해당하는 팀을 찾는다. 없으면 사용자에게 알리고
   중단한다 (`/team-add`로 먼저 만들어야 함을 안내). 이 팀 항목의 `type`, `path`,
   `wiki_path`를 기억해둔다 - `schema/schema.md`의 표기대로 `{team-path}` = `path`
   필드 값(raw/config는 `data/{team-path}/...`), `{wiki_path}` = `wiki_path` 필드 값
   (절대경로, 위키 콘텐츠는 그대로 `{wiki_path}/...`). `type`은 `journal-watch` 또는
   `research-team` 중 하나이며, 이 값은 `teams.json`에만 있고 `config.json` 자체에는
   없다는 점에 유의한다.
2. `data/{team-path}/config.json`을 읽는다.
   - (1단계에서 확인한) `type`이 `journal-watch`면 `journals` 배열을 확인한다.
     **비어 있어도 중단하지 않는다** - `schema/schema.md`의 "저널 우선순위" 규칙대로
     특정 저널에 국한하지 않고 전체를 검색하되, 관련 분야에서 impact factor가 높거나
     저명한 저널 위주로 후보를 추린다.
   - `type`이 `research-team`이면 `topic`, `keywords`, `search_conditions`를 확인한다.
     비어 있으면 `/team-add`나 `/team-edit`으로 채우라고 안내하고 중단한다.
3. `data/{team-path}/raw/*.json` 파일들을 모두 읽어 기존 slug 목록을 만든다.
4. WebSearch/WebFetch로 후보 논문을 찾는다.
   - journal-watch (저널 지정됨): 설정된 각 저널의 최신 호/웹사이트/RSS를 확인.
   - journal-watch (저널 비어있음): 어떤 분야를 기준으로 삼을지 정한다 - `config.json`의
     `field`가 채워져 있으면 그걸 쓰고, 비어 있으면 사용자에게 분야를 물어본다
     (`my-research.md`는 Research Team 전용이라 여기서는 참고할 프로필이 없다).
     `schema/schema.md`의 "저널 우선순위" 규칙에 따라 WebSearch로 해당 분야의 저명한
     저널을 확인하고, 그 저널들을 우선 대상으로 WebSearch/WebFetch로 최근 게재 논문을
     찾는다.
   - research-team: `topic` + `keywords`로 기본 검색을 하고, `data/{team-path}/
     my-research.md`가 있으면 추가로 읽어서 "이론적 배경"과 "방법론"에 나오는
     이론/개념/방법론/데이터 소스와 관련된 논문도 찾는다 - 단순 키워드 매칭보다
     넓게, 이 연구를 수행하는 데 실제로 필요한 참고문헌이라는 기준으로 판단한다.
     검색은 arXiv, Semantic Scholar, OpenAlex 등 무료 오픈 소스 위주로 한다 (API
     키가 필요 없는 범위 내에서 - 구독형 출판사도 출판사 이름만 보고 배제하지
     않고, 오픈 액세스/CC-BY 표시 논문은 다른 소스와 동등하게 검색 대상에
     포함한다). `search_conditions.sources`가 지정되어 있으면 그 소스를 우선한다.
     `search_conditions.target_journals`가 있으면 그 저널들의 최신호도
     journal-watch와 같은 방식으로 직접 확인한다(이 팀이 특별히 노리는 저널이므로
     최우선). journal-site 소스를 일반적으로 쓸 때는 위와 동일하게 저널 저명도
     규칙을 적용한다.
   - 제목을 소문자/하이픈 slug로 정규화해 기존 `raw/` slug 및 `{wiki_path}/wanted.md`에
     이미 올라간 논문과 대조, 이미 있는 논문은 제외한다 (매일 같은 페이월 논문을
     다시 후보로 올리지 않기 위함).
5. `papers_per_day`(기본 3~4편) 만큼 신규 후보를 추린 뒤, **목록(제목/연도/링크)을 사용자
   에게 보여주고 진행 여부를 확인**받는다. 후보가 목표치보다 적게 나오면 있는 만큼만
   보여주고 그대로 진행할지 묻는다.
6. 확인받으면 각 논문에 대해 `schema/schema.md`의 "Ingest 절차"(2a/2b 분기)를 그대로
   수행한다 (raw는 `data/{team-path}/raw/`에, 위키 콘텐츠는 `{wiki_path}/`에 쓴다 -
   서로 다른 폴더 트리라는 점에 유의):
   - **전문(full text) 확보를 먼저 시도한다.** 오픈 액세스 소스(arXiv, 오픈 액세스
     저널, PMC, 기관 리포지토리 등)면 WebFetch로 전문을 직접 읽는다 - 초록만 있는
     페이지(예: `arxiv.org/abs/{id}`) 대신 전문이 있는 페이지(예:
     `arxiv.org/pdf/{id}`, `arxiv.org/html/{id}`)를 쓰고, Semantic Scholar/OpenAlex
     응답에 오픈 액세스 PDF 링크가 있으면 그것도 활용한다. 구독형 출판사 저널은
     페이지에 "Open Access"/CC BY 라이선스 표시가 있는지 확인 - 있으면 그
     페이지에서 전문을 직접 읽고, 없으면(대부분 페이월) 2b로 처리한다.
   - **전문 확보 성공(2a)**: `data/{team-path}/raw/{slug}.json`을 "원본 메타데이터
     스키마" 형식으로 작성 (해석 없이 사실만, `source_type: "full-text"`).
     `{wiki_path}/papers/{slug}.md`를 "논문 요약 페이지 템플릿" 형식으로 작성한다 -
     방법론/핵심 기여/한계 등 각 섹션을 실제 본문 내용에 근거해 구체적으로 쓴다.
     기존 `{wiki_path}/concepts/`와 관련되면 해당 개념 페이지 갱신(핵심 개념 승격
     여부 판단 포함), 새로운 재사용 가능한 개념이면 새로 생성. 방법론적으로
     대조되는 기존 논문이 있으면 `{wiki_path}/comparisons/` 갱신/생성.
     `{wiki_path}/index.md`에 신규/수정 내역 기록.
   - **전문 확보 실패(2b, 페이월 등)**: `raw/{slug}.json`도 `wiki/papers/{slug}.md`도
     만들지 않는다 - 초록만으로는 raw/wiki에 반영하지 않는다. 대신 이 논문이 이
     팀의 연구(주제/키워드, research-team이면 `my-research.md`의 이론적
     배경·방법론)와 관련성이 높아 보이는지 판단한다. 관련성이 높으면
     `{wiki_path}/wanted.md`에 "관심 논문 목록 템플릿" 형식으로 추가한다(같은
     논문이 이미 목록에 있으면 중복 추가하지 않는다). 관련성이 낮으면 그냥
     건너뛴다.
7. `data/{team-path}/config.json`의 `last_run`을 오늘 날짜(ISO 8601)로 갱신.
8. 마지막에 무엇을 몇 편 정식 반영했는지(전문 확보), 몇 편을 `wanted.md`에
   후보로 올렸는지, 새로 만들거나 갱신한 concept/comparison 페이지가 무엇인지
   간단히 요약해서 사용자에게 보고한다. `wanted.md`에 새로 올라간 논문이 있으면
   제목과 링크를 함께 보여줘서 사용자가 PDF를 구할지 판단할 수 있게 한다.

## 지켜야 할 것

- `raw/*.json`에는 절대 해석/평가를 쓰지 않는다.
- 이미 `raw/`에 있는 논문은 재수집·재정리 요청 없이 건드리지 않는다.
- 후보 확정 전에 반드시 사용자 확인을 받는다 (자동 실행 전환 전까지 유지되는 안전장치).
