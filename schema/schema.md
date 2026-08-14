# Research Office 위키 운영 규칙 (Schema)

이 문서는 Agent(Claude)가 팀별 위키를 어떻게 관리해야 하는지 정의한다. 모든 팀
(고정 Journal Watch 타입 팀 1개 + `data/teams/<slug>/`의 Research Team들)이 이
규칙을 공유한다. Karpathy의 "LLM Wiki" 패턴(raw 원문 불변, wiki는 AI가 관리)을
기반으로 (1) 자동 논문 수집에 맞게 "새 논문 판별"을 PDF가 아닌 메타데이터 기준으로
바꾸고, (2) `synthesis/`와 `advisor/` 두 산출물을 추가했고, (3) 원본
메타데이터(raw)와 위키 콘텐츠(wiki)의 저장 위치를 물리적으로 분리했다.

## 저장 위치 (raw는 data/, wiki는 Obsidian Vault)

이 문서와 커맨드들에서 쓰는 표기: `{team-path}` = `data/teams.json`의 해당 팀 항목의
`path` 필드 값(예: `<journal-watch-id>`, `teams/<slug>` - `data/` 기준 상대경로,
그 자체엔 `data/`가 안 붙어있음). `{wiki_path}` = 같은 항목의 `wiki_path` 필드 값.
그래서 raw/config는 항상 `data/{team-path}/...`로, 위키 콘텐츠는 항상
`{wiki_path}/...`로 (앞에 `data/`를 붙이지 않고) 표기한다.

`wiki_path`는 사용자가 정한, Obsidian Vault로 열 폴더(`<WIKI_VAULT_ROOT>`) 아래의
팀별 경로다(`<WIKI_VAULT_ROOT>/<team-id>/`) - 절대경로로 저장해도 되고, 프로젝트
루트 기준 상대경로로 저장해도 된다. Read/Write 도구는 절대경로만 받으므로, 상대경로로
저장했다면 실제로 쓸 때는 먼저 절대경로로 변환한 뒤 그 절대경로로 Read/Write한다.

- **`raw/`, `config.json`** - 고정 Journal Watch 타입 팀(`data/<journal-watch-id>/`)과
  각 Research Team(`data/teams/<slug>/`) 안에 있다. `data/`는 독립 git 저장소(private
  repo)이며, 소스 오브 트루스와 팀 레지스트리를 git으로 버전관리하기 위한 위치다.
- **`wiki/`의 실제 내용물(papers/concepts/comparisons/synthesis/advisor/index.md)**은
  `data/` 안이 아니라 `<WIKI_VAULT_ROOT>/<team-id>/`에 생성된다. 이 경로는 사용자가
  Obsidian Vault로 여는 폴더이며, git으로 버전관리하지 않는다(OneDrive/iCloud 동기화 +
  Obsidian 자체 설정에 맡긴다).
  - 팀별 폴더 하위에는 `wiki/`라는 중첩 폴더를 두지 않는다 - `<WIKI_VAULT_ROOT>/<team-id>/`
    자체가 그 팀의 위키 루트다 (`papers/`, `concepts/`, `comparisons/`, `synthesis/`,
    `advisor/`, `index.md`가 바로 그 밑에 온다).
  - 각 팀의 위키 위치는 `data/teams.json`의 해당 팀 항목 `wiki_path` 필드에
    기록되어 있다 - Agent는 이 필드를 읽고 절대경로로 변환해 어디에 쓸지 결정한다.
- 이 분리 때문에 논문 요약 페이지에서 원본 메타데이터를 가리킬 때 상대경로
  `raw/{slug}.json`을 쓸 수 없다 (더 이상 같은 폴더 트리에 있지 않음) - 아래 "논문 요약
  페이지 템플릿"의 형식을 따른다.

## 폴더 역할

