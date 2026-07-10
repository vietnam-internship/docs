# PRD v1.3 반영 내역 — Admin (Domain 6)

> 비교: `TravelX_PRD_v1.2_EN.docx` → `TravelX_PRD_v1.3_EN.docx`
> 근거 문서: `be/PRD_Feedback_v2.md`, `be/PRD_Feeback_v2.2.md`, `prd/planning/v1.2-feedback-review.md`
>
> 다른 4개 도메인과 달리 Admin 도메인은 전용 wireframe 피드백 문서가 없었다. 하지만 `be/PRD_Feedback_v2.md`(2차 피드백 확정본)의 "Branch Management" 결정이 PRD v1.2 본문에 전혀 반영되어 있지 않아, 이번에 별도 도메인 문서로 신설했다.

---

## 1. 적용된 변경 — 이미 확정된 결정 반영

| # | PRD 위치 | 이전 | 이후 |
|---|---|---|---|
| 1 | §6.2 Admin Features | "Manage exchange offices (branches)" 한 줄 — 등록 방식 미정의 | "Register new branch (name, address, map coordinates, business hours, contact, supported currencies)" / "Edit / deactivate branch" / "**Set per-branch time-slot reservation capacity (30-minute slots)**" 3개 bullet로 확장 |
| 2 | §8 FR "Branch Management (Admin)" | Input: "Branch details" | "Branch details (name, address, coordinates, hours, contact, supported currencies, **time-slot capacity**)" |
| 3 | §14 API Specification | 지점 신규 등록 API 없음 (`PATCH /branches/{id}/rate`만 존재) | 신규 행 `POST /admin/branches`, `PATCH /admin/branches/{id}` |
| 4 | §12 Branch 테이블 | 시간대 정원 필드 없음 | 신규 필드 `timeSlotCapacity` |
| 5 | §16 Future Features | 파트너 셀프온보딩 언급 없음 | 신규 bullet: "**Partner self-service branch onboarding** (application, document review, approval workflow)" |
| 6 | §19.1 Reservation Policy 표 | 최소 환전 금액 정책 없음 | 신규 행 **"Minimum exchange amount"** — "10,000 VND (equivalent) — reservations below this cannot be completed" |

## 2. §16에 "파트너 셀프온보딩"을 추가한 이유 — 기존 포크(fork) 해소

`prd/planning/v1.2-feedback-review.md` §1이 지적한 문제: `user-flow.md`(Admin Flow)는 "환전소 등록 신청 접수 → 서류 확인 → 승인/반려"(파트너 셀프신청 모델, Option B)를 전제하는데, `be/PRD_Feedback_v2.md` #1의 실제 확정 결정은 **Admin이 콘솔에서 직접 등록**(Option A)이다. 두 모델이 서로 다른 문서에 암묵적으로 혼재해 있었다.

이번 v1.3은 **Option A를 MVP 기능으로 §6.2/§8/§14에 반영**하고, Option B(파트너 신청/승인)는 **§16 Future Features로 명시적으로 이동**시켜 포크를 해소했다. `v1.2-feedback-review.md`가 권장한 방향과 일치한다.

**§7.1 Admin 유저플로우 다이어그램(래스터 이미지)은 여전히 파트너 신청 모델을 그리고 있다** — 이미지 자체는 수정하지 않았고, 대신 다이어그램 아래에 "[v1.3 note — diagram not yet redrawn]" 문단으로 올바른 흐름(Register branch directly → Review preferential rate data → Publish rates → …)을 텍스트로 명시해 두었다. 실제 다이어그램 재작도는 디자이너 승인 후 별도 작업.

## 3. 팀 확인이 필요한 사항

1. **최소 환전 금액 "10,000 VND"** — `be/PRD_Feedback_v2.md` #3에 명시된 확정값을 그대로 옮겼으나, 미화로 환산 시 1달러 미만으로 매우 작은 금액이다. 오탈자(자릿수 누락)가 아닌지, 또는 의도적으로 낮은 진입장벽을 위한 값인지 팀 재확인 권장.
2. **§16 "Partner self-service branch onboarding"**을 향후에라도 실제 지원할 계획인지, 아니면 "MVP 범위 밖"이라는 사실만 명시하고 실제로는 지원하지 않을 것인지 — 로드맵 우선순위 논의 필요.
3. **§7.1 다이어그램 재작도** — Admin/Reservation/MyReservation 세 도메인에 걸쳐 총 3개 그림(§7.1, §7.2, §7.3)과 1개 신규 시퀀스(§17.3)가 텍스트로만 반영되고 이미지가 없는 상태다. 디자이너 리소스 배정 필요.
