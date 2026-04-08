---
name: disaster-ontology-orchestrator
description: "재난 온톨로지 스터디 에이전트팀을 조율하는 오케스트레이터. 스터디 준비, 주차별 작업 실행, 리서치-구현-리뷰 파이프라인 운영. '스터디 준비해줘', '이번 주차 작업 시작', '팀 돌려줘', '하네스 실행', '온톨로지 팀', '주차 작업' 등의 요청에 트리거."
---

# Disaster Ontology Study Orchestrator

재난 온톨로지 스터디를 위한 에이전트팀(researcher → builder → reviewer)을 조율하는 통합 스킬.

## 실행 모드: 에이전트 팀

## 에이전트 구성

| 팀원 | 에이전트 파일 | 역할 | 스킬 | 출력 |
|------|-------------|------|------|------|
| researcher | agents/researcher.md | 자료 조사 | ontology-research | `_workspace/{week}_researcher_*.md` |
| builder | agents/builder.md | 온톨로지/모델 구현 | ontology-build | `src/` 디렉토리 코드 |
| reviewer | agents/reviewer.md | 코드 리뷰/검증 | ontology-review | `_workspace/{week}_reviewer_review.md` |

## 워크플로우

### Phase 1: 준비
1. 사용자 입력에서 **대상 주차** 파악 (1~5회차)
2. `_workspace/` 디렉토리 존재 확인, 없으면 생성
3. 이전 주차 산출물이 있으면 참조 경로 확인

### Phase 2: 팀 구성

```
TeamCreate(
  team_name: "disaster-ontology-team",
  members: [
    {
      name: "researcher",
      agent_type: "general-purpose",
      model: "opus",
      prompt: "당신은 재난 온톨로지 리서치 에이전트입니다.
        에이전트 정의: .claude/agents/researcher.md를 읽고 역할을 파악하세요.
        스킬: .claude/skills/ontology-research/skill.md를 읽고 작업 방식을 따르세요.
        현재 주차: {week}회차
        작업 디렉토리: _workspace/
        이전 산출물: {previous_artifacts}"
    },
    {
      name: "builder",
      agent_type: "general-purpose",
      model: "opus",
      prompt: "당신은 온톨로지 구현 에이전트입니다.
        에이전트 정의: .claude/agents/builder.md를 읽고 역할을 파악하세요.
        스킬: .claude/skills/ontology-build/skill.md를 읽고 작업 방식을 따르세요.
        현재 주차: {week}회차
        TypeDB 참고: /home/faeqsu10/projects/typedb/
        researcher의 조사 결과를 기다린 후 구현을 시작하세요."
    },
    {
      name: "reviewer",
      agent_type: "general-purpose",
      model: "opus",
      prompt: "당신은 코드 리뷰/검증 에이전트입니다.
        에이전트 정의: .claude/agents/reviewer.md를 읽고 역할을 파악하세요.
        스킬: .claude/skills/ontology-review/skill.md를 읽고 작업 방식을 따르세요.
        현재 주차: {week}회차
        builder의 구현이 완료될 때마다 증분 리뷰를 수행하세요.
        Codex CLI를 활용하여 독립적 리뷰를 병행하세요."
    }
  ]
)
```

작업 등록 (주차별로 다르게 구성):

```
TaskCreate(tasks: [
  { title: "{week}회차 자료 조사", assignee: "researcher" },
  { title: "{week}회차 구현", assignee: "builder", depends_on: ["자료 조사"] },
  { title: "{week}회차 리뷰", assignee: "reviewer", depends_on: ["구현"] },
  { title: "발표 자료 정리", assignee: "researcher", depends_on: ["리뷰"] }
])
```

### Phase 3: 실행

**파이프라인 흐름:**

```
researcher: 자료 조사
    ↓ SendMessage(to: builder, "조사 완료. _workspace/{week}_researcher_*.md 참조")
builder: 온톨로지/모델 구현
    ↓ SendMessage(to: reviewer, "구현 완료. src/ 리뷰 요청")
reviewer: 코드 리뷰 + Codex 리뷰
    ↓ SendMessage(to: builder, "리뷰 결과. 수정 필요 사항: ...")
builder: 수정 반영 (필요 시)
    ↓ SendMessage(to: leader, "수정 완료")
```

**팀원 간 통신 규칙:**
- researcher → builder: 조사 완료 시 핵심 발견사항 + 파일 경로
- builder → reviewer: 모듈 완성 시 리뷰 요청 (증분)
- reviewer → builder: 리뷰 결과 + 구체적 수정 코드
- 모든 팀원 → 리더: 진행률 업데이트

**산출물 저장:**

| 팀원 | 출력 경로 |
|------|----------|
| researcher | `_workspace/{week}_researcher_{topic}.md` |
| builder | `src/ontology/`, `src/models/`, `src/integration/` |
| reviewer | `_workspace/{week}_reviewer_review.md` |

### Phase 4: 통합
1. 리더가 모든 산출물 확인
2. 주차별 종합 요약 생성: `_workspace/{week}_summary.md`
3. 발표 자료 핵심 포인트 정리

### Phase 5: 정리
1. 팀원들에게 종료 요청: `SendMessage({to: name, message: {type: "shutdown_request"}})`
2. 팀 정리
3. `_workspace/` 보존

## 주차별 작업 매핑

| 주차 | researcher | builder | reviewer |
|------|-----------|---------|----------|
| 1회차 | 온톨로지 기본 개념 정리 | (대기) | (대기) |
| 2회차 | 오픈 온톨로지 사례 조사 | (대기) | (대기) |
| 3회차 | 재난 데이터/설계방법론 조사 | 데이터 수집 코드 + 스키마 초안 | 코드 리뷰 |
| 4회차 | 추가 자료 보충 | 온톨로지 구현 + 예측 모델 | 스키마 + 모델 리뷰 |
| 5회차 | LLM 연동 사례 보충 | LLM 연동 + 전체 파이프라인 | 통합 리뷰 |

## 에러 핸들링

| 에러 유형 | 대응 |
|----------|------|
| researcher 조사 실패 | 대체 검색어로 재시도, 불가 시 누락 명시 |
| builder 구현 오류 | 에러 로그 분석 후 수정, 재시도 |
| reviewer Codex 호출 실패 | 자체 리뷰로 대체 |
| 팀원 무응답 | 30초 대기 후 리더가 직접 개입 |
| 상충 정보 | 양쪽 모두 보존, 출처 병기 |

## 테스트 시나리오

### 정상 흐름
```
사용자: "3회차 작업 시작해줘"
→ 팀 구성 → researcher 재난 데이터 조사
→ builder 데이터 수집 코드 + 스키마 초안 구현
→ reviewer 코드 리뷰 → 수정 → 종합 요약
```

### 에러 흐름
```
사용자: "4회차 작업 시작해줘" (3회차 미완료 상태)
→ 이전 주차 산출물 확인 → 부족 시 researcher에게 보충 조사 우선 지시
→ 보충 완료 후 정상 흐름 진행
```
