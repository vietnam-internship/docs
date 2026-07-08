# PRD 기획 문서 (Planning Docs)

멘토 피드백 반영을 위한 기획 초안 모음입니다. 각 문서는 담당자별로 작성되었으며, 순서대로 읽으면 의사결정 흐름을 따라갈 수 있습니다.

| # | 파일 | 제목 | 담당 | 상태 |
|---|---|---|---|---|
| 1 | [goal-kpi.md](./goal-kpi.md) | Goal & KPI | A | 초안 |
| 2 | [scope.md](./scope.md) | Scope 선정 | A | 초안 |
| 3 | [persona.md](./persona.md) | Persona | B | 초안 |
| 4 | [competitor-research.md](./competitor-research.md) | Reference 조사 (경쟁 서비스) | A+B | 조사 완료 |
| 5 | [legal-research.md](./legal-research.md) | 법률 및 규제 조사 | A | 조사 완료 |
| 6 | [reservation-policy.md](./reservation-policy.md) | 예약정책 | A | 초안 |
| 7 | [fx-data-model.md](./fx-data-model.md) | 환율 데이터 모델 | A | 초안 |
| 8 | [ai-scope.md](./ai-scope.md) | AI 기능 범위 | B | 초안 |
| 9 | [user-flow.md](./user-flow.md) | User Flow | B | 초안 |
| 10 | [wireframe-requirements.md](./wireframe-requirements.md) | Wireframe 요구사항 | B | 초안 |
| 11 | [functional-requirements.md](./functional-requirements.md) | Functional Requirements | A+B | TBD |

## 읽는 순서 (의존관계)
```
goal-kpi → scope ─┬─→ ai-scope
                  └─→ persona → user-flow → wireframe-requirements
scope, persona 확정 후:
reservation-policy, fx-data-model (legal-research 근거) → functional-requirements
```

## 핵심 크로스체크 항목
- `competitor-research.md`의 마이뱅크 이슈 → `goal-kpi.md`의 핵심가치 문구에 "예약" 강조 반영 여부
- `persona.md`의 신뢰 Pain Point → `reservation-policy.md`의 신뢰 보완 장치 반영 여부
- `user-flow.md`의 "2시간 자동취소" ↔ `reservation-policy.md` 수치 일치 여부
- `scope.md`의 AI 보조기능 명시 ↔ `ai-scope.md` KPI 방향 일치 여부
