# Research Office 위키 운영 규칙 (Schema)

이 문서는 Agent(Claude)가 팀별 위키를 어떻게 관리해야 하는지 정의한다. 모든 팀
(고정 Journal Watch 타입 팀 1개 + `data/teams/<slug>/`의 Research Team들)이 이
규칙을 공유한다. 원본은
Karpathy "LLM Wiki" 패턴(`research-wiki/schema.md`)이며, 여기서는 (1) 자동 논문 수집에
맞게 "새 논문 판별"을 PDF가 아닌 메타데이터 기준으로 바꾸고, (2) `synthesis/`와
`advisor/` 두 산출물을 추가했고, (3) 원본 메타데이터(raw)와 위키 콘텐츠(wiki)의 저장
위치를 물리적으로 분리했다.

## 저장 위치 (raw는 data/, wiki는 Obsidian Vault)

이 문서와 커맨드들에서 쓰는 표기: `{team-path}` = `data/teams.json`의 해당 팀 항목의
`path` 필드 값(예: `trend-watch`, `teams/urban-vitality` - `data/` 기준 상대경로,
그 자체엔 `data/`가 안 붙어있음). `{wiki_path}` = 같은 항목의 `wiki_path` 필드 값.
그래서 raw/config는 항상 `data/{team-path}/...`로, 위키 콘텐츠는 항상
`{wiki_path}/...`로 (앞에 `data/`를 붙이지 않고) 표기한다.

**`wiki_path`는 이 프로젝트 루트를 기준으로 한 상대경로**(예: `../../1. Wiki/<team-id>`
- 프로젝트 루트에서 두 단계 위인 OneDrive 루트로 올라간 뒤 `1. Wiki/<team-id>`로
들어감)로 저장한다. 이 프로젝트와 `1. Wiki` 폴더가 둘 다 같은 OneDrive 루트 아래
있는 형제 폴더라서, 이 상대 위치는 Mac/Windows 등 기기가 바뀌어도 항상 동일하다
(반면 절대경로는 기기마다 `/Users/yein/...`, `D:\Users\User\...`처럼 달라진다).
Read/Write 도구는 절대경로만 받으므로, `{wiki_path}`를 실제로 쓸 때는 **먼저 이
프로젝트 루트의 현재 절대경로에 이 상대경로를 이어붙여 절대경로로 변환한 뒤** 그
절대경로로 Read/Write한다. (예전 데이터에 절대경로가 그대로 남아있을 수도 있는데,
그 경우는 이미 완전한 경로이므로 변환 없이 그대로 쓴다.)

- **`raw/`, `config.json`** - 고정 Journal Watch 타입 팀(`data/trend-watch/`)과
  각 Research Team(`data/teams/<slug>/`) 안에 있다. `data/`는 독립 git 저장소(향후
  private repo)이며, 소스 오브 트루스와 팀 레지스트리를 git으로 버전관리하기 위한
  위치다.
- **`wiki/`의 실제 내용물(papers/concepts/comparisons/synthesis/advisor/index.md)**은
  `data/` 안이 아니라 `1. Wiki/<team-id>/`(OneDrive 루트 바로 아래, 프로젝트 루트
  기준 `../../1. Wiki/<team-id>/`)에 생성된다. 이 경로는 사용자가 Obsidian Vault로
  여는 OneDrive 폴더이며, git으로 버전관리하지 않는다 (원본 `research-wiki`와
  동일한 방식 - OneDrive 동기화 + Obsidian 자체 설정에 맡긴다).
  - 예: 고정 Journal Watch 타입 팀(id `trend-watch`) → `1. Wiki/trend-watch/papers/`,
    `.../concepts/`, ...
  - 예: Research Team `urban-vitality` → `1. Wiki/urban-vitality/papers/`, ...
  - 팀별 폴더 하위에는 더 이상 `wiki/`라는 중첩 폴더를 두지 않는다 - `1. Wiki/<team-id>/`
    자체가 그 팀의 위키 루트다 (`papers/`, `concepts/`, `comparisons/`, `synthesis/`,
    `advisor/`, `index.md`가 바로 그 밑에 온다).
  - 각 팀의 위키 위치는 `data/teams.json`의 해당 팀 항목 `wiki_path` 필드에
    기록되어 있다 - Agent는 이 필드를 읽고 위 규칙대로 절대경로로 변환해 어디에
    쓸지 결정한다.
- 이 분리 때문에 논문 요약 페이지에서 원본 메타데이터를 가리킬 때 상대경로
  `raw/{slug}.json`을 쓸 수 없다 (더 이상 같은 폴더 트리에 있지 않음) - 아래 "논문 요약
  페이지 템플릿"의 형식을 따른다.

## 폴더 역할

- `raw/{slug}.json` (위치: `data/`) - 논문의 원본 메타데이터. API/웹에서 그대로
  가져온 사실 정보만 담는다 (title, authors, year, journal, doi, url, abstract,
  source_api, collected_at, collected_by_team, source_type). AI의 해석·요약·평가는
  절대 넣지 않는다. 오픈 액세스 논문(arXiv, 오픈 액세스 저널, PMC, 기관 리포지토리,
  ScienceDirect(Elsevier)의 오픈 액세스/CC-BY 표시 논문 등)은 전문(full text)을
  직접 읽어 `source_type: "full-text"`로 기록한다. **전문을
  확보하지 못한(페이월) 논문은 raw/wiki에 아예 반영하지 않는다** -
  `source_type: "abstract-only"`인 항목을 만들지 않는다. 대신 아래 `wanted.md`를
  참고. 사용자가 PDF를 직접 구해서 `raw/`에 넣으면(파일명은 아무거나 상관없다,
  `/paper-review-pdf`가 원문에서 제목/연도를 읽어 표준 slug로 리네임한다)
  `/paper-review-pdf`로 전문 기반 반영한다.
- `wanted.md` (위치: `1. Wiki/<team-id>/`) - Paper Collector가 검색은 했지만 전문을
  확보하지 못한 논문 중, 이 팀의 연구와 관련성이 높아 보이는 것만 골라 올려두는
  목록. raw나 wiki papers로 반영되지는 않은, "PDF를 구하면 좋을 후보" 목록이다.
  사용자가 PDF를 구해서 `/paper-review-pdf`로 반영하면 이 목록에서 빠진다.
- `papers/` (위치: `1. Wiki/<team-id>/`) - 논문 1편당 요약 페이지 1개. **AI가 raw를
  근거로 작성한 해석.** 전문을 확보한(raw에 반영된) 논문만 여기 들어온다.
- `concepts/` - 여러 논문에 공통으로 등장하는 개념 페이지.
- `comparisons/` - 두 개 이상 방법론/논문을 비교하는 페이지.
- `synthesis/latest.md` - Research Synthesizer 산출물(최근 동향/핵심개념/방법론/
  emerging topics). 갱신될 때마다 이전 버전은 `synthesis/history/{date}.md`로 이동.
- `advisor/latest.md` - Research Advisor 산출물. **Research Team 전용** (Journal
  Watch에는 없음). 갱신 시 이전 버전은 `advisor/history/{date}.md`로 이동.
- `index.md` - 전체 페이지 목록과 최근 갱신 이력.

## 저널 우선순위 (Journal Prioritization)

Journal Watch의 `journals` 목록은 **선택 사항**이다. 비어 있어도 팀은 정상 동작한다.

- `journals`가 비어 있으면 특정 저널에 국한하지 않고 arXiv/Semantic Scholar/OpenAlex와
  일반 웹 검색으로 전체를 검색한다. 다만 후보를 추릴 때는 관련 분야에서 impact factor가
  높거나 저명한 저널에 실린 논문을 우선한다. 출판사도 특정 소스로 좁히지 않는다 -
  Elsevier(ScienceDirect) 저널도 arXiv/PMC와 동등하게 검색 대상이며, 저명도가
  충분하면 후보에서 배제하지 않는다(전문 확보 가능 여부는 아래 Ingest 절차에서
  별도로 판단).
