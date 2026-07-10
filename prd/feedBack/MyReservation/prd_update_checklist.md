# PRD v1.2 수정 체크리스트 — My Reservations 기준

> 비교 대상: `check_data.md`(My Reservations 화면 설계, RSV-01~RSV-04) vs `TravelX_PRD_v1.2_EN.pdf`(전 28페이지)
> 참고: `prd/feedBack/PRD_Feedback_v2.md`, `PRD_Feeback_v2.2.md`(이미 합의된 2차 피드백 반영 사항 — QR/Time Slot 관련 결정은 여기서 이미 확정됨. 이 문서는 그 결정이 PRD 본문에는 아직 반영 안 된 부분을 짚는다)
>
> 형식: **[PRD 섹션 (페이지)] — 문제 → 수정 제안**. 체크박스는 각 섹션 원본에 실제로 반영했을 때 체크.


## 1. 지금까지 와이어프레임에서 정리된 내용 (F4: My Reservation)

| 화면 ID | 주요 내용 |
|---|---|---|
| MYRSV-01 | Look up the list of currency exchanges that you have currently booked.
It will be implemented as an infinite scroll. |
| MYRSV-02 | Inquire the list of completed currency exchange transaction details.
There are two states: Completed and Canceled.
It will be implemented as an infinite scroll.
| MYRSV-03 | Check the details of past currency exchange transaction details |

---

## §6.1 User Features (p.5)

- [ ] **Cash Pickup 항목** — 현재 "Present reservation confirmation" 한 줄뿐, QR 언급 없음. `PRD_Feedback_v2.md` #2에서 이미 "이미지 → QR Code, 1회 사용"으로 결정됐는데 본문에 미반영.
  → **추가**: "Present QR code (single-use) for staff to scan; QR is invalidated immediately upon redemption."
- [ ] **Reservation Management 항목** — "View reservations / Cancel reservation / View reservation history" 세 줄이 뭉뚱그려져 있음. `check_data.md`는 이 둘을 **별개 화면**으로 명확히 분리함: RSV-01 "My Reservations"(전체 상태: upcoming/completed/cancelled 탭) vs RSV-04 "Exchange History"(completed/cancelled 전용, 환전 전후 금액 표시, 별도 진입점).
  → **추가**: "View reservations"와 "View exchange history"를 별개 bullet으로 분리하고 각각의 스코프(상태 필터, 표시 필드) 명시.

---

## §7.2 Userflow Staff (p.8)

- [ ] 현재 흐름: `Receive reservation alert → Check upcoming visitors → Verify ID and take payment → Mark exchange complete`. QR 스캔 단계가 없이 바로 "Verify ID"로 넘어감 — QR 결정과 어긋남.
  → **추가**: "Check upcoming visitors"와 "Verify ID and take payment" 사이에 **"Scan customer's QR code → look up reservation"** 노드 삽입.

## §7.3 Userflow Tourist (p.9)

- [ ] 현재 흐름은 예약 생성→픽업까지의 선형 해피패스만 존재("Show ID for verification"까지). `check_data.md`의 RSV-01~04(예약 목록 조회/상세/취소/이력)에 해당하는 흐름이 §7 어디에도 없음.
  → **추가 (택1)**: (a) "Show ID for verification" 노드를 "Show QR code + ID for verification"으로 수정, **그리고** (b) 신규 "7.4 Userflow — My Reservations / Cancellation" 다이어그램을 추가하거나, `check_data.md`가 그 역할을 한다고 §7에 명시적으로 상호 참조 추가.

---

## §8 Functional Requirements (p.10–11)

- [ ] **"Cash Pickup" FR 행 자체가 없음.** §6.1엔 MVP 기능으로 나열되지만 8장 표에는 대응 행이 없다. 가장 가까운 "Identity Verification (KYC)" 행은 신분증 대조만 다루고, QR 디코드·상태검증·1회성 처리·완료전환은 다루지 않음.
  → **추가**: 새 행 `Cash Pickup` — Input: QR코드(or 예약번호)+제시된 ID / Process: QR 디코드 → 예약 status==RESERVED 검증 → ID와 예약자 정보 대조 → 원자적으로 COMPLETED 전환(재사용 거부) / Output: 픽업 승인 또는 거부(만료·이미사용·불일치).
