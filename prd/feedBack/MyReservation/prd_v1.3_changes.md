# PRD v1.3 반영 내역 — My Reservations (Domain 4, F4)

> 비교: `TravelX_PRD_v1.2_EN.docx` → `TravelX_PRD_v1.3_EN.docx`
> 근거 문서: `MyReservation/check_data.md`, `MyReservation/prd_update_checklist.md`, `be/PRD_Feedback_v2.md`, `be/PRD_Feeback_v2.2.md`

이 도메인이 이번 v1.3 개정에서 **가장 변경 폭이 크다** — QR 기반 픽업 인증이 F4/F3/Admin 세 도메인에 걸쳐 있고, PRD v1.2에는 그 결정이 거의 반영되어 있지 않았기 때문이다.

---

## 1. 적용된 변경 — 이미 확정된 결정 반영 (QR 관련, `be/PRD_Feedback_v2.md` #2 근거)

| # | PRD 위치 | 이전 | 이후 |
|---|---|---|---|
| 1 | §6.1 Cash Pickup | "Present reservation confirmation" | "**Present QR code (single-use) for staff to scan; QR is invalidated immediately upon redemption**" |
| 2 | §8 FR "Reservation Confirmation" | Process: "Generate reservation number" | "Generate reservation number **and a unique single-use QR code encoding the reservation ID**" |
| 3 | §8 FR (신규 행) | Cash Pickup 행 자체가 없었음 | 신규 행 **"Cash Pickup"** — Input: QR/예약번호+제시된 ID / Process: "Decode → verify status==RESERVED → match ID → atomically transition to COMPLETED (reject reuse)" / Output: 승인 또는 거부(만료/이미사용/불일치) |
| 4 | §17 (신규) | QR 픽업 시퀀스 다이어그램 없음 | **§17.3 Cash Pickup / QR Redemption** — mermaid 텍스트 추가 (4-lane: User/Staff/Server/DB). 렌더링된 이미지는 미삽입, §17.1/17.2 스타일로 추후 추가 필요 |
| 5 | §14 API | QR 상환용 지점 엔드포인트 없음 | 신규 행 `POST /branches/{id}/reservations/{reservationId}/redeem` |
| 6 | §12 Reservation 테이블 | `pickedUpAt` 필드 없음 | 신규 필드 — "Timestamp when staff completed the QR pickup redemption (nullable)" |

## 2. 적용된 변경 — 안전한 보강(safe-additive)

| # | PRD 위치 | 이전 | 이후 |
|---|---|---|---|
| 7 | §6.1 Reservation Management | "View reservations / Cancel reservation / View reservation history" 3줄, My Reservations와 Exchange History 구분 없음 | 4개 bullet로 재구성: "View reservations (Upcoming/Completed/Cancelled, filterable)" / "Cancel reservation (confirmation step required)" / "**View exchange history**(Completed/Cancelled, separate list)" / "**Download receipt**(completed only)" |
| 8 | §8 FR "Cancel Reservation" | Process에 확인모달 요구사항 없음 | "…(client must show a confirmation modal before finalizing cancellation)" 각주 추가 |
| 9 | §8 FR (신규 행) | Download Receipt 기능 자체가 PRD에 없음 | 신규 행 **"Download Receipt"** |
| 10 | §14 API | 예약 단건 조회, 영수증 다운로드 API 없음 | 신규 행 `GET /reservations/{id}`, `GET /reservations/{id}/receipt` |
| 11 | §12 Reservation 테이블 | `lockedRate` 필드 없음 | 신규 필드 — "Exchange rate locked at reservation time (used to compute amount-from/amount-to)" |
| 12 | §12 Branch 테이블 | `locationDetail` 대응 필드 없음 | 신규 필드 `pickupLocationDetail` — "층, 부스 등 상세 위치, 자유 텍스트" |
| 13 | §12 ReservationStatus enum | enum(RESERVED/COMPLETED/CANCELLED) ↔ UI 라벨 매핑 문서화 없음 | 각주 추가: "RESERVED → \"Upcoming\", COMPLETED → \"Completed\", CANCELLED → \"Cancelled\"" |
| 14 | §24 "My Reservations" 행 | "Reservation status (**pending**/complete/cancelled)" | "Reservation status (**Upcoming**/Completed/Cancelled)" — 실제 화면 카피(`check_data.md`)에 맞춰 용어 통일 |
| 15 | §24 "Staff Confirmation" 행 | 성공 경로만 명시 | "; QR-scan action, rejected-state UI (expired / already used / identity mismatch)" 추가 |
| 16 | §24 (신규 3행) | Reservation Complete/Detail/Exchange History 화면 행 없음 | 신규 행 **Reservation Complete**, **Reservation Detail**, **Exchange History** 추가 |

### §2.11/§24 "pending" → "Upcoming" 용어 통일에 대한 판단 근거
이 항목은 원래 원본 체크리스트의 §5 "확인 필요 질문"(4번)이었다. 하지만 세 표기(`RESERVED`/pending/Upcoming) 중 **실제로 완성된 와이어프레임 화면 카피**(`check_data.md`)가 "Upcoming"을 쓰고 있어, PRD 내부 문서 간 용어 불일치를 실제 디자인 산출물에 맞춰 정정하는 것으로 판단해 적용했다. 팀이 다른 라벨을 최종으로 원하면 §24/§12 각주를 재조정하면 된다.

## 3. 팀 확인이 필요한 사항 (PRD에 반영하지 않음 — 원본 §5 질문 그대로 이월)

1. **MYRSV-01/02/03**이 `check_data.md` 원본의 RSV-01(My Reservations)/RSV-04(Exchange History)/RSV-03(Reservation Detail)에 각각 대응하는지 미확정.
2. **Reservation Complete(RSV-02), QR 화면(RSV-03-1), 취소 확인/완료(RSV-03-2/3)**가 F3(Reservation Flow) 도메인으로 넘어간 것인지, F4에 별도 화면(MYRSV-04 등)으로 남는지 미확정.
3. **F4가 최종적으로 `MYRSV-` 접두어로 전면 통일되는지** — F2/F3와의 `RSV-` 접두어 충돌 문제와 함께, 팀 전체 wireframe 화면 ID 체계 정리가 필요하다 (Reservation Flow 도메인 문서 §4의 6번 항목과 동일 이슈).
4. **countdown-to-expiry 요소**가 실제 디자인(MYRSV-01 또는 03)에 존재하는지 — §24 "My Reservations" 행의 "countdown to expiry" 문구는 **v1.2 원문 그대로 유지**했다(삭제도 추가도 하지 않음). 디자이너 확인 후 존재하지 않으면 문구 삭제 필요.
5. §17.3 QR Redemption 다이어그램의 **렌더링된 이미지가 아직 없다** — mermaid 텍스트 소스만 반영. §17.1/17.2와 동일한 스타일로 이미지를 그려 삽입하는 작업이 남아있음.
