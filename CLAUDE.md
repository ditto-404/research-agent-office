# Research Office

AI Agent들이 논문을 자동 수집하고, 팀별로 완전히 독립된 Wiki(Obsidian Vault)에 축적하며,
이를 바탕으로 연구 동향과 사용자 연구에 대한 인사이트를 제공하는 파이프라인.

이 저장소는 GitHub **public** 템플릿이다 - 파이프라인 코드/커맨드/스키마만 담고 있으며
개인 연구 데이터는 없다. 실제 데이터(수집한 논문, 팀 레지스트리, 연구 프로필)는 별도의
독립된 **private** git 저장소 `data/`에 둔다. 두 저장소는 절대 섞이지 않는다.

## 구조

```
CLAUDE.md                     이 문서
README.md                     프로젝트 소개 (한/영 미러링)
schema/schema.md              위키 운영 규칙 - 모든 팀이 공유하는 단일 진실
schema/templates/*            paper / concept / comparison / synthesis / advisor / wanted / my-research / team-config 템플릿
.claude/commands/*.md         Agent 파이프라인 = 커스텀 슬래시 커맨드
examples/                     실제로 파이프라인을 돌려서 만든 예시 팀 전체
data/                         독립 git 저장소 (여기 없음). raw 메타데이터 + 팀 레지스트리(teams.json)
```

팀의 **위키 콘텐츠 자체**(papers/concepts/comparisons/synthesis/advisor/index.md)는
`data/` 안이 아니라 사용자가 지정한 Obsidian Vault 폴더(`<WIKI_VAULT_ROOT>/<team-id>/`)에
생긴다 - git이 아니라 OneDrive/iCloud 등으로 동기화되는 일반 폴더다. raw/config만
`data/`에 git으로 버전관리된다. 왜 이렇게 나뉘는지, 각 팀의 위키 경로를 어떻게 찾는지는
`schema/schema.md`의 "저장 위치" 섹션 참고.

`data/`의 세부 구조와 팀 데이터 규칙은 자신의 private 데이터 저장소 쪽 문서를 참고할 것.
위키 문서 작성 규칙(어떤 섹션을 어떻게 채우는지)은 전부 `schema/schema.md`에 있다 - 이
문서가 Agent들의 행동 기준이다.

## 팀 종류 (두 종류, 구조는 동일 패턴)

1. **Journal Watch** (`data/<id>/` + `<WIKI_VAULT_ROOT>/<id>/`, 고정 1개) - 사용자가
   지정한 저널에서(또는 저널 없이 분야만 지정해서) 매일 신규 논문 3~4편 수집 → Wiki
   정리 → Synthesizer만 실행 (Advisor 없음).
2. **Research Team** (`data/teams/<slug>/` + `<WIKI_VAULT_ROOT>/<slug>/`, 사용자가
   자유롭게 추가/삭제/수정) - 팀마다 이름/주제·키워드/검색조건과 **독립된 연구 프로필**
   (`data/teams/<slug>/my-research.md`)을 가지고, 완전히 독립된 Wiki를 가짐. 매일
   3~4편 수집 → Synthesizer → **Advisor**까지 실행 (Advisor가 이 팀의
   `my-research.md`를 참고해 사용자 자신의 연구와 연결한 인사이트를 만드는 게
   Journal Watch와의 차이). `my-research.md`는 Collector가 논문을 찾을 때도
   참고한다 - 한 사용자가 서로 다른 연구 주제로 여러 팀을 운영할 수 있으므로 팀마다
   독립적이다.

모든 팀은 `data/teams.json`에 등록되어야 존재한다. 이 레지스트리가 동적 팀 구조의 핵심 -
각 항목이 `path`(raw/config 위치)와 `wiki_path`(위키 콘텐츠 위치) 둘 다 갖는다.

## 파이프라인

```
Journal Watch  → raw/ + wiki  → Synthesizer
Research Team  → raw/ + wiki  → Synthesizer → Advisor
```

각 화살표는 커스텀 슬래시 커맨드 하나에 대응한다:

| 커맨드 | 역할 |
|---|---|
| `/paper-collect <team-id>` | Paper Collector. 팀 설정과 my-research.md 기반으로 신규 논문 검색 → raw/ 메타데이터 + 위키 papers 요약 생성 |
| `/paper-review-pdf <team-id>` | Paper Collector (PDF 경로). 사용자가 raw/에 직접 넣은 PDF 원문을 전문 기반으로 위키에 반영 |
| `/synthesize <team-id>` | Research Synthesizer. 팀 위키 전체를 읽고 synthesis/latest.md 갱신 |
| `/advise <team-id>` | Research Advisor. Research Team 전용. synthesis + 이 팀의 my-research.md 기반 advisor/latest.md 갱신 |
| `/my-research-setup <team-id>` | 해당 팀의 `my-research.md` 작성/갱신 |
| `/team-add` | 새 Research Team 생성 (대화형) |
| `/team-edit <team-id>` | 팀 설정 수정 |
| `/team-remove <team-id>` | 팀 등록 해제 (데이터는 기본 보존) |
| `/journal-watch-setup` | Journal Watch의 저널 목록/조건 편집 |
| `/office-status` | 전체 팀 상태를 스캔해 `data/status.json` 갱신 |
| `/wiki-review <team-id>` | 팀 위키 정리. 고아 페이지·오래된 synthesis/advisor는 보고만, 인용 백필링·태그 중복 정리·개념 페이지 신설 후보는 확인 후 실제로 고침 |

## 원본 vs AI 생성 지식 분리 (중요)

- `raw/{slug}.json` (위치: `data/`) - API/웹에서 그대로 가져온 사실 정보(title, authors,
  abstract, DOI...). **AI가 해석/요약을 넣지 않는다.** 출처 추적의 기준점.
- 위키의 `**/*.md` (위치: 사용자의 Obsidian Vault) - AI(Claude)가 raw를 근거로 작성한
  해석/요약/종합. 항상 `raw/`를 인용한다.

이 분리와 인용 규칙의 정확한 형식은 `schema/schema.md`를 따른다.

## 논문 검색 소스

별도 API 키 없이 WebSearch/WebFetch로 arXiv, Semantic Scholar, OpenAlex, 저널 웹사이트/
RSS를 조회한다. Claude Code 세션 자체가 LLM 역할을 하므로 Synthesizer/Advisor도 별도
API 비용이 들지 않는다.

## 시작하기

새 프로젝트에 이 템플릿을 적용할 때는 README.md의 "설치" 절을 따른다 - 옆에 자신만의
`data/` private 저장소를 만들고, 위키로 쓸 폴더를 정한 뒤 `/team-add`로 첫 팀을
만들면 된다.