- [ ] **"Reservation Confirmation" 행** — Process가 "Generate reservation number"뿐, QR 발급이 빠짐(QR 결정은 이미 확정됨).
  → **수정**: Process를 "Generate reservation number **and a unique single-use QR code encoding the reservation ID**"로 변경.
- [ ] **"Cancel Reservation" 행** — `check_data.md`는 취소 전 확인 팝업(RSV-03-2-POPUP)을 필수 요구("즉시 취소 금지"가 검증 체크리스트에 명시됨)하지만 FR 표엔 이 UX 조건이 없음.
  → **추가**: Process 설명에 "(클라이언트) 취소 확정 전 확인 모달 필수" 각주 추가.
- [ ] **"Reservation History" 행** — §6.1과 동일하게 "My Reservations"와 "Exchange History"가 한 행에 뭉쳐 있음.
  → **수정**: 두 행으로 분리하거나, Output에 두 스코프(전체 상태 리스트 / completed·cancelled 전용 + 금액상세)를 각각 명시.
- [ ] **"Download Receipt" FR이 아예 없음.** RSV-03의 completed 상태에 "Download receipt" 버튼이 있는데 PRD 어디에도 영수증 생성/다운로드 기능이 없음.
  → **추가**: 새 행 `Download Receipt` — Input: Reservation ID(status==COMPLETED) / Process: 예약+지점+환율 데이터로 영수증 생성 / Output: 다운로드 가능한 영수증 파일.

---

## §12 Database Design (p.15–17)

- [ ] **Reservation 테이블에 lockedRate(또는 krwAmount) 필드 없음.** §8 Reservation 행 Process엔 "lock rate"가 명시되어 있는데 저장할 필드가 없다. `check_data.md`의 데이터 모델(`amountFrom`/`amountTo`, 환전 전후 금액)을 화면에 안정적으로 보여주려면 예약 시점에 잠긴 환율(또는 원화 환산액)을 저장해야 함.
  → **추가 필드**: `Reservation.lockedRate` (or `krwAmount`).
- [ ] **픽업 완료 시각을 기록할 필드 없음.** `status`만으로는 언제 COMPLETED로 바뀌었는지 감사(audit) 불가.
  → **추가 필드**: `Reservation.pickedUpAt` (nullable timestamp).
- [ ] **Branch 테이블에 층/부스 등 상세 위치 필드 없음.** `check_data.md` 데이터 모델의 `locationDetail`(상세 위치)에 대응하는 필드가 Branch(id/name/address/phone/latitude/longitude/businessHours)에 없음.
  → **추가 필드**: Branch(or Reservation)에 `pickupLocationDetail` 자유 텍스트 필드.
- [ ] **ReservationStatus enum 표기가 세 문서에서 제각각.** DB enum(`RESERVED`/`COMPLETED`/`CANCELLED`, p.16) vs §24 Wireframe 표(p.27, "pending/complete/cancelled") vs 실제 화면 카피(`check_data.md`, "Upcoming/Completed/Cancelled") — 세 곳이 서로 다른 용어를 씀.
  → **추가**: enum→UI 라벨 매핑 각주 명시 (`RESERVED` → "Upcoming" 등), §24의 "pending"을 "Upcoming"으로 정정 (아래 §24 항목과 동일 이슈).

---

## §14 API Specification (p.19)

- [ ] **`GET /reservations/{id}` 없음.** 목록(`GET /reservations`)과 취소(`DELETE /reservations/{id}`)만 있고, RSV-03(예약 상세)이 필요로 하는 단건 조회 API가 없음.
  → **추가**: `GET /reservations/{id}` — 예약 상세(+ QR 페이로드) 조회.
- [ ] **영수증 다운로드 엔드포인트 없음.**
  → **추가**: `GET /reservations/{id}/receipt` — 완료된 예약의 영수증 다운로드.
- [ ] **QR 스캔/픽업 확정용 지점(Staff) 엔드포인트 없음.** `/admin/reservations`(전체 조회), `PATCH /branches/{id}/rate`(우대율/재고 갱신)만 있고, 직원이 QR을 스캔해 픽업을 확정 처리하는 API가 없음.
  → **추가**: 예) `POST /branches/{id}/reservations/{reservationId}/redeem` — QR 스캔 결과로 상태검증 후 COMPLETED 전환.

---

## §17 Sequence Diagrams (p.22)