- 어떤 분야의 "저명한 저널"인지 판단할 때는, 로컬에 아래 참고 자료가 있으면 먼저
  확인한다 (없거나 해당 분야가 안 실려 있으면 WebSearch로 최신 저널 순위/IF를 확인):
  이 프로젝트 루트 경로에서 두 단계 위(OneDrive 루트) 폴더로 올라간 뒤
  `3. Claude Code Redistribution/2. academic-research-skills-main/academic-research-skills-main/academic-paper-reviewer/references/top_journals_by_field.md`
  를 이어붙인 절대경로 (education, CS/AI, business, social sciences, engineering,
  medicine, information systems, interdisciplinary, 아시아/지역 저널 섹션 포함.
  IF 수치는 시점에 따라 달라질 수 있으므로 참고용이며, 이 파일에 없는 분야는
  WebSearch로 보완한다. 이 경로는 프로젝트가 어느 기기에 있든 OneDrive 동기화
  구조상 항상 이 상대 위치에 있다 - Mac/Windows 절대경로를 하드코딩하지 않는다.)
- `journals`가 채워져 있으면 그 목록을 그대로 우선 사용하고, 위 저명도 판단은 목록에
  없는 추가 후보를 찾을 때만 보조적으로 쓴다.
- Research Team(`search_conditions`)에도 동일하게 적용한다 - 검색 소스가 arXiv 등
  프리프린트 서버뿐이면 저널 저명도 개념이 없을 수 있으니, journal-site 소스를
  검색할 때만 이 규칙을 적용한다.

## "새 논문" 판별 규칙 (자동 수집 기준)

`/paper-collect <team-id>`가 실행되면:

1. 팀의 `raw/` 안에 있는 모든 `{slug}.json`의 slug 목록과, `{wiki_path}/wanted.md`
   에 이미 올라와 있는 논문 목록을 확인한다 (`wanted.md`에 있는 논문은 어제 이미
   전문 접근 불가로 판별된 것이므로 매일 다시 찾아서 또 보여주지 않는다).
2. 팀 설정(저널 목록 또는 topic/keywords)에 따라 WebSearch/WebFetch로 후보 논문을 찾는다
   (소스: arXiv, Semantic Scholar, OpenAlex, ScienceDirect(Elsevier) 포함 저널
   웹사이트/RSS 등, 무료·API 키 불필요 범위 내 - 출판사 이름만 보고 후보에서
   제외하지 않는다). Research Team이면 `topic`/`keywords`로 만든 기본 검색에 더해
   `{team-path}/my-research.md`(있으면)의 "이론적 배경"과 "방법론"에 나오는
   이론/개념/방법론/데이터 소스와 관련된 논문도 함께 찾는다 - 단순 키워드
   매칭보다 넓게, 이 연구를 수행하는 데 실제로 필요한 참고문헌을 찾는다는 기준으로
   판단한다. **최근 트렌드 논문에만 치우치지 않는다** - `my-research.md`의
   "이론적 배경 / 관점"에 이름이 언급된 고전·저명 이론(예: Jacobs, Third Place
   이론처럼 특정 학자·이론명이 명시된 경우)이 있으면, 그 이론의 원 문헌이나 그
   이론을 이 팀의 주제에 적용한 후속 연구도 검색 대상에 포함한다(원 문헌이
   유료·절판이라 전문을 못 구해도 무방 - 이 경우 2b 절차대로 처리).
   **특정 갈래(방법론/데이터 소스)로만 검색이 쏠리지 않게 한다** - `my-research.md`의
   "이론적 배경 / 관점"에 여러 갈래(strand)가 나열돼 있으면(예: 고전 이론/특정
   이론명/측정 전통/정책 관점처럼 성격이 다른 여러 항목), 매 회차 최소 한 번씩은
   각 갈래를 명시적으로 검색해본다. 실제로 겪은 문제: `config.json`의 `keywords`가
   방법론·데이터 용어(MGWR, GeoDetector 등) 위주로 채워져 있으면 `topic`/`keywords`
   기반 기본 검색 결과도 그쪽으로 쏠리고, 검색을 실행하는 쪽도 이미 결과가 잘
   나오는 갈래를 계속 반복하기 쉽다 - 그 결과 여러 회차에 걸쳐 특정 이론 갈래
   (예: 고전 이론)가 한 번도 검색되지 않는 일이 생겼다. `keywords`를 채우거나
   고칠 때(`/team-add`, `/team-edit`, 또는 이 단계에서 스스로 판단할 때)도 이론적
   배경에 나열된 갈래 수만큼 골고루 대표 키워드를 넣는다(아래 "팀 설정 스키마"
   참고).
3. 후보의 제목을 slug로 정규화(소문자, 하이픈)하고 **끝에 발행년도를 밑줄로
   붙인다**(`{title-slug}_{year}`, 예: `mapping-urban-vitality-tallinn_2026`) -
   같은 논문의 개정판·프리프린트가 연도만 다르게 재게재되는 경우를 구분하고,
   `papers/` 폴더를 파일명만으로도 연도순으로 훑어볼 수 있게 하기 위함. 이 slug로
   기존 `raw/` slug 및 `wanted.md` 목록과 대조 - 이미 있으면 제외.
4. 하루 목표 편수(팀 설정의 `papers_per_day`, 기본 3~4편)만큼 신규 후보를 추리고,
   **목록을 사용자에게 먼저 보여주고 확인**받는다 (기존 원본 스키마의 확인 절차 유지 -
   자동 실행으로 전환되기 전까지는 사람이 최종 확인).
5. 확인 후 아래 "Ingest 절차"를 각 논문에 대해 순서대로 수행한다 (전문을 확보하지
   못하면 raw/wiki에는 안 남고, 관련성이 높을 때만 `wanted.md`에 남는다).
6. 이미 `raw/`에 있는 논문은 별도 요청(재정리 등) 없이 건드리지 않는다.

## Ingest 절차 (신규 논문 추가)

1. **전문(full text) 확보를 먼저 시도한다.** 오픈 액세스 소스면 WebFetch로 전문
   페이지나 PDF를 직접 읽는다 - 예: arXiv는 초록만 있는 `arxiv.org/abs/{id}` 대신
   전문이 있는 `arxiv.org/pdf/{id}`나 `arxiv.org/html/{id}`를 쓴다. Semantic
   Scholar/OpenAlex 응답에 오픈 액세스 PDF 링크(`openAccessPdf` 등)가 있으면
   그것도 활용한다. 저널 웹사이트가 Open Access로 표시되어 있으면 그 페이지에서
   전문을 직접 읽는다. **단, `sciencedirect.com`(Elsevier)은 예외다** -
   `linkinghub.elsevier.com` 리다이렉트를 포함해 WebFetch로 시도하면 CC-BY
   오픈 액세스 논문이라도 HTTP 403이나 빈 "Redirecting" shell만 돌아온다(실제로
   겪은 문제, 2026-08-14 - OA/페이월 여부와 무관하게 사이트 자체가 자동화된
   접근을 차단한다). 그래서 ScienceDirect 논문은 **오픈 액세스/CC-BY 표시가
   있어도 WebFetch로 전문을 읽으려 시도하지 않고 곧장 2b로 간다** - `wanted.md`
   등재 사유에 "CC-BY 오픈 액세스이지만 ScienceDirect가 자동 접근을 차단해
   전문 미확보(페이월 아님)"라고 구분해서 적어, 사용자가 이건 직접 열람만 하면
   바로 반영 가능하다는 걸 알 수 있게 한다.
