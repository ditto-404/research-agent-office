# RAG 아키텍처 분류 (RAG Architecture Taxonomy)

## 정의
Retrieval-Augmented Generation(RAG) 시스템을 검색기 중심(retriever-centric), 생성기
중심(generator-centric), 하이브리드(hybrid), 견고성 중심(robustness-oriented)의 네
범주로 나누어 설계 공간을 정리하는 분류 체계.

## 보편적 설명
RAG는 LLM이 파라미터에 저장하지 못하는 최신/외부 지식을 추론 시점에 검색해 생성에
반영하는 접근이다. 시스템마다 검색기 개선에 집중하는지, 생성기의 컨텍스트 활용 방식에
집중하는지, 아니면 둘을 결합하거나 파이프라인의 견고성(노이즈·모순되는 근거에 대한
강건함)에 집중하는지가 갈리며, 이 분류는 그런 설계 선택들을 한 틀 안에서 비교하기 위한
장치다.

## 왜 중요한가 / 왜 등장했는가
RAG 연구가 빠르게 늘면서 서로 다른 논문이 서로 다른 축(검색 정밀도, 생성 유연성, 효율성,
충실도, 모듈성)을 기준으로 기여를 주장해 비교가 어려워졌다. 공통 분류체계가 있어야
"이 방법이 정말 더 복잡한 접근을 능가하는가"(예: DOS RAG vs ReadAgent/RAPTOR) 같은
질문에 아키텍처 범주를 맞춰 공정하게 비교할 수 있다.

## 논문별 등장 방식
- [[rag-comprehensive-survey-architectures]]: 이 분류체계를 직접 제안한 논문. 4개
  범주와 함께 검색 정밀도-생성 유연성, 효율성-충실도, 모듈성-조정 사이의 트레이드오프를
  정리.
- [[stronger-baselines-rag-long-context]]: 이 분류체계를 명시적으로 쓰진 않지만, 단순
  재현율 기반 접근(DOS RAG)이 복잡한 다단계 파이프라인(ReadAgent, RAPTOR)을 능가함을
  보여 "복잡성 대비 단순성" 트레이드오프의 실증 사례로 읽을 수 있음.

## 대표 방법론/사례
- DOS RAG(단순 재현율 우선, 문서 구조 유지) - 단순 베이스라인이 강력함을 보여준 사례 →
  [[stronger-baselines-rag-long-context]]

## 관련 논문
- 위 "논문별 등장 방식" 목록 참고.