- [ ] 현재 17.1(환율비교), 17.2(예약 생성+취소) 두 개뿐. `check_data.md`가 가장 상세히 다루는 **QR 픽업/상환(redemption) 흐름**에 대한 시퀀스 다이어그램이 없음.
  → **추가**: "17.3 Cash Pickup / QR Redemption" — User가 QR 제시 → Staff 앱이 스캔 → Server가 디코드+상태검증+ID대조 → DB status RESERVED→COMPLETED → 승인/거부 응답.

---

## §19.1 Reservation Policy (p.24) — 참고(부차적)

- [ ] `PRD_Feedback_v2.md` #3 / `PRD_Feeback_v2.2.md`(Waiting Queue Policy)에서 이미 "30분 단위 Time Slot + 지점별 최대 예약 인원, 초과 시 예약 불가"로 확정됐는데 §19나 §12 어디에도 반영 안 됨. (참고: `check_data.md`의 My Reservations 화면 자체엔 슬롯 선택 UI가 없는 게 맞음 — 슬롯 선택은 예약 생성 위저드 쪽 화면이라 `wireFrameDomain.md` 기준으로도 별도 도메인. 다만 데이터 모델/정책 문서엔 반영이 필요.)
  → **추가**: §19에 Time-slot capacity 행 추가, §12 Branch에 슬롯당 최대인원 필드 추가(§6.2 Admin 등록 폼 필드와 동일 값).

---

## §24 Wireframe Requirements (p.27)

- [ ] **"My Reservations" 행의 "pending/complete/cancelled"** — 실제 화면 카피("Upcoming/Completed/Cancelled")와 불일치.
  → **수정**: "pending" → "Upcoming".
- [ ] **"countdown to expiry"가 요구사항에 있는데 `check_data.md`의 RSV-01/RSV-03 어디에도 카운트다운 UI가 없음.** 디자인 누락인지, 요구사항이 stale한 건지 확인 필요.
  → **확인 필요(결정 보류)**: 디자이너와 확인 후 (a) RSV-03에 카운트다운 요소 추가하거나 (b) §24에서 해당 요구사항 삭제.
- [ ] **행 자체가 누락됨** — 표에는 Home/Exchange Comparison/Reservation/AI Recommendation/My Reservations/Staff Confirmation 6개 화면만 있고, `check_data.md`가 정의하는 아래 3개 화면에 대응하는 행이 없음.
  → **추가 행 3개**:
    - `Reservation Complete` (RSV-02): 체크 아이콘, 예약번호, Amount/Location/Date 요약, "View reservation"/"Back to home" 버튼
    - `Reservation Detail` (RSV-03): Pickup location/time, 통화쌍, 예약번호, 상태별 분기(QR 보기/취소/영수증 다운로드/취소됨 안내)
    - `Exchange History` (RSV-04): Completed/Cancelled 탭, 환전 전→후 금액 리스트
- [ ] **"Staff Confirmation (branch)" 행** — "Reservation-holder info, ID-check button, payment-complete button"만 있고 QR 스캔 요소, 실패 상태(만료/이미사용/불일치) UI가 없음.
  → **추가**: "QR-scan input / reservation lookup" 요소 + "거부 상태(만료·이미사용·명의불일치)" UI 요구사항 추가.

---

## 참고 — 문서 간 상충 안내

이전 세션에서 작성한 `prd/planning/v1.2-feedback-review.md`는 이 5개 이슈(등록/QR/웨이팅/플로우/지도)에 대해 **옵션을 제안하는 형태**로 작성했는데, 이후 `prd/feedBack/PRD_Feedback_v2.md` / `PRD_Feeback_v2.2.md`를 확인해보니 이미 다음과 같이 **확정**되어 있었습니다:
- 환전소 등록: 파트너 셀프신청이 아니라 **Admin 직접 등록 폼** (옵션 A로 확정)
- 웨이팅: 대기열 자동승격이 아니라 **슬롯 정원 초과 시 단순 예약불가** (제 제안보다 단순한 버전으로 확정)

두 문서가 상충하는 부분은 **`PRD_Feedback_v2.md`/`v2.2.md`(확정본)가 우선**이며, `v1.2-feedback-review.md`의 해당 항목(옵션 B, 웨이팅 자동승격 등)은 참고용 대안으로만 남겨두는 것을 권장합니다.