2. 전문을 확보했으면 2a로, 페이월(또는 ScienceDirect처럼 접근 자체가 막힌 경우)로
   확보하지 못했으면 2b로 간다.

   **2a. 전문 확보 성공 - 정식 반영**
   1. `data/{team-path}/raw/{slug}.json` 생성 - 아래 "원본 메타데이터 스키마" 형식,
      `source_type: "full-text"`. 사실만, 해석 없음.
   2. `{wiki_path}/papers/{slug}.md` 생성 - 아래 "논문 요약 페이지 템플릿" 형식
      (`wiki_path`는 `data/teams.json`의 해당 팀 항목에서 가져온 절대경로).
      방법론/핵심 기여/한계를 실제 본문 내용에 근거해 구체적으로 쓴다.
   3. 논문이 다루는 개념 중 `{wiki_path}/concepts/`에 이미 있는 페이지가 있으면
      해당 페이지의 "관련 논문"(핵심 개념이면 "논문별 등장 방식") 섹션에 이번
      논문 추가. 이 시점에 핵심 개념 조건을 새로 충족하면 기본 템플릿 → 확장
      템플릿으로 승격.
   4. 새 개념이 등장하고 재사용 가능성이 있으면 `{wiki_path}/concepts/{slug}.md`
      생성.
   5. 기존 논문과 방법론적으로 대조되는 지점이 있으면 `{wiki_path}/comparisons/`
      에 생성/갱신.
   6. `{wiki_path}/index.md`에 신규/수정 페이지 기록.

   **2b. 전문 확보 실패 - raw/wiki papers에는 반영하지 않음**
   1. `raw/{slug}.json`도 `wiki/papers/{slug}.md`도 만들지 않는다 - 초록만으로
      항목을 만들지 않는다.
   2. 이 논문이 팀의 `topic`/`keywords`나 `my-research.md`(있으면)의 이론적
      배경/방법론과 얼마나 관련 있는지 판단한다. **관련성이 높다고 판단되면**
      `{wiki_path}/wanted.md`에 항목을 추가한다 (아래 "관심 논문 목록 템플릿"
      형식, 이미 같은 논문이 올라와 있으면 중복 추가하지 않는다). 관련성이
      낮으면 그냥 건너뛴다 (아무것도 기록하지 않음).
3. `data/{team-path}/config.json`의 `last_run`을 갱신.

## 관심 논문 목록 템플릿 (`{wiki_path}/wanted.md`)

```markdown
# {팀명} 관심 논문 목록 (전문 접근 불가)

Paper Collector가 검색은 했지만 페이월 등으로 전문을 확보하지 못한 논문 중, 이
연구와 관련성이 높아 보이는 것만 골라둔 목록이다. raw나 위키 papers로 반영된
상태가 아니다. PDF를 구해서 `raw/`에 넣고(파일명은 아무거나 상관없다)
`/paper-review-pdf <team-id>`를 실행하면 정식으로 반영되고 이 목록에서는 빠진다.

## {논문 제목} ({연도})
- 저자:
- 저널/출처:
- 링크/DOI:
- 왜 필요해 보이는지: (이 팀의 연구와 관련성 판단, 1~2문장)
- 발견일: {date}
```

## PDF 직접 추가 시 (Full-text Ingest) 절차

`/paper-collect`는 오픈 액세스 논문이면 이미 전문을 직접 읽지만, 페이월에 막혀
온라인으로 전문을 확보하지 못해 raw/wiki에 반영되지 않은 논문(관련성이 높으면
`wanted.md`에 올라있을 것이다)이나, 사용자가 도서관·기관 구독 등으로 직접 구한
PDF는 `/paper-review-pdf <team-id>`로 반영한다. **사용자는 PDF를 아무 파일명으로나
`raw/`에 넣어도 된다** - 파일명을 표준 형식에 맞추는 건 이 절차가 대신한다.

1. `data/{team-path}/raw/`에서 확장자가 `.pdf`인 파일을 전부 확인한다.
2. 각 PDF를 Read 도구로 열어 제목/저자/연도/DOI 등을 원문에서 직접 확인한다.
   "새 논문 판별 규칙"과 동일한 방식으로 제목을 slug로 정규화(소문자, 하이픈)하고
   끝에 발행년도를 밑줄로 붙여 `{title-slug}_{year}` 형식의 slug를 만든다.
3. 이 slug로 `raw/{slug}.json`이 이미 있고 `source_type`이 `full-text`인지
   확인한다. 이미 있으면 처리된 것이므로 건너뛴다(그 경우 원본 PDF도 이미
   `raw/{slug}.pdf`로 이름이 맞춰져 있을 것이다). 없으면 "검토 대상 PDF"로
   판별한다.
4. 새로 판별된 PDF 목록을 (원래 파일명 → 새로 정할 slug 매핑과 함께) 사용자에게
   보여주고 확인받는다.
5. 확인된 PDF마다:
   - 파일명이 이미 `{slug}.pdf`가 아니면 `raw/{slug}.pdf`로 이름을 바꾼다(rename,
     내용은 그대로 두고 파일명만 변경).
   - `raw/{slug}.json`을 작성한다 - `source_api: "manual"`, `source_type:
     "full-text"`, `collected_at`은 오늘 날짜. 제목/저자/연도/DOI 등은 원문에서
     직접 추출한 값을 쓰고, 확인 안 되면 빈 값으로 둔다. 이 파일에도 해석은
     넣지 않는다 (사실만).
6. "Ingest 절차"의 2a(전문 확보 성공 경로)를 그대로 수행해 `wiki/papers/{slug}.md`
   등을 작성한다 - 실제로 원문에 근거가 있는 만큼만 서술하고, 원문에 없는 내용을
   지어내지 않는다.
7. `{wiki_path}/wanted.md`에 같은 논문으로 보이는 항목이 있으면 그 항목을
   제거한다 (정식으로 반영됐으므로).

## 원본 메타데이터 스키마 (`raw/{slug}.json`)

`slug`는 위 "새 논문 판별 규칙"에서 정한 `{title-slug}_{year}` 형식을 그대로 쓴다 -
`raw/{slug}.json`과 `{wiki_path}/papers/{slug}.md`가 항상 같은 slug(파일명)를
공유한다.

```json
{
  "slug": "",
  "title": "",
  "authors": [],
  "year": null,
  "journal": "",
  "doi": "",
  "url": "",
  "abstract": "",
  "source_api": "arxiv | semantic-scholar | openalex | journal-site | manual",
  "source_type": "abstract-only | full-text",
  "collected_at": "",
  "collected_by_team": ""
}
```

## 논문 요약 페이지 템플릿 (`{wiki_path}/papers/{slug}.md`)

주의: 이 템플릿은 실제 로컬 작업(root 프로젝트, `data/pipeline/` 동기화 대상)
전용이다. `research-agent-office` 공개 저장소를 빌드할 때는 이 템플릿을 그대로
싣지 않고, 프론트매터·확장 섹션이 없는 단순 버전(연구문제/방법론/핵심
기여/한계/관련 개념/관련 논문 6단 구성)으로 대체한다. 이 팀의 실제 연구
도메인(도시연구 등)에 맞춘 필드(`region`, `data_period`, Discussion 섹션 등)가
다른 분야를 다루는 공개 템플릿 사용자에게는 맞지 않기 때문이다. 원본 단순
버전은 `MEMORY.md`의 "공개 저장소용 단순 논문 템플릿" 항목에 보존되어
있다.

아래 템플릿의 괄호 안 문구((이 논문이 풀려는 문제, 왜 필요한가)처럼 생긴 것)는
**이 문서를 읽는 Claude를 위한 작성 지침이지, 실제 위키 페이지에 그대로 옮겨
적는 문장이 아니다.** 실제 논문 페이지를 쓸 때는 이 괄호 문구를 지우고 그
자리에 실제 내용을 채운다 - 지침 문장이 실제 페이지에 그대로 남아있으면 안
된다.

