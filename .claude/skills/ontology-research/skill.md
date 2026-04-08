---
name: ontology-research
description: "재난 온톨로지, 오픈 온톨로지, 재난 데이터, 예측 모델 관련 리서치를 수행하는 스킬. 온톨로지 개념 조사, 사례 분석, 데이터 소스 탐색, 논문/기술 동향 조사 시 사용. '조사해줘', '자료 찾아줘', '사례 알려줘', '데이터 소스', '온톨로지란', '재난 데이터' 등의 요청에 트리거."
---

# Ontology Research Skill

재난 온톨로지 스터디의 리서치 작업을 체계적으로 수행하는 스킬.

## 조사 프레임워크

### 1. 온톨로지 기초 조사
- OWL, RDF, RDFS, SPARQL 기본 개념과 관계
- TypeDB와 전통적 온톨로지(OWL)의 차이점
- 온톨로지 설계 방법론 (상향식/하향식/혼합)

### 2. 오픈 온톨로지 사례 조사

**민간 분야:**
- Schema.org, DBpedia, Wikidata
- FOAF, Dublin Core, SKOS
- 산업별 도메인 온톨로지

**공공 분야:**
- 한국 공공데이터 온톨로지 (data.go.kr)
- 국가 표준 온톨로지 (KISTI, NIA)
- 해외: US NIEM, EU ISA

**재난 분야:**
- MOAC (Management of a Crisis)
- HXL (Humanitarian Exchange Language)
- 행안부 재난안전 데이터 표준
- 기상청 기상 온톨로지

### 3. 재난 데이터 소스

| 소스 | URL | 데이터 유형 |
|------|-----|-----------|
| 공공데이터포털 | data.go.kr | 재난 통계, 이력 |
| 기상청 API | data.kma.go.kr | 기상 관측, 예보 |
| 행안부 | mois.go.kr | 재난 대응 이력 |
| 소방청 | nfa.go.kr | 화재/구조 데이터 |
| 국토부 | molit.go.kr | 지질/토양 데이터 |

### 4. 온톨로지 + LLM 결합 동향
- Knowledge Graph + LLM (GraphRAG)
- 온톨로지 기반 프롬프트 엔지니어링
- LLM을 활용한 자동 온톨로지 생성
- 재난 분야 AI 예측 모델 동향

## 출력 형식

리서치 문서는 다음 구조로 작성한다:

```markdown
# {주제}

## 개요
{1-2문장 요약}

## 핵심 내용
{구조화된 조사 결과}

## 사례/데이터
{구체적 사례, 데이터 소스, 코드 예시}

## 참고 자료
- {출처 URL 또는 논문명}

## 스터디 활용 포인트
{이 자료를 스터디 발표에 어떻게 활용할 수 있는지}
```

## 주차별 조사 가이드

상세 주차별 조사 항목은 `references/weekly-guide.md`를 참조한다.
