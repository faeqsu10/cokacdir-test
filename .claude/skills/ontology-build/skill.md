---
name: ontology-build
description: "재난 온톨로지 설계/구현, TypeDB 스키마 작성, 예측 모델 개발, LLM 연동 구현을 수행하는 스킬. TypeDB tql, Python 코드, 온톨로지 스키마, 예측 모델, SPARQL 쿼리, GraphRAG 구현 시 사용. '온톨로지 만들어줘', '스키마 작성', '모델 구현', 'TypeDB', '예측 모델', 'LLM 연동', 'tql' 등의 요청에 트리거."
---

# Ontology Build Skill

재난 온톨로지 구현과 예측 모델 개발을 수행하는 빌드 스킬.

## 프로젝트 구조

```
cokacdir-test/
├── src/
│   ├── ontology/          # TypeDB 스키마 및 쿼리
│   │   ├── schema.tql     # 재난 온톨로지 스키마
│   │   ├── rules.tql      # 추론 규칙
│   │   ├── data.tql       # 초기 데이터 삽입
│   │   └── queries.tql    # 주요 쿼리 모음
│   ├── models/            # 예측 모델
│   │   ├── data_loader.py # 데이터 수집/전처리
│   │   ├── predictor.py   # 위험도 예측 모델
│   │   └── evaluate.py    # 모델 평가
│   └── integration/       # LLM 연동
│       ├── graph_rag.py   # 온톨로지 기반 RAG
│       ├── nl2typeql.py   # 자연어 → TypeQL 변환
│       └── pipeline.py    # 전체 파이프라인
├── data/                  # 수집된 데이터
├── notebooks/             # 실험/시연용 노트북
└── _workspace/            # 팀 중간 산출물
```

## 구현 가이드

### 1. TypeDB 온톨로지 설계

재난 도메인 핵심 엔티티:

```tql
define

# 재난 유형
disaster sub entity,
    owns name,
    owns severity,
    owns occurrence-date,
    plays affects:disaster,
    plays triggers:cause;

# 지역
region sub entity,
    owns name,
    owns population,
    owns area,
    plays affects:region,
    plays has-risk:region;

# 위험도 평가
risk-assessment sub entity,
    owns risk-level,
    owns confidence,
    owns assessed-date,
    plays has-risk:assessment;
```

사용자의 TypeDB 경험 참고: `/home/faeqsu10/projects/typedb/` 디렉토리의 기존 스키마 패턴을 활용한다.

### 2. 예측 모델 구현

재난 위험도 예측 파이프라인:
1. 데이터 수집 (기상, 재난 이력, 지역 특성)
2. 특성 엔지니어링 (온톨로지 기반 특성 추출)
3. 모델 학습 (XGBoost 또는 RandomForest 우선)
4. 평가 (정확도, 재현율, F1)

### 3. LLM 연동

온톨로지 + LLM 연동 패턴:
- **GraphRAG:** TypeDB 쿼리 결과를 LLM 컨텍스트로 제공
- **NL2TypeQL:** 자연어 질문 → TypeQL 쿼리 변환
- **추론 체인:** 온톨로지 추론 + LLM 판단 결합

## 주차별 구현 목표

상세 구현 가이드는 `references/implementation-guide.md`를 참조한다.