```markdown
---
title: "{논문 제목(영문 원제)}"
authors: []
year: {연도}
journal: "{저널명}"
doi: "{doi}"
url: "{url}"
apa: "{APA 7판 인용}"
region: ""
data_period: ""
importance: "★★★ 핵심 | ★★☆ 유용 | ★☆☆ 참고"
read: true
methods: []
tags: []
type: paper
source_api: "{source_api}"
source_type: "{source_type}"
collected_at: "{collected_at}"
---

# {논문 제목(영문 원제)} ({연도})
### {한국어 번역 제목}

- 저자:
- 저널: **{저널명}**
- 링크/DOI:
- 원본 메타데이터: Research Office data 저장소 `data/{team-path}/raw/{slug}.json` (private repo)
- 출처: {source_api}, 수집일 {collected_at} ({source_type})

## 연구 목적 및 배경
(이 논문이 풀려는 문제, 왜 필요한가 - 충분히 구체적으로 서술한다)

## 방법론
(핵심 아이디어를 충분히 구체적으로 - 데이터, 분석 기법, 절차를 포함)

## 연구의 범위
- 공간적 범위: (어떤 지역/공간 단위를 다루는지, 해당 없으면 생략)
- 시간적 범위: (어떤 기간/시점의 데이터인지, 해당 없으면 생략)
- 내용적 범위: (이 논문이 실제로 다루는 개념/지표/관계의 범위, 다루지 않는 것도
  있으면 함께)

## Research Framework
(**항상 그린다 - 생략하지 않는다.** 연구의 데이터 → 변수 구성 → 분석 기법 → 결과로
이어지는 전체 파이프라인을 한눈에 파악할 수 있도록, 단순한 연구든 복잡한 연구든 그
구조에 맞는 수준까지 자세히 그린다. `.claude/skills/markdown-mermaid-writing/`
스킬(로컬 프로젝트 스킬로 복제해둠, 출처는 `CLAUDE.md`의 "외부 스킬 연동" 참고)의
복잡도 등급을 그대로 따른다: 10개 이하 노드는 subgraph 없이 평평하게, 10~20개는
subgraph로 2~4개 논리 그룹으로 묶고, 20~30개는 subgraph를 반드시 쓰며(3~6개), 30개를
넘으면 개요+세부 다이어그램으로 분리한다. subgraph는 데이터/변수 구성/분석 기법/결과처럼
연구 단계별로 묶는 게 기본이고, 세부 분류가 필요하면 한 단계까지 중첩해도 된다
(예: 독립변수 subgraph 안에 이론적 하위범주별 subgraph).

방향은 데이터→결과로 흐르는 일반적인 분석 파이프라인이면 `flowchart TB`(세로)를
쓰고, SEM 경로도처럼 자극→반응 구조가 자연스러운 경우에만 `flowchart LR`(가로)을
쓴다. 노드 라벨은 3~6단어로 짧게 유지하되, `<br/>`로 한 번까지는 줄바꿈해서
기간·표본수 같은 핵심 수치를 함께 적어도 된다 - 복잡도는 라벨을 늘려서가 아니라
subgraph 구조로 표현한다. `accTitle`/`accDescr`를 반드시 넣어 다이어그램이 무엇을
보여주는지 한두 문장으로 설명한다.

색은 **무채색 4단 그레이스케일만** 쓴다(화려한 색은 쓰지 않는다 - 이 위키는 표/토글로
이미 정보 밀도가 높아 색까지 화려하면 산만해진다는 판단). 파이프라인의 앞 단계(데이터/
자극)일수록 밝게, 뒷 단계(결과/반응)일수록 어둡게 4단으로 배정한다:

```
classDef data_style     fill:#f9fafb,stroke:#d1d5db,stroke-width:2px,color:#374151
classDef variable_style fill:#e5e7eb,stroke:#9ca3af,stroke-width:2px,color:#1f2937
classDef method_style   fill:#d1d5db,stroke:#6b7280,stroke-width:2px,color:#111827
classDef result_style   fill:#9ca3af,stroke:#4b5563,stroke-width:2px,color:#030712
```

인라인 `style`은 쓰지 않고 `classDef`만 쓴다. **`classDef` 이름은 subgraph id와
절대 겹치면 안 된다** - 겹치면 렌더링 오류가 난다(실제로 겪은 문제). 그래서 클래스
이름에 항상 `_style` 접미사를 붙인다(예: subgraph id `data` + classDef
`data_style`). 매개/간접효과처럼 실선과 구분해야 하는 관계는 점선 화살표에 표준
라벨 문법(`A -.->|label| B`)을 쓴다 - `A -. label .-> B` 형태는 파싱 오류를 일으킨
적이 있어 쓰지 않는다.

mermaid 코드펜스를 노트에 직접 쓴다(별도 파일로 렌더링해 이미지로 심는 방식은
시도했다가 되돌렸다 - 논문마다 파일이 2개씩 더 생기는 게 번거롭다는 피드백).
**`<div>` 등 HTML로 감싸지 않는다** - `<div>`와 코드펜스 사이에 빈 줄을 둬도
Obsidian이 `</div>`를 만날 때까지 그 사이 전체를 raw HTML로 삼켜서, 코드펜스가
mermaid로 렌더링되지 않고 줄바꿈까지 뭉개진 텍스트로 그대로 노출된다(실제로 겪은
문제, Reading view에서 확인). 마크다운 소스에는 순수 mermaid 코드펜스만 쓴다 -
여는 줄은 백틱 3개 뒤에 바로 `mermaid`, 그 아래에 다이어그램 내용, 마지막 줄은
백틱 3개만 있는 구조를 그대로 쓰고 다른 문법으로 감싸지 않는다.

다이어그램 크기는 각 팀 Obsidian vault의 `.obsidian/snippets/mermaid-scroll.css`
CSS 스니펫으로 제어한다 - 마크다운 소스를 건드리지 않고 Obsidian이 코드펜스를
자동으로 감싸는 자체 래퍼(`.block-language-mermaid`)에 CSS만 적용하는 방식이라
위 raw HTML 문제와 무관하다. `.block-language-mermaid`의 `max-width`를 100%로
막고, 내부 `svg`에 `max-width:100% !important; height:auto !important`를 줘서
노트 폭에 맞춰 가로는 자동 축소되게 하고, **세로는 별도로 제한하지 않아 다이어그램
높이만큼 박스가 그대로 늘어난다**(스크롤 없음). 중간에 두 가지를 시도했다가
버렸다 - 사용자가 직접 크기를 드래그로 조절하는 리사이즈 핸들(`resize:both`,
`!important`까지 붙여도 이 Obsidian 빌드에서는 핸들 자체가 안 나타남), 세로
고정 최대높이 + 스크롤(다이어그램이 조금만 길어도 매번 스크롤해야 해서 불편하다는
피드백). vault의
`.obsidian/appearance.json` `enabledCssSnippets`에 `"mermaid-scroll"`이 켜져
있어야 적용된다. 새 Research Team을 `/team-add`로 만들 때 이 팀이 처음
Obsidian에서 열려 `.obsidian/` 폴더가 생기면, 기존 팀(`online-vitality`,
`test-demo-rag`)의 `.obsidian/snippets/mermaid-scroll.css`를 그대로 복사하고
`appearance.json`에 같은 설정을 넣어준다.

## 데이터 및 변수
(실증/정량 연구가 아니면 이 섹션 전체를 생략한다. 논문에서 확인 가능한 변수
정보는 축약하지 말고 빠짐없이 적는다 - 변수명만 나열하지 않는다. 변수 하나당
"측정 방법"/"측정 목적"/"단위"/"출처" 네 줄로 나눠 쓴다(측정 시점은 별도
줄을 두지 않고 `출처 (연도)` 형태로 출처 줄에 합쳐 쓴다). 변수명은 논문 원문
영문 표기와 한글 번역을 `영문 / 한글` 형태로 함께 적는다. **논문이 여러 하위
지표를 하나로 묶어 부르더라도 ("POI 다양성 6종", "3D 지표 5개" 처럼) 실제로
개별 측정값·계수가 따로 있는 지표라면 절대 한 항목으로 뭉치지 않는다** - 하위
지표 각각을 자기만의 4줄 그룹으로 나눈다. 진짜로 하나의 값으로만 측정·보고된
경우에만 한 항목으로 쓴다.

**표는 마크다운이 아니라 HTML로 쓴다** - 순수 마크다운 표는 내용 길이에 따라
칸 너비가 표마다 제각각으로 자동 계산돼서, 이 섹션처럼 표가 여러 개(카테고리별)
나오면 표끼리 크기가 안 맞아 보인다는 피드백을 받았다. `<table
style="width:100%; table-layout: fixed;">`로 페이지 너비에 맞춘 고정
레이아웃을 쓰고, `<th style="width:35%">`처럼 칼럼별 비율(변수명 35% / 구분
15% / 내용 50%)을 명시한다. 변수명 칸은 `rowspan="4"`로 실제 병합해서
반복·공백 없이 깔끔하게 보이게 한다. Obsidian은 이 HTML을 그대로 렌더링한다.)

### 종속변수

<table style="width:100%; table-layout: fixed;">
<tr><th style="width:35%">변수명</th><th style="width:15%">구분</th><th style="width:50%">내용</th></tr>
<tr><td rowspan="4">{영문 / 한글}</td><td>측정 방법</td><td></td></tr>
<tr><td>측정 목적</td><td></td></tr>
<tr><td>단위</td><td></td></tr>
<tr><td>출처</td><td>{출처 (연도)}</td></tr>
</table>

### 독립변수
(카테고리를 표 칼럼으로 넣지 않고 **카테고리마다 `####` 소제목을 달아 표를
나눈다** - 소제목 하나로 카테고리를 한 번만 밝히는 게 훨씬 잘 읽힌다. 카테고리
자체가 더 세분화될 수 있으면 `####` 아래 `#####`로 한 단계 더 중첩해도 된다 -
카테고리 단계 수를 하나로 고정하지 않는다. 한 카테고리 안에 변수가 여럿이면
`<table>` 하나 안에 변수마다 `rowspan="4"` 블록을 이어붙인다 - 헤더는
카테고리당 한 번만 나온다.)

