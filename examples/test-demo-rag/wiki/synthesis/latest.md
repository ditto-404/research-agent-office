# Research Synthesis - Test Demo - RAG (2026-08-13)

## 이번 갱신 요약
최초 synthesis. 논문 2편(실증 연구 1편, 서베이 1편)을 근거로 작성. 표본이 매우 작아
(2편) 아래 "동향"은 잠정적이며, 추가 수집 후 재작성이 필요함.

## 최근 연구 동향
- 장문맥(long-context) LLM이 강력해지면서 "복잡한 다단계 RAG가 여전히 필요한가"라는
  질문이 제기되고 있으며, 최소 이 표본에서는 단순한 접근(문서 구조 유지 + 재현율 우선)이
  더 복잡한 파이프라인(ReadAgent, RAPTOR)을 능가하는 사례가 보고됨
  [stronger-baselines-rag-long-context].
- RAG 설계 공간을 정리하려는 시도가 진행 중이며, 검색기 중심/생성기 중심/하이브리드/
  견고성 중심의 4분류 프레임이 제안됨 [rag-comprehensive-survey-architectures].

## 핵심 개념 지도
- [[rag-architecture-taxonomy]] - RAG 시스템을 설계 초점(검색기/생성기/하이브리드/
  견고성)에 따라 분류하는 틀. 이 팀 위키의 유일한 핵심 개념이자, 두 논문을 잇는 축.

## 주요 방법론
- 벤치마크 기반 비교 평가 (여러 RAG 파이프라인을 동일 QA 벤치마크·토큰 예산에서
  직접 비교) [stronger-baselines-rag-long-context]
- 문헌 분류/서베이 방법론 (기존 RAG 연구를 아키텍처 축으로 재분류)
  [rag-comprehensive-survey-architectures]

## Emerging Topics
- adaptive retrieval, 실시간 지식 통합, multi-hop reasoning, privacy-aware RAG -
  서베이 논문이 향후 유망 영역으로 지목 [rag-comprehensive-survey-architectures].
  아직 이 위키에 해당 주제를 직접 다루는 논문은 없음(향후 수집 대상).

## 다루는 논문
- [[stronger-baselines-rag-long-context]] (2025)
- [[rag-comprehensive-survey-architectures]] (2025)
