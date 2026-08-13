# Research Advisor Insight - Test Demo - RAG (2026-08-13)

## 내 연구와의 관계
`data/my-research.md`가 아직 템플릿 상태(비어 있음)라 사용자의 실제 연구 주제·질문을
확인할 수 없었다. 아래 내용은 이 팀 위키(논문 2편)만 근거로 한 일반적 분석이며,
`my-research.md`를 채운 뒤 다시 `/advise test-demo-rag`를 실행하면 사용자 연구와
직접 연결된 인사이트로 갱신된다.

## 이론적 고찰
이 위키가 다루는 두 논문은 RAG를 "복잡도"라는 단일 축으로 재조명한다. 서베이 논문의
4분류(검색기/생성기/하이브리드/견고성 중심) [rag-comprehensive-survey-architectures]는
설계 공간을 넓게 정리하지만, 실증 논문 [stronger-baselines-rag-long-context]은 정작
가장 단순한 접근(DOS RAG)이 강력하다는 점을 보여 "복잡한 아키텍처가 곧 더 나은 성능"이라는
암묵적 가정에 반례를 제시한다. 즉 아키텍처 분류체계는 필요조건이지만, 각 범주 내에서
"단순성-복잡성" 축을 함께 평가하지 않으면 설계 선택을 오도할 수 있다는 시사점.

## Research Gap
- 표본이 2편뿐이라 일반화하기엔 이르지만, 서베이가 지목한 emerging topics(adaptive
  retrieval, 실시간 통합, multi-hop reasoning, privacy-aware RAG)를 실제로 다루는
  논문이 이 위키에 아직 없다 - 다음 `/paper-collect`에서 우선 탐색할 만한 방향.
- "단순 베이스라인이 복잡한 파이프라인을 능가한다"는 결과가 long-context QA를 넘어
  다른 태스크(멀티홉 추론, 도메인 특화 검색 등)에서도 재현되는지가 이 위키만으로는
  확인되지 않음.

## 주요 비교/개념
- [[rag-architecture-taxonomy]] - 향후 이 팀에 논문이 쌓이면, 각 신규 논문을 4분류 중
  어디에 속하는지 태깅해두면 synthesis가 "어느 범주에 연구가 쏠려 있는지"를 더 정확히
  짚을 수 있음.

## 최근 트렌드가 내 연구에 주는 시사점
`my-research.md`가 비어 있어 이 섹션은 채울 수 없음 - 사용자가 연구 주제/질문을 채운
뒤 재실행 필요.

## 참고
- [[synthesis/latest]]
- [[stronger-baselines-rag-long-context]]
- [[rag-comprehensive-survey-architectures]]