#### {카테고리명}

<table style="width:100%; table-layout: fixed;">
<tr><th style="width:35%">변수명</th><th style="width:15%">구분</th><th style="width:50%">내용</th></tr>
<tr><td rowspan="4">{영문 / 한글}</td><td>측정 방법</td><td></td></tr>
<tr><td>측정 목적</td><td></td></tr>
<tr><td>단위</td><td></td></tr>
<tr><td>출처</td><td>{출처 (연도)}</td></tr>
<tr><td rowspan="4">{영문 / 한글 2}</td><td>측정 방법</td><td></td></tr>
<tr><td>측정 목적</td><td></td></tr>
<tr><td>단위</td><td></td></tr>
<tr><td>출처</td><td>{출처 (연도)}</td></tr>
</table>

#### {카테고리명2}

<table style="width:100%; table-layout: fixed;">
<tr><th style="width:35%">변수명</th><th style="width:15%">구분</th><th style="width:50%">내용</th></tr>
<tr><td rowspan="4">{영문 / 한글}</td><td>측정 방법</td><td></td></tr>
<tr><td>측정 목적</td><td></td></tr>
<tr><td>단위</td><td></td></tr>
<tr><td>출처</td><td>{출처 (연도)}</td></tr>
</table>

### 통제변수
(논문에 통제변수가 없으면 이 하위 섹션 자체를 생략한다. 통제변수도 카테고리별로
나뉜다면 독립변수와 같은 방식으로 `####` 소제목을 쓴다.)

<table style="width:100%; table-layout: fixed;">
<tr><th style="width:35%">변수명</th><th style="width:15%">구분</th><th style="width:50%">내용</th></tr>
<tr><td rowspan="4">{영문 / 한글}</td><td>측정 방법</td><td></td></tr>
<tr><td>측정 목적</td><td></td></tr>
<tr><td>단위</td><td></td></tr>
<tr><td>출처</td><td>{출처 (연도)}</td></tr>
</table>

## 주요 결과

## Discussion (이론적·정책적 함의)
(논문 자체의 Discussion/Conclusion에서 실제로 논의한 내용을 정리한다. 저자가
다루지 않은 함의를 지어내지 않는다. 해당 없는 줄은 생략)
- 이론적 함의:
- 정책적·실무적 함의:
- 향후 연구 제언:

> [!note]- 변수별 해석
> ("주요 결과"가 수치·통계량 같은 사실만 담는다면, 여기는 논문의 Discussion이
> 그 수치를 **왜 그렇게 나왔다고 해석했는지**를 변수별로 정리하는 자리다 -
> 유의했던 변수마다 저자가 실제로 제시한 해석을 적는다. 저자가 해석을 안 한
> 변수는 이 목록에서 뺀다(지어내지 않음). 접이식 콜아웃으로 감싸는 이유는
> 변수가 많으면 이 목록이 아주 길어질 수 있어서다. **각 줄은 반드시 `> -`로
> 시작한다** - 콜아웃 안 불릿 목록이 한 줄이라도 `>`만 있고 `-`가 빠지면 그
> 줄부터 굵게/불릿이 깨져 보인다. HTML `<details>`는 쓰지 않는다 - Obsidian이
> 태그 안 마크다운(굵게 `**`, 불릿 `-`)을 파싱하지 않고 그대로 raw 텍스트로
> 보여준다(실제로 겪은 문제).)
>
> - **{변수명}**: (저자가 이 변수의 결과를 어떻게 해석했는지 - 왜 이 방향/크기로
>   나타났다고 설명하는지)
> - **{변수명2}**: ...

## 연구의 한계
- 

## 관련 개념
- [[개념명]]

## 관련 논문
- [[다른-논문]] - 어떤 관계인지 한 줄

## 주요 인용 문헌
- 
```

`importance`와 `read`는 Paper Collector가 초안으로 채우되(예: 신규 반영 논문은
`read: true`, importance는 팀 연구와의 관련성으로 판단), 사용자가 Obsidian에서
직접 조정할 수 있는 필드다. `apa`는 저자·연도·제목·저널·권(호)·페이지·DOI를 갖춘
APA 7판 형식 인용 문자열이다(예: `Osunkoya, K. M., Väisänen, T., Partanen, J., &
Järv, O. (2026). Mapping urban vitality through dynamic population presence.
Environment and Planning B. https://doi.org/...`) - **본문에는 표시하지 않고
프론트매터에만 채운다**(본문에 넣기엔 너무 길다는 피드백으로 제거, 대신 아래
"논문 아카이브 Base 뷰"의 정렬·표 컬럼 용도로만 쓴다). `tags`는 이 논문의
핵심 키워드를 Obsidian 태그 규칙(공백 없이 소문자-하이픈)으로 적는 배열이다 -
Obsidian 그래프 뷰/태그 패널에 그대로 노출된다(예: `tags:
["third-place-theory", "urban-vitality"]`). **예전에는 `keywords`(사람이 읽는
표현)와 `tags`(kebab-case 미러링)를 따로 뒀지만 사실상 같은 정보의 중복이라는
지적으로 `tags` 하나로 통일했다** - Base 뷰의 키워드 칼럼도 `tags`를 그대로
쓴다. 아래 "논문 아카이브 Base 뷰"가 이 프론트매터를 근거로 표/카드 뷰를
만든다.

## 논문 아카이브 Base 뷰 (`{wiki_path}/papers.base`)

Obsidian Base 파일로, 팀 위키의 `papers/` 아래 `type: paper` 프론트매터를 가진
모든 논문 페이지를 표/카드로 모아 보여준다. 새 팀을 만들 때 `/team-add`가
`wanted.md`, `index.md`와 함께 이 파일도 스캐폴딩한다 (아래 템플릿 그대로,
빈 vault에서는 표시할 논문이 없을 뿐 파일 자체는 미리 만들어둔다).

