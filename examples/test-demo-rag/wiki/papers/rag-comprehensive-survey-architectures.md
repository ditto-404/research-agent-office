# Retrieval-Augmented Generation: A Comprehensive Survey of Architectures, Enhancements, and Robustness Frontiers (2025)

- 저자: Chaitanya Sharma
- 링크/DOI: https://arxiv.org/abs/2506.00054
- 원본 메타데이터: Research Office data 저장소 `teams/test-demo-rag/raw/rag-comprehensive-survey-architectures.json` (private repo)
- 출처: arxiv, 수집일 2026-08-13 (abstract-only)

## 연구문제
RAG가 LLM의 지식 저장 한계를 보완하는 강력한 패러다임으로 부상했지만, 검색 품질과
파이프라인 견고성을 둘러싼 새로운 문제들이 파편적으로 논의되어 왔다. 이를 통합적으로
정리하는 분류체계(taxonomy)가 필요하다.

## 방법론
RAG 시스템을 retriever-centric, generator-centric, hybrid, robustness-oriented 4개
아키텍처 범주로 분류하고, 검색 최적화·컨텍스트 필터링·디코딩 제어·효율성 개선 각각의
접근을 정리한 뒤 QA 벤치마크 결과를 근거로 비교하는 서베이.

## 핵심 기여
- RAG 아키텍처를 4개 범주로 나누는 분류체계 제시.
- 검색 정밀도 vs 생성 유연성, 효율성 vs 충실도(faithfulness), 모듈성 vs 조정
  (coordination) 사이의 반복되는 트레이드오프를 명시적으로 정리.
- adaptive retrieval, 실시간 통합, multi-hop reasoning, privacy-aware 메커니즘을
  향후 유망 연구 영역으로 제시.

## 한계
- 서베이 논문 특성상 개별 기법에 대한 신규 실험적 기여는 없음.
- 초록 수준 정보만 확보 - 분류체계에 포함된 구체적 논문 목록은 원문 확인이 필요함.

## 관련 개념
- [[rag-architecture-taxonomy]]

## 관련 논문
- [[stronger-baselines-rag-long-context]] - 이 survey의 "효율성 vs 충실도" 트레이드오프
  논의는 stronger-baselines 논문이 실증한 "단순 재현율 기반 DOS RAG가 복잡한 다단계
  기법을 능가"하는 결과의 이론적 배경을 제공.