- `raw/{slug}.json` (위치: `data/`) - 논문의 원본 메타데이터. API/웹에서 그대로
  가져온 사실 정보만 담는다 (title, authors, year, journal, doi, url, abstract,
  source_api, collected_at, collected_by_team, source_type). AI의 해석·요약·평가는
  절대 넣지 않는다. 오픈 액세스 논문(arXiv, 오픈 액세스 저널, PMC, 기관 리포지토리
  등)은 전문(full text)을 직접 읽어 `source_type: "full-text"`로 기록한다. **전문을
  확보하지 못한(페이월) 논문은 raw/wiki에 아예 반영하지 않는다** -
  `source_type: "abstract-only"`인 항목을 만들지 않는다. 대신 아래 `wanted.md`를
  참고. 사용자가 PDF를 직접 구해서 `raw/`에 넣으면(파일명은 아무거나 상관없다,
  `/paper-review-pdf`가 원문에서 제목/연도를 읽어 표준 slug로 리네임한다)
  `/paper-review-pdf`로 전문 기반 반영한다.
- `wanted.md` (위치: `<WIKI_VAULT_ROOT>/<team-id>/`) - Paper Collector가 검색은
  했지만 전문을 확보하지 못한 논문 중, 이 팀의 연구와 관련성이 높아 보이는 것만 골라
  올려두는 목록. raw나 wiki papers로 반영되지는 않은, "PDF를 구하면 좋을 후보"
  목록이다. 사용자가 PDF를 구해서 `/paper-review-pdf`로 반영하면 이 목록에서 빠진다.
- `papers/` (위치: `<WIKI_VAULT_ROOT>/<team-id>/`) - 논문 1편당 요약 페이지 1개.
  **AI가 raw를 근거로 작성한 해석.** 전문을 확보한(raw에 반영된) 논문만 여기 들어온다.
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
  구독형 출판사(Elsevier 등)의 저널도 arXiv/PMC와 동등하게 검색 대상이며, 저명도가
  충분하면 후보에서 배제하지 않는다(전문 확보 가능 여부는 아래 Ingest 절차에서
  별도로 판단).
- 어떤 분야의 "저명한 저널"인지 판단할 때는, WebSearch로 최신 저널 순위/impact
  factor를 확인한다. 로컬에 분야별 저널 순위 참고 자료가 있으면 먼저 그걸 확인하고,
  없으면 WebSearch로 보완한다.
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
   (소스: arXiv, Semantic Scholar, OpenAlex, 저널 웹사이트/RSS 등, 무료·API 키 불필요
   범위 내 - 출판사 이름만 보고 후보에서 제외하지 않는다). Research Team이면
   `topic`/`keywords`로 만든 기본 검색에 더해 `{team-path}/my-research.md`(있으면)의
   "이론적 배경"과 "방법론"에 나오는 이론/개념/방법론/데이터 소스와 관련된 논문도
   함께 찾는다 - 단순 키워드 매칭보다 넓게, 이 연구를 수행하는 데 실제로 필요한
   참고문헌을 찾는다는 기준으로 판단한다. **최근 트렌드 논문에만 치우치지 않는다** -
   `my-research.md`의 "이론적 배경 / 관점"에 특정 학자·이론명이 명시된 경우가 있으면,
   그 이론의 원 문헌이나 그 이론을 이 팀의 주제에 적용한 후속 연구도 검색 대상에
   포함한다(원 문헌이 유료·절판이라 전문을 못 구해도 무방 - 이 경우 2b 절차대로
   처리).
3. 후보의 제목을 slug로 정규화(소문자, 하이픈)하고 **끝에 발행년도를 밑줄로
   붙인다**(`{title-slug}_{year}`, 예: `example-paper-title_2026`) -
   같은 논문의 개정판·프리프린트가 연도만 다르게 재게재되는 경우를 구분하고,
   `papers/` 폴더를 파일명만으로도 연도순으로 훑어볼 수 있게 하기 위함. 이 slug로
   기존 `raw/` slug 및 `wanted.md` 목록과 대조 - 이미 있으면 제외.
