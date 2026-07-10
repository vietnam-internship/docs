# PRD v1.3 반영 내역 — Reservation Flow (Domain 3, F3)

> 비교: `TravelX_PRD_v1.2_EN.docx` → `TravelX_PRD_v1.3_EN.docx`
> 근거 문서: `reservation/wireFrame-feature3.md`, `be/PRD_Feedback_v2.md`, `be/PRD_Feeback_v2.2.md`(2차 피드백 확정본)

---

## 1. 적용된 변경 — 이미 확정된 결정 반영 (`be/PRD_Feedback_v2.md`/`v2.2.md` 근거)

| # | PRD 위치 | 이전 | 이후 |
|---|---|---|---|
| 1 | §8 FR "Reservation" | Process: "Validate reservation-only stock, lock rate, hold stock for 2 hours" | "Validate reservation-only stock **and time-slot capacity**, lock rate, hold stock for 2 hours" |
| 2 | §12 Branch 테이블 | 시간대 정원 필드 없음 | 신규 필드 `timeSlotCapacity` — "Maximum concurrent reservations allowed per 30-minute pickup time slot at this branch" |
| 3 | §19.1 Reservation Policy 표 | Time-slot 정책 행 없음 | 신규 행 **"Time-slot capacity"** — "Configurable per branch, applied in 30-minute pickup-time windows; a full slot becomes unselectable" |
| 4 | §19 (신규 서브섹션) | — | **§19.3 Time-Slot & Waiting Policy** 신설: "30분 단위 슬롯, 지점별 정원 설정, 초과 시 예약 불가 — 웨이팅/자동승격 없음"을 명문화 |

이는 `PRD_Feeback_v2.2.md` "Waiting Queue Policy" 및 `be/PRD_Feedback_v2.md` #3 "예약 시간 및 Waiting 정책"에서 이미 확정된 내용을 PRD 본문에 옮긴 것 — **새로운 결정이 아니라 기존 결정의 문서화 공백을 메운 것**이다.

## 2. 적용된 변경 — 안전한 보강(safe-additive)

| # | PRD 위치 | 이전 | 이후 |
|---|---|---|---|
| 5 | §8 FR "Cancel Reservation" | Process: "Validate ownership & status, release held stock" | 문구 추가: "…(client must show a confirmation modal before finalizing cancellation)" |
| 6 | §12 Reservation 테이블 | `lockedRate`, `pickedUpAt` 필드 없음 | 신규 필드 2개 추가 (아래 §17.3, MyReservation 도메인과 공유되는 변경) |
| 7 | §17 Sequence Diagrams | 17.1(환율비교), 17.2(예약생성/취소) 두 개뿐 | **§17.3 Cash Pickup / QR Redemption** 신규 — mermaid 소스를 텍스트로 추가 (4-lane: User/Staff/Server/DB). **다이어그램 이미지는 아직 미삽입** — §17.1/17.2 스타일에 맞춰 추후 렌더링 필요 |

## 3. §7 유저플로우 다이어그램 — 이미지 미수정, 텍스트 노트만 추가

`§7.1/7.2/7.3`은 래스터 이미지(PNG)라 이번 작업에서 **그림 자체는 고치지 않았다**. 대신 각 다이어그램 바로 아래에 이탤릭체 "[v1.3 note — diagram not yet redrawn]" 문단을 추가해 그림과 확정 결정 사이의 괴리를 명시했다:

- **§7.1 Admin**: 현재 그림은 "Receive shop registration request → Review documents → Approve or reject"(파트너 셀프신청 모델)인데, 확정된 모델은 **Admin 직접 등록**(`be/PRD_Feedback_v2.md` #1)이다. 노트에 올바른 흐름(Register branch directly → Review preferential rate data → Publish rates → …)을 텍스트로 명시했다.
- **§7.2 Staff**: "Check upcoming visitors"와 "Verify ID and take payment" 사이에 "Scan customer's QR code → look up reservation" 단계 삽입 필요를 명시했다.
- **§7.3 Tourist**: "Show ID for verification" → "Show QR code + ID for verification"로 바뀌어야 함을 명시했다.

**→ 실제 다이어그램 재작도(redraw)는 디자이너/기획자 승인 후 별도 작업으로 남겨둠.**

## 4. 팀 확인이 필요한 사항 (PRD에 반영하지 않음 — 원본 §5 질문 그대로 이월)

1. **통화/금액 입력 시점**: PRD §7 유저플로우 순서(Exchange Comparison 단계에서 입력) vs 와이어프레임 설명("시간 선택 화면에서 통화 선택")이 반대 순서로 보임 — 이번엔 **§7 순서를 변경하지 않았다.** 팀 확인 후 (a) 와이어프레임이 F3 내부에서 통화를 재선택/확정하는 것뿐인지, (b) 실제로 입력 시점 자체가 재설계된 것인지 결정 필요.
2. **지점 선택 단계**: RSV-01(지도+AI추천)에서 핀 클릭 시 바로 시간 선택으로 넘어가는지, 지점 상세 화면(RSV-01-1, 아직 제안 단계)을 거치는지 미확정.
3. **RSV-04("Back to List")**가 PRD §24의 "My Reservations"와 동일 화면인지 확인 필요 (MyReservation 도메인 문서의 화면ID 이슈와 연결됨).
4. **KYC 1단계(휴대폰 인증)** 화면을 별도 화면(`RSV-02-1`)으로 뺄지, 예약 확인 화면 내 인라인 컴포넌트로 넣을지 — PRD §14에 `POST /auth/verify-phone`은 이미 존재하나 화면 배치는 미정.
5. **§21.4 환전소 귀책 미이행 케이스**(방문 시 재고 없음)의 유저 화면을 F3 흐름 안에 포함할지, 별도 알림/플로우로 분리할지 — 미정.
6. **`RSV-` 접두어 충돌**: F2/F3/F4 세 문서가 `RSV-01~04`를 서로 다른 화면에 사용 중 (예: F3의 RSV-02는 "Time & Currency Selection"이지만 F4 원본의 RSV-02는 "Reservation Complete"). PRD 본문은 화면 ID를 사용하지 않아 PRD 자체엔 영향 없으나, **팀 전체 wireframe 화면 ID 체계를 한 번에 정리하는 것을 권장**한다.
