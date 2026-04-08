# 주차별 구현 가이드

## 3회차: 데이터 수집 + 온톨로지 설계

### 데이터 수집 구현
```python
# src/models/data_loader.py
import requests
import pandas as pd

class DisasterDataLoader:
    """공공데이터포털, 기상청 API에서 재난 데이터를 수집한다."""
    
    def fetch_weather(self, date, region):
        """기상청 API에서 기상 데이터를 가져온다."""
        pass
    
    def fetch_disaster_history(self, year_range):
        """행안부 재난 이력 데이터를 가져온다."""
        pass
    
    def preprocess(self, raw_data):
        """원시 데이터를 모델 학습용으로 전처리한다."""
        pass
```

### LLM 기반 온톨로지 초안 생성
- 재난 관련 문서에서 엔티티/관계 자동 추출
- Claude API로 도메인 전문가 수준의 스키마 제안
- 사용자 검토 후 TypeDB 스키마로 변환

---

## 4회차: 온톨로지 구현 + 예측 모델

### TypeDB 환경 구성
```bash
# TypeDB 서버 시작 (이미 설치된 경우)
typedb server

# 데이터베이스 생성
typedb console
> database create disaster-ontology
> transaction disaster-ontology schema write
> source src/ontology/schema.tql
> commit
```

### 추론 규칙 구현
```tql
define

# 고위험 지역 추론 규칙
rule high-risk-region:
    when {
        $r isa region, has population $p;
        $p > 100000;
        $d isa disaster, has severity "high";
        (disaster: $d, region: $r) isa affects;
    } then {
        (region: $r, assessment: $a) isa has-risk;
    };
```

### 예측 모델 학습
```python
# src/models/predictor.py
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.model_selection import cross_val_score

class DisasterRiskPredictor:
    """재난 위험도를 예측하는 모델."""
    
    def __init__(self, model_type="rf"):
        if model_type == "rf":
            self.model = RandomForestClassifier(n_estimators=100)
        elif model_type == "xgb":
            self.model = GradientBoostingClassifier(n_estimators=100)
    
    def train(self, X, y):
        """모델을 학습한다."""
        scores = cross_val_score(self.model, X, y, cv=5, scoring='f1_weighted')
        self.model.fit(X, y)
        return {"f1_mean": scores.mean(), "f1_std": scores.std()}
    
    def predict(self, X):
        """위험도를 예측한다."""
        return self.model.predict_proba(X)
```

---

## 5회차: LLM 연동 + 통합

### GraphRAG 구현
```python
# src/integration/graph_rag.py
from typedb.driver import TypeDB, SessionType, TransactionType

class DisasterGraphRAG:
    """TypeDB 온톨로지 기반 RAG 시스템."""
    
    def query_context(self, question):
        """자연어 질문에서 관련 온톨로지 컨텍스트를 추출한다."""
        typeql = self.nl2typeql(question)
        results = self.execute_query(typeql)
        return self.format_context(results)
    
    def nl2typeql(self, question):
        """자연어를 TypeQL로 변환한다 (LLM 활용)."""
        pass
    
    def reason_with_llm(self, question, context):
        """온톨로지 컨텍스트 + LLM으로 추론한다."""
        pass
```

### 전체 파이프라인
```
[사용자 질문] → [NL2TypeQL] → [TypeDB 쿼리] → [온톨로지 추론]
                                                      ↓
[최종 답변] ← [LLM 추론] ← [예측 모델 결과] ← [컨텍스트 구성]
```
