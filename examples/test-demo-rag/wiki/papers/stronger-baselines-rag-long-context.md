# Stronger Baselines for Retrieval-Augmented Generation with Long-Context Language Models (2025)

- 저자: Alex Laitenberger, Christopher D. Manning, Nelson F. Liu
- 링크/DOI: https://arxiv.org/abs/2506.03989 (EMNLP 2025)
- 원본 메타데이터: Research Office data 저장소 `teams/test-demo-rag/raw/stronger-baselines-rag-long-context.json` (private repo)
- 출처: arxiv, 수집일 2026-08-13 (abstract-only)

## 연구문제
장문맥(long-context) LLM이 점점 강력해지면서, ReadAgent나 RAPTOR 같은 정교한 다단계
RAG 파이프라인이 여전히 단순한 RAG 대비 이점을 갖는지가 불명확해졌다. 이 논문은 복잡한
파이프라인과 단순한 베이스라인을 동일한 조건에서 직접 비교해 이 질문에 답한다.

## 방법론
ReadAgent, RAPTOR 같은 다단계 RAG 기법과, 문서 구조를 유지하며 토큰 예산 내 재현율을
최대화하는 단순한 베이스라인 "DOS RAG"를 여러 장문맥 QA 벤치마크에서 토큰 예산을
체계적으로 바꿔가며 비교 평가한다.

## 핵심 기여
- DOS RAG가 여러 장문맥 QA 벤치마크에서 더 복잡한 다단계 기법들과 동등하거나 그 이상의
  성능을 보임.
- 이 강건함은 문서 구조 유지, 유효 컨텍스트 윈도우 내 재현율 우선, 파이프라인 복잡도
  최소화에서 기인한다고 분석.
- DOS RAG를 향후 RAG 연구의 표준 베이스라인으로 제안.

## 한계
- 초록(abstract) 수준의 정보만 확보한 상태 - 실험에 사용된 구체적 벤치마크 목록,
  토큰 예산 범위, 통계적 유의성 등은 원문 확인이 필요함 (source_type: abstract-only).

## 관련 개념
- [[rag-architecture-taxonomy]]

## 관련 논문
- [[rag-comprehensive-survey-architectures]] - 이 논문이 지적하는 "복잡성 대비 단순성"
  문제의식은 survey 논문이 정리한 retriever-centric vs generator-centric 아키텍처
  분류와 맞닿아 있음(단순 재현율 기반 접근이 어느 카테고리에 속하는지 비교할 수 있음).