```yaml
filters:
  and:
    - 'type == "paper"'

views:
  - type: table
    name: "전체 논문"
    order:
      - apa
      - file.name
      - year
      - journal
      - importance
      - read
      - methods
      - tags
    sort:
      - property: apa
        direction: ASC

  - type: table
    name: "읽지 않음"
    filters:
      and:
        - 'read == false'
    order:
      - apa
      - file.name
      - year
      - journal
      - importance
    sort:
      - property: apa
        direction: ASC

  - type: table
    name: "핵심 논문"
    filters:
      and:
        - 'importance == "★★★ 핵심"'
    order:
      - apa
      - file.name
      - year
      - journal
      - tags
    sort:
      - property: apa
        direction: ASC

  - type: cards
    name: "카드 뷰"
    order:
      - apa
      - file.name
      - year
      - journal
      - importance
```

정렬을 APA 인용 문자열(`apa`) 오름차순으로 두는 이유는 APA 참고문헌 목록 관례(제1
저자 성 알파벳순)를 따르기 위해서다 - `apa` 필드가 항상 `저자(연도)` 형태로
시작하므로 문자열 정렬만으로 이 순서가 그대로 나온다.

이 Base 뷰도 위 논문 템플릿과 마찬가지로 **로컬 작업/private 전용**이다 - 공개
저장소 빌드에는 포함하지 않는다.

## 개념(Concept) 페이지 템플릿

기본 템플릿:

```markdown
# {개념명}

## 정의
(한두 문장으로)

## 왜 중요한가 / 왜 등장했는가
(이 개념이 필요해진 배경이나 풀려던 문제)

## 대표 방법론/사례
- {방법론명} - 한 줄 설명 → [[논문-slug]]

## 관련 논문
- [[논문-slug]]
```

### 핵심/기초 개념 확장 규칙

다음 중 하나에 해당하면 "핵심 개념"으로 간주하고 확장 템플릿을 쓴다:
- 위키에 추가된 논문 여러 편이 공통으로 전제/다루는 개념
- 분야에서 이미 정립된 전통적/보편적 개념
- 앞으로도 계속 다른 논문들이 참조할 가능성이 높은 토대(foundation) 개념

확장 템플릿은 "보편적 설명"과 "논문별 등장 방식" 섹션을 추가한다 (원본 스키마와 동일,
`research-wiki/schema.md` 참고).

## 비교(Comparison) 페이지 템플릿

```markdown
# {A} vs {B}

## 비교 기준
| 항목 | A | B |
|---|---|---|

## 결론
(언제 무엇을 선택해야 하는지)

## 참고
- [[논문-slug]]
- [[개념-slug]]
```

## Synthesis 페이지 템플릿 (`{wiki_path}/synthesis/latest.md`)

Research Synthesizer(`/synthesize <team-id>`)가 팀의 `{wiki_path}/papers`,
`{wiki_path}/concepts`, `{wiki_path}/comparisons` 전체를 다시 읽고 작성한다. 매번
전면 재작성하며, 이전 버전은 `{wiki_path}/synthesis/history/{date}.md`로 이동시킨 뒤
새로 쓴다.

```markdown
# Research Synthesis - {팀명} ({작성일})

## 이번 갱신 요약
(직전 synthesis 이후 추가된 논문 수와 무엇이 달라졌는지 2~3문장)

## 최근 연구 동향
(팀 위키 전체를 관통하는 흐름을 3~5개 포인트로)

## 핵심 개념 지도
- [[개념-slug]] - 한 줄 요약
  (concepts/의 핵심 개념들을 나열, 서로 어떻게 연결되는지 한 줄씩)

## 주요 방법론
(이 팀 위키에서 반복되는 방법론/데이터 소스 계열 정리)

## Emerging Topics
(아직 소수 논문만 다루지만 앞으로 중요해질 수 있는 주제)

## 다루는 논문
- [[논문-slug]] (연도)
  (이번 synthesis가 근거로 삼은 전체 논문 목록)
```

## Advisor 페이지 템플릿 (`{wiki_path}/advisor/latest.md`, Research Team 전용)

Research Advisor(`/advise <team-id>`)가 `{wiki_path}/synthesis/latest.md`, 팀 위키
전체, `data/{team-path}/my-research.md`(이 팀의 연구 프로필)를 근거로 작성한다.
Journal Watch 타입 팀에는 이 폴더도, `my-research.md`도 없다. 갱신 시 이전 버전은
`{wiki_path}/advisor/history/{date}.md`로 이동.

```markdown
# Research Advisor Insight - {팀명} ({작성일})

## 내 연구와의 관계
(이 팀의 my-research.md에 담긴 연구 주제·질문과 이 팀 위키가 어떻게 맞닿아 있는지)

## 이론적 고찰
(이 위키가 축적한 이론/개념이 내 연구의 이론적 토대에 주는 시사점)

## Research Gap
(이 위키에서 드러나는, 아직 다뤄지지 않았거나 부족한 부분 - 내 연구가 채울 수 있는 지점.
gap마다 `deep-research` quick 모드로 위키 밖 문헌을 점검한 결과를 한 줄로 덧붙인다 -
예: "(deep-research 점검: 미개척 영역으로 확인됨)" 또는 "(deep-research 점검: {유사
선행연구}가 부분적으로 다룸 - 이 지점에서 차별화 필요)")

## 주요 비교/개념
- [[comparison 또는 concept slug]] - 왜 내 연구에 중요한지

## 최근 트렌드가 내 연구에 주는 시사점
(synthesis의 최근 동향/emerging topics를 내 연구 맥락에서 재해석)

## 참고
- [[synthesis/latest]]
- [[논문-slug]] (인용한 논문들)
```

## 연구 프로필 스키마 (`{team-path}/my-research.md`, Research Team 전용)

**팀마다 독립된 연구 프로필을 갖는다.** 한 사용자가 서로 다른 연구 주제로 여러
Research Team을 운영할 수 있으므로, `my-research.md`는 `data/` 전역이 아니라 각
Research Team 폴더(`data/{team-path}/my-research.md`) 안에 있다. Journal Watch
타입 팀은 특정 연구주제에 매이지 않는 팀이므로 이 파일이 없다.

이 문서는 두 곳에서 쓰인다:

1. **Research Advisor**(`/advise <team-id>`)가 인사이트를 만들 때 항상 참고한다.
2. **Paper Collector**(`/paper-collect <team-id>`)가 이 팀에 어떤 논문을 모을지
   판단할 때도 참고한다 - `config.json`의 `topic`/`keywords`가 검색 질의를 만드는
   빠른 요약이라면, `my-research.md`는 "이 연구를 수행하는 데 필요한 이론적 고찰,
   관점, 개념, 방법론(데이터 포함)"이 무엇인지 알려주는 더 깊은 근거다. Collector는
   키워드 매칭만으로 좁게 검색하지 않고, `my-research.md`의 "이론적 배경"과
   "방법론"에 언급된 이론/개념/방법론/데이터 소스와 관련된 논문도 적극적으로
   찾는다.

```markdown
# 내 연구 프로필

## 연구 주제
(한두 문장으로 - 이 팀이 다루는 연구가 무엇인지)

## 핵심 연구 질문
- 

## 이론적 배경 / 관점
(어떤 이론·프레임워크를 기반으로 접근하는지)

## 연구의 범위
- 공간적 범위: (어떤 지역/공간 단위로 한정하는지, 해당하지 않으면 생략)
- 시간적 범위: (어떤 기간/시점으로 한정하는지, 해당하지 않으면 생략)
- 개념적 범위:
  - 포함하는 것: (이 연구가 실제로 다루는 개념/지표/관계)
  - 포함하지 않는 것: (의도적으로 범위 밖에 둔 것 - 다른 연구와의 경계를 분명히 함)

## 방법론
(데이터 소스, 분석 방법/도구를 포함해서 구체적으로)
```

`연구의 범위`는 세 축 중 이 연구와 무관한 축이 있으면 생략해도 된다 (예: 이론
중심 연구라 공간적 범위가 의미 없는 경우). 다만 **개념적 범위(포함/미포함)는
거의 모든 연구에 적용되므로 되도록 채운다** - 이게 빠지면 Collector가 관련
없는 인접 개념까지 논문으로 끌어올 위험이 커진다.