4. 하루 목표 편수(팀 설정의 `papers_per_day`, 기본 3~4편)만큼 신규 후보를 추리고,
   **목록을 사용자에게 먼저 보여주고 확인**받는다 (자동 실행으로 전환되기 전까지는
   사람이 최종 확인).
5. 확인 후 아래 "Ingest 절차"를 각 논문에 대해 순서대로 수행한다 (전문을 확보하지
   못하면 raw/wiki에는 안 남고, 관련성이 높을 때만 `wanted.md`에 남는다).
6. 이미 `raw/`에 있는 논문은 별도 요청(재정리 등) 없이 건드리지 않는다.

## Ingest 절차 (신규 논문 추가)

1. **전문(full text) 확보를 먼저 시도한다.** 오픈 액세스 소스면 WebFetch로 전문
   페이지나 PDF를 직접 읽는다 - 예: arXiv는 초록만 있는 `arxiv.org/abs/{id}` 대신
   전문이 있는 `arxiv.org/pdf/{id}`나 `arxiv.org/html/{id}`를 쓴다. Semantic
   Scholar/OpenAlex 응답에 오픈 액세스 PDF 링크(`openAccessPdf` 등)가 있으면
   그것도 활용한다. 저널 웹사이트가 Open Access로 표시되어 있으면 그 페이지에서
   전문을 직접 읽는다 - 구독형 출판사 저널도 동일하게 적용한다: 논문 페이지에
   "Open Access" 배지나 CC BY 라이선스 표시가 있으면 그 페이지에서 전문을 직접
   읽고, 없으면(대부분 페이월) 2b로 간다.
2. 전문을 확보했으면 2a로, 페이월 등으로 확보하지 못했으면 2b로 간다.

   **2a. 전문 확보 성공 - 정식 반영**
   1. `data/{team-path}/raw/{slug}.json` 생성 - 아래 "원본 메타데이터 스키마" 형식,
      `source_type: "full-text"`. 사실만, 해석 없음.
   2. `{wiki_path}/papers/{slug}.md` 생성 - 아래 "논문 요약 페이지 템플릿" 형식.
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

```markdown
# {논문 제목} ({연도})

- 저자:
- 링크/DOI:
- 원본 메타데이터: private 데이터 저장소 `data/{team-path}/raw/{slug}.json`
- 출처: {source_api}, 수집일 {collected_at} ({source_type})

## 연구문제
(이 논문이 풀려는 문제가 뭔가)

## 방법론
(핵심 아이디어를 2~4문장으로)

## 핵심 기여
-

## 한계
-

## 관련 개념
- [[개념명]]

## 관련 논문
- [[다른-논문]] - 어떤 관계인지 한 줄
```

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

확장 템플릿은 기본 템플릿에 "보편적 설명"(개념 자체에 대한 일반적 설명)과 "논문별
등장 방식"(이 위키의 각 논문이 이 개념을 어떻게 다루는지) 두 섹션을 추가한다.

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
(이 위키에서 드러나는, 아직 다뤄지지 않았거나 부족한 부분 - 내 연구가 채울 수 있는 지점)

## 주요 비교/개념
- [[comparison 또는 concept slug]] - 왜 내 연구에 중요한지

## 최근 트렌드가 내 연구에 주는 시사점
(synthesis의 최근 동향/emerging topics를 내 연구 맥락에서 재해석)