## 팀 설정 스키마 (`config.json`)

Journal Watch 타입 팀 (`data/trend-watch/config.json` - 이 프로젝트에서는 id도
`trend-watch`로 고정):

```json
{
  "field": "특정 세부 주제 없이 폭넓게 훑을 분야 (선택, 비워도 됨) - 예: '도시연구/건조환경'",
  "journals": [
    {"name": "", "url_or_feed": "", "notes": ""}
  ],
  "papers_per_day": {"min": 3, "max": 4},
  "last_run": null
}
```

"저널 우선순위" 규칙의 분야 판단은 이 `field` 값을 근거로 한다. `my-research.md`는
Research Team 전용이라 Journal Watch 타입 팀에는 참고할 프로필이 없다 - `field`가
비어 있으면 그때 사용자에게 직접 분야를 물어본다.

Research Team (`data/teams/<slug>/config.json`):

```json
{
  "id": "",
  "name": "",
  "topic": "",
  "keywords": [],
  "search_conditions": {
    "sources": ["arxiv", "semantic-scholar", "openalex", "sciencedirect"],
    "target_journals": ["선택 - 이 팀이 특히 노리는/추적할 구체적 저널명 목록 (우선순위 순)"],
    "notes": ""
  },
  "papers_per_day": {"min": 3, "max": 4},
  "last_run": null,
  "created": ""
}
```

**`keywords`는 방법론/데이터 용어로만 채우지 않는다.** `my-research.md`의
"이론적 배경 / 관점"에 나열된 갈래(고전 이론, 특정 이론명, 측정 전통, 정책적
관점 등) 각각에서 대표 키워드를 최소 1~2개씩 뽑아 골고루 넣는다 - 방법론
키워드(예: MGWR, GeoDetector, 통계 기법명)가 이론적 배경 키워드보다 많아지지
않게 한다. `keywords`가 방법론/데이터 용어 위주로 쏠려 있으면 "새 논문 판별
규칙"의 기본 검색 자체가 그쪽으로 편향된다(실제로 겪은 문제 - "특정 갈래로만
검색이 쏠리지 않게 한다" 참고). `/team-add`로 새로 만들 때나 `/team-edit`으로
고칠 때 `my-research.md`가 이미 채워져 있으면 그 "이론적 배경" 절을 참고해서
갈래별 균형을 맞춘다.

## 팀 레지스트리 스키마 (`data/teams.json`)

각 팀 항목의 `wiki_path`가 그 팀의 `{wiki_path}` 값이다 (프로젝트 루트 기준 상대경로,
`1. Wiki/<team-id>/` 아래 - 위 "저장 위치" 섹션의 절대경로 변환 규칙 참고):

```json
{
  "id": "",
  "type": "journal-watch | research-team",
  "name": "",
  "created": "",
  "path": "data/ 안에서의 상대경로 (예: trend-watch, teams/<slug>)",
  "wiki_path": "../../1. Wiki/<team-id> (프로젝트 루트 기준 상대경로)",
  "pipeline": ["collector", "synthesizer"] 또는 ["collector", "synthesizer", "advisor"]
}
```

## 외부 스킬 연동 (academic-research-skills)

이 프로젝트는 `.claude/settings.json`의 `skillDirectories`로 외부 플러그인
`academic-research-skills`(`deep-research`, `academic-paper`,
`academic-paper-reviewer`, `academic-pipeline` 4개 스킬)를 이 프로젝트 범위에서만
사용할 수 있게 연결해뒀다.

- **`/synthesize`**: 팀 위키를 종합할 때 `deep-research` 스킬을 (가능하면 Skill 도구로,
  lit-review 또는 quick 모드) 호출해 그 방법론(체계적 문헌 검색 커버리지 점검,
  cross-source synthesis, 출처 검증)을 참고해 종합 품질을 높인다. 결과는 그대로 쓰지
  않고 이 프로젝트의 Synthesis 템플릿 형식으로 다시 정리해서 쓴다.
- **`/my-research-setup <team-id>`**: 해당 Research Team의
  `data/{team-path}/my-research.md`를 작성/갱신할 때 `deep-research` 스킬의
  Socratic guided research dialogue 모드를 호출해 연구 질문/이론적 배경을
  구체화하는 데 쓴다.
- **`/advise <team-id>`**: Advisor 페이지의 Research Gap 섹션 초안을 쓴 뒤,
  각 gap 후보를 `deep-research` 스킬의 quick 모드로 점검한다 - 이 팀의 위키 밖
  더 넓은 문헌에서 이미 다뤄진 주제는 아닌지 확인하는 용도다. gap을 기각하기
  위한 것이 아니라 경고 신호(advisory)로만 쓴다 - 이미 다뤄진 것으로 확인되면
  서술을 좁히거나 가장 가까운 기존 연구를 함께 언급하고, 미개척 영역으로
  확인되면 그대로 유지한다. 이 단계에서 찾은 문헌은 원문을 확보해 위키에
  정식 반영하지 않는다.
- **`/concept-review <team-id>`**: `concepts/`/`comparisons/`에서 찾은 병합·재구성
  후보를 실제로 고치기 전에 `deep-research` 스킬을 quick 모드로 호출해, 정말 같은
  개념/비교인지 재확인한다.
- **`/paper-collect`(일일 3~4편 수집)**: 매일 도는 가벼운 작업이라 매번 무거운
  다중 에이전트 파이프라인을 도는 대신, 기존처럼 WebSearch/WebFetch 기반으로 유지한다
  (저널 우선순위 판단에는 위에서 이미 `top_journals_by_field.md`를 참고 자료로 연결
  해뒀다). deep-research를 매 논문마다 전체 호출하는 것은 과함 - 이 판단이 바뀌길
  원하면 사용자가 명시적으로 다시 요청할 것.
- Skill 도구 목록에 4개 스킬이 아직 안 보이면, `skillDirectories` 설정은 보통 세션
  시작 시점에 로드되므로 Claude Code 세션을 재시작해야 할 수 있다. 재시작 전까지는
  이 커맨드들이 해당 스킬의 `SKILL.md`/`references/`를 직접 Read해서 방법론만
  참고하는 방식으로 대체 동작한다.

## 질문 응답 시 (Query) 절차

1. 먼저 팀의 `{wiki_path}/` 안에서 답변에 필요한 정보를 찾는다 (`raw/`를 매번 다시
   읽지 않음).
2. `{wiki_path}/`에 없는 정보가 필요하면 그때만 `data/{team-path}/raw/` 원본
   (메타데이터 또는 PDF)을 확인.
3. 답변 후, 나중에 재사용할 가치가 있는 종합/비교라면 `{wiki_path}/comparisons/`에
   새 페이지로 저장할지 사용자에게 물어본다.

## 인용 규칙

- 모든 사실 주장에는 `[논문 slug]` 형태로 출처 표시.
- 논문 원문을 통째로 옮기지 않고 요약 위주로 작성.
- `{wiki_path}/*.md`의 내용은 항상 `data/{team-path}/raw/`로 추적 가능해야 한다 - 새
  사실을 추가할 때 근거 없는 raw가 없다면 wiki에도 쓰지 않는다.

## Lint / 위키 정리 절차 (`/wiki-review <team-id>`)

시간이 지나 논문이 쌓이면 위키에 여러 문제가 누적된다: (1) 나중에 추가된
논문/개념이 예전 논문의 "주요 인용 문헌"에 평문으로만 적혀 있어 `[[링크]]`로
연결되지 않은 상태로 남고, (2) 여러 세션에 걸쳐 비슷한 태그(단수/복수, 하이픈
유무, 한/영 표기 차이 등)가 조금씩 다른 형태로 계속 새로 생기고, (3)
`/paper-collect`가 논문을 한 편씩만 보고 판단하다 보니 여러 논문에 걸쳐서만
드러나는 공통 개념이 `concepts/`로 승격되지 못한 채 남는다. `/wiki-review
<team-id>`가 이들을 포함해 위키 상태를 점검하고, **사용자 확인을 받은 뒤 실제로
고친다** (기존 점검 항목은 보고만 하던 것과 다름 - 아래 구분 참고).

**점검만 하고 보고만 하는 항목** (자동으로 고치지 않음 - 사용자 판단 필요):
- 어떤 페이지에서도 링크되지 않는 "고아 페이지"가 있는가
- `concepts/` 페이지 간 서로 모순되는 설명이 있는가
- `synthesis/latest.md`가 최신 논문 추가 이후 갱신되지 않은 채로 오래됐는가
- (Research Team만) `advisor/latest.md`가 synthesis 갱신 이후 갱신되지 않았는가

**점검 후 확인받아 실제로 고치는 항목**:
- `papers/`에는 있는데 `index.md`에는 반영 안 된 논문 → `index.md`에 추가.
- `raw/`에는 있는데 `papers/`에 대응 페이지가 없는 논문(역방향 누락) → 발견
  사실을 보고(자동 생성은 하지 않음 - 전문을 다시 읽어 정식으로 반영해야 하므로
  `/paper-collect`나 `/paper-review-pdf`로 유도).
- **인용/참조 백필링**: 논문 페이지의 "주요 인용 문헌"/"관련 논문"/"관련 개념"에
  평문으로만 적힌 인용이, 그 사이 위키에 실제로 추가된 논문·개념과 저자/제목/
  개념명이 일치하면 `[[slug]]` 링크로 교체할 후보로 제안. 확실하지 않으면(저자
  성만 같고 주제가 다른 경우 등) 후보에서 제외 - 잘못된 링크보다 놓치는 편이
  낫다.
- **태그 근접 중복 정리**: 팀 전체 논문의 `tags`를 모아, 같은 개념을 가리키는
  근접 중복(단수/복수, 하이픈 유무, 한/영 표기, 유사어)을 찾아 대표 표기로
  통합할 안을 제안. 애매하면(다른 개념일 수도 있으면) 묶지 않는다.
- **개념 페이지 신설 후보 제안**: `/paper-collect`의 Ingest 절차는 논문을 한 편씩
  반영하면서 그 논문 하나만 놓고 "재사용 가능한가"를 판단하기 때문에, 여러 논문에
  걸쳐 흩어져 있지만 한 편씩 볼 때는 눈에 띄지 않는 공통 개념/방법론/이론적 틀을
  놓칠 수 있다. `/wiki-review`는 위키 전체를 한 번에 훑으므로 이 사각지대를
  다시 점검한다: 세 편 이상의 논문이 본문(방법론/데이터 및 변수/Discussion 등)에서
  공통으로 다루는데도 아직 `concepts/`에 대응 페이지가 없는 개념이 있으면, "핵심/
  기초 개념 확장 규칙"(여러 논문 공통 전제 / 분야에서 정립된 개념 / 앞으로도 계속
  참조될 토대 개념) 기준에 비춰 신설 후보로 제안한다. 단순히 같은 단어가 우연히
  겹치는 경우 등 확실하지 않으면 후보에서 제외 - 억지로 개념을 만들어내지 않는다.

세 "실제로 고치는" 항목 모두 **고치기 전에 후보 목록을 사용자에게 보여주고
전체/항목별/중단 중 확인받는다** - 확인 없이 파일을 고치거나 새로 만들지 않는다.
개념 페이지 신설이 확정되면 위 "개념(Concept) 페이지 템플릿"과 "핵심/기초 개념
확장 규칙"을 그대로 적용해 기본/확장 템플릿 중 맞는 쪽으로 생성하고, 그 개념을
다루는 논문들의 "관련 개념" 섹션과 `index.md`의 개념 목록에도 새 링크를 추가한다.
정확한 절차는 `.claude/commands/wiki-review.md` 참고.

## Concept/Comparison 정리 절차 (`/concept-review <team-id>`)

`/wiki-review`는 페이지 단위 점검(고아 페이지, 인용 백필링, 태그 중복, 개념 신설
후보)이 중심이라, `concepts/`와 `comparisons/`가 계속 쌓이면서 생기는 **구조적
중복**(사실상 같은 개념/비교를 다른 slug로 따로 만든 경우)까지는 다루지 않는다.
`/concept-review <team-id>`가 이 둘을 대상으로 전체를 훑어 중복 병합·재구성 후보를
찾고, **사용자 확인을 받은 뒤 실제로 고친다** (안전장치는 `/wiki-review`와 동일한
"먼저 목록 제시 → 전체/항목별/중단 확인 → 확인된 것만 수정" 패턴).

**점검만 하고 보고만 하는 항목**:
- 개념 페이지 사이에 상위/하위 또는 인접 관계가 있어 보이는데 서로 링크되어 있지
  않은 경우 (예: 세부 개념이 상위 개념을 언급하지 않음). 개념 템플릿에는 "관련
  개념" 필드가 없으므로, 연결 방식(템플릿 확장 여부)은 사용자 판단이 필요해 자동
  수정하지 않는다.

**점검 후 확인받아 실제로 고치는 항목**:
- **개념 페이지 병합**: `concepts/`의 모든 페이지를 서로 비교해, 제목/정의가
  사실상 같은 대상을 가리키거나(동의어, 표기 차이) "관련 논문" 목록이 크게
  겹치는 근접 중복 쌍을 찾는다. 확실한 경우만 병합 후보로 제안하고, 관련은 있지만
  서로 다른 개념이면(포함 관계·인접 개념 등) 병합하지 않고 위 "보고만" 항목으로
  넘긴다.
- **비교 페이지 병합/재구성**: `comparisons/`의 모든 페이지를 서로 비교해, 같은
  대상 쌍(A vs B)을 다른 slug로 중복 작성한 경우를 찾아 병합 후보로 제안한다.
  A vs B, A vs C처럼 한 대상(A)을 축으로 여러 개별 비교가 흩어져 있고 사실상 A/B/C
  다자간 비교로 보는 게 더 명확하면, 하나의 다자간 비교 페이지로 재구성하는 안도
  제안할 수 있다 - 이 경우도 병합과 동일하게 확인 후에만 진행한다.
- **개념 정의 갱신**: 어떤 개념 페이지의 "관련 논문"에는 없지만, 이후 추가된
  논문이 본문(방법론/Discussion 등)에서 그 개념을 실제로 다루고 있으면, 그 논문을
  반영해 "정의"/"왜 중요한가"/"대표 방법론" 초안을 갱신할 후보로 제안한다(기존
  내용을 지우지 않고 확장하는 방향 - 논문이 실제로 뒷받침하지 않는 내용을 지어내지
  않는다).

병합이 확정되면: 살아남는 쪽 slug 하나로 두 페이지의 내용(정의/왜 중요한가/대표
방법론/관련 논문 또는 비교 기준/결론/참고)을 합쳐 다시 쓰고, 사라지는 slug를
가리키던 위키 전체(`papers/`, `concepts/`, `comparisons/`, `synthesis/latest.md`,
`advisor/latest.md`(있으면), `index.md`)의 `[[링크]]`를 살아남는 slug로 일괄
치환한다. **사라지는 쪽 페이지 파일은 실제로 지우지 않는다** - 위키 폴더는 git
버전관리 대상이 아니라 삭제가 되돌리기 어렵다. 대신 그 페이지 내용을 "이 {개념/
비교}는 [[살아남는 slug]]로 통합되었습니다."라는 한 줄짜리 리다이렉트 스텁으로
바꿔 남긴다.

**외부 스킬 연동**: 병합·재구성 후보를 판단할 때 `deep-research` 스킬(가능하면
Skill 도구로, quick 모드)을 참고해 후보의 타당성(정말 같은 개념/비교인지, 병합
후 놓치는 뉘앙스는 없는지)을 재확인한다 - `/synthesize`와 동일한 연동 방식
("외부 스킬 연동" 섹션 참고).

정확한 절차는 `.claude/commands/concept-review.md` 참고.