## 참고
- [[synthesis/latest]]
- [[논문-slug]] (인용한 논문들)
```

**외부 스킬 연동**: Research Gap 섹션 초안을 쓴 뒤, 각 gap 후보를 `deep-research`
스킬(가능하면 Skill 도구로, quick 모드)로 점검해 이 팀의 위키 밖 문헌에서 이미
다뤄진 주제는 아닌지 확인한다. gap을 기각하기 위한 것이 아니라 경고 신호일
뿐이다 - 이미 다뤄진 것으로 확인되면 서술을 좁히거나 가장 가까운 기존 연구를
함께 언급하고, 미개척 영역으로 확인되면 그대로 유지한다.

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
- 공간적 범위: (어떤 지역/공간 단위를 다루는지, 해당 없으면 생략)
- 시간적 범위: (어떤 기간/시점의 데이터인지, 해당 없으면 생략)
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

Journal Watch 타입 팀 (`data/<journal-watch-id>/config.json` - id는 자유롭게
정하되, 폴더명과 통일하는 것을 권장):

```json
{
  "field": "특정 세부 주제 없이 폭넓게 훑을 분야 (선택, 비워도 됨)",
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
    "sources": ["arxiv", "semantic-scholar", "openalex"],
    "target_journals": ["선택 - 이 팀이 특히 노리는/추적할 구체적 저널명 목록 (우선순위 순)"],
    "notes": ""
  },
  "papers_per_day": {"min": 3, "max": 4},
  "last_run": null,
  "created": ""
}
```

## 팀 레지스트리 스키마 (`data/teams.json`)

각 팀 항목의 `wiki_path`가 그 팀의 `{wiki_path}` 값이다:

```json
{
  "id": "",
  "type": "journal-watch | research-team",
  "name": "",
  "created": "",
  "path": "data/ 안에서의 상대경로 (예: <journal-watch-id>, teams/<slug>)",
  "wiki_path": "<WIKI_VAULT_ROOT>/<team-id> (절대경로 또는 프로젝트 기준 상대경로)",
  "pipeline": ["collector", "synthesizer"] 또는 ["collector", "synthesizer", "advisor"]
}
```

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
논문/개념이 예전 논문의 "관련 논문"에 평문으로만 적혀 있어 `[[링크]]`로
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
- **인용/참조 백필링**: 논문 페이지의 "관련 논문"/"관련 개념"에 평문으로만 적힌
  인용이, 그 사이 위키에 실제로 추가된 논문·개념과 저자/제목/개념명이 일치하면
  `[[slug]]` 링크로 교체할 후보로 제안. 확실하지 않으면(저자 성만 같고 주제가
  다른 경우 등) 후보에서 제외 - 잘못된 링크보다 놓치는 편이 낫다.
- **태그 근접 중복 정리**: 팀 전체 논문의 태그를 모아, 같은 개념을 가리키는
  근접 중복(단수/복수, 하이픈 유무, 한/영 표기, 유사어)을 찾아 대표 표기로
  통합할 안을 제안. 애매하면(다른 개념일 수도 있으면) 묶지 않는다.
- **개념 페이지 신설 후보 제안**: `/paper-collect`의 Ingest 절차는 논문을 한 편씩
  반영하면서 그 논문 하나만 놓고 "재사용 가능한가"를 판단하기 때문에, 여러 논문에
  걸쳐 흩어져 있지만 한 편씩 볼 때는 눈에 띄지 않는 공통 개념/방법론/이론적 틀을
  놓칠 수 있다. `/wiki-review`는 위키 전체를 한 번에 훑으므로 이 사각지대를
  다시 점검한다: 세 편 이상의 논문이 본문(방법론/데이터·결과 등)에서 공통으로
  다루는데도 아직 `concepts/`에 대응 페이지가 없는 개념이 있으면, "핵심/기초
  개념 확장 규칙"(여러 논문 공통 전제 / 분야에서 정립된 개념 / 앞으로도 계속
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
  논문이 본문에서 그 개념을 실제로 다루고 있으면, 그 논문을 반영해 "정의"/"왜
  중요한가"/"대표 방법론" 초안을 갱신할 후보로 제안한다(기존 내용을 지우지 않고
  확장하는 방향 - 논문이 실제로 뒷받침하지 않는 내용을 지어내지 않는다).

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
후 놓치는 뉘앙스는 없는지)을 재확인한다 - `/synthesize`와 동일한 연동 방식.

정확한 절차는 `.claude/commands/concept-review.md` 참고.
