# F4: My Reservations 플로우 — 와이어프레임 정리 & PRD 반영 제안

작성 목적: 와이어프레임 작업 중 정리된 F4(My Reservations) 플로우 내용을 정리하고, `TravelX_PRD_v1.2_EN.pdf` 대비 빠진 부분과 반영 위치를 제안한다. (`reservation/wireFrame-feature3.md`와 동일한 형식)

참고 문서: `RateDiscovery/prd_update_checklist.md`(F2, 동일 형식), `reservation/wireFrame-feature3.md`(F3), `PRD_Feedback_v2.md`/`PRD_Feeback_v2.2.md`(QR·Time Slot 관련 결정은 이미 확정 — 이 문서는 그 결정이 PRD 본문에는 아직 반영 안 된 부분을 짚는다)

---

## 1. 지금까지 와이어프레임에서 정리된 내용 (F4: My Reservations)

| 화면 ID | 주요 내용 |
|---|---|
| MYRSV-01 | Look up the list of currency exchanges that you have currently booked. It will be implemented as an infinite scroll. |
| MYRSV-02 | Inquire the list of completed currency exchange transaction details. There are two states: Completed and Canceled. It will be implemented as an infinite scroll. |
| MYRSV-03 | Check the details of past currency exchange transaction details |

> ※ 이 표는 3개 화면뿐이지만 `check_data.md` 원본은 7개 화면(RSV-01~04 + 서브화면 3개)을 정의한다. 아래 §3에서 원본 체계를 병기하고, 대응 관계는 §5 질문으로 남긴다 — 여기서 임의로 매핑하지 않음.

---

## 2. PRD 대비 빠진 부분

### 2.1 Cash Pickup 항목에 QR 언급 없음
- PRD §6.1(p.5) "Cash Pickup": "Present reservation confirmation" 한 줄뿐.
- → `PRD_Feedback_v2.md` #2에서 이미 "이미지 → QR Code, 1회 사용"으로 결정됐는데 본문에 미반영.
- **반영 위치 제안**: "Present QR code (single-use) for staff to scan; QR is invalidated immediately upon redemption."로 수정.

### 2.2 "My Reservations"와 "Exchange History"가 구분되지 않음
- PRD §6.1(p.5) "Reservation Management": "View reservations / Cancel reservation / View reservation history" 세 줄이 뭉뚱그려짐. §8(p.10) "Reservation History" FR도 동일.
- → `check_data.md`는 이 둘을 별개 화면으로 명확히 분리(RSV-01: 전체 상태 탭 vs RSV-04: completed/cancelled 전용 + 금액상세).
- **반영 위치 제안**: §6.1과 §8 모두 "View reservations"와 "View exchange history"를 별개 항목으로 분리하고 각각의 스코프(상태 필터, 표시 필드) 명시.

### 2.3 Staff 플로우에 QR 스캔 단계 없음
- PRD §7.2(p.8) Userflow Staff: `Receive reservation alert → Check upcoming visitors → Verify ID and take payment → Mark exchange complete`.
- → QR 스캔 단계 없이 바로 "Verify ID"로 넘어감 — QR 결정과 어긋남.
- **반영 위치 제안**: "Check upcoming visitors"와 "Verify ID and take payment" 사이에 "Scan customer's QR code → look up reservation" 노드 삽입.

### 2.4 Tourist 플로우에 예약 관리(조회/취소/이력) 흐름 없음
- PRD §7.3(p.9) Userflow Tourist: 예약 생성→픽업까지의 선형 해피패스만 존재("Show ID for verification"까지).
- → `check_data.md`의 예약 목록 조회/상세/취소/이력에 해당하는 흐름이 §7 어디에도 없음.
- **반영 위치 제안**: (a) "Show ID for verification" 노드를 "Show QR code + ID for verification"으로 수정, (b) 신규 "7.4 Userflow — My Reservations / Cancellation" 다이어그램 추가하거나 F4 문서를 §7에 명시적으로 상호 참조.

### 2.5 "Cash Pickup" FR 행 자체가 없음
- PRD §8(p.10-11) Functional Requirements 표: Sign In~Branch Rate & Stock Setting까지 19개 행 중 Cash Pickup 행이 없음. 가장 가까운 "Identity Verification (KYC)" 행은 신분증 대조만 다룸.
- → QR 디코드·상태검증·1회성 처리·완료전환 프로세스가 어디에도 없음.
- **반영 위치 제안**: 새 행 `Cash Pickup` — Input: QR코드(or 예약번호)+제시된 ID / Process: QR 디코드 → 예약 status==RESERVED 검증 → ID와 예약자 정보 대조 → 원자적으로 COMPLETED 전환(재사용 거부) / Output: 픽업 승인 또는 거부(만료·이미사용·불일치).

### 2.6 "Reservation Confirmation" FR에 QR 발급 누락
- PRD §8(p.10): Process "Generate reservation number"뿐.
- → QR 발급은 이미 확정된 결정인데 반영 안 됨.
- **반영 위치 제안**: Process를 "Generate reservation number and a unique single-use QR code encoding the reservation ID"로 수정.

### 2.7 취소 확인 모달 요구사항 없음
- PRD §8(p.10) "Cancel Reservation": Process "Validate ownership & status, release held stock".
- → `check_data.md`는 취소 전 확인 팝업을 필수 요구("즉시 취소 금지"가 검증 체크리스트에 명시)하지만 FR 표엔 이 UX 조건이 없음.
- **반영 위치 제안**: Process 설명에 "(클라이언트) 취소 확정 전 확인 모달 필수" 각주 추가.

### 2.8 "Download Receipt" 기능이 PRD 전체에 없음
- PRD §8(p.10-11), §14(p.19) 어디에도 영수증 생성/다운로드 관련 행이 없음.
- → `check_data.md`의 completed 상태 화면에 "Download receipt" 버튼이 있음.
- **반영 위치 제안**: §8에 새 행 `Download Receipt`(Input: Reservation ID(status==COMPLETED) / Process: 예약+지점+환율 데이터로 영수증 생성 / Output: 다운로드 가능한 영수증 파일), §14에 `GET /reservations/{id}/receipt` 추가.

### 2.9 Reservation 테이블에 lockedRate/pickedUpAt 필드 없음
- PRD §12(p.16) Reservation: id, userId, currencyId, branchId, amount, pickupDate, pickupTime, status.
- → §8 Reservation FR Process엔 "lock rate"가 명시되어 있는데 저장할 필드가 없음. `check_data.md`의 amountFrom/amountTo(환전 전후 금액) 표시를 안정적으로 하려면 예약 시점에 잠긴 환율이 필요. 픽업 완료 시각을 기록할 필드도 없어 감사(audit) 불가.
- **반영 위치 제안**: `Reservation.lockedRate`(or `krwAmount`), `Reservation.pickedUpAt`(nullable timestamp) 필드 추가.

### 2.10 Branch에 층/부스 등 상세 위치 필드 없음
- PRD §12(p.15) Branch: id, name, address, phone, latitude, longitude, businessHours.
- → `check_data.md` 데이터 모델의 `locationDetail`(상세 위치)에 대응하는 필드가 없음.
- **반영 위치 제안**: Branch(or Reservation)에 `pickupLocationDetail` 자유 텍스트 필드 추가.

### 2.11 ReservationStatus 라벨이 세 문서에서 제각각
- DB enum(§12, p.16): `RESERVED`/`COMPLETED`/`CANCELLED` vs §24 Wireframe 표(p.27): "pending/complete/cancelled" vs 실제 화면 카피(`check_data.md`): "Upcoming/Completed/Cancelled".
- → 세 곳이 서로 다른 용어를 씀. `pending`을 문자 그대로 렌더링하면 실제 카피와 어긋남.
- **반영 위치 제안**: enum→UI 라벨 매핑 각주 추가(`RESERVED`→"Upcoming" 등), §24의 "pending"을 "Upcoming"으로 정정.

### 2.12 예약 단건 조회 API 없음
- PRD §14(p.19): `GET /reservations`(목록), `DELETE /reservations/{id}`(취소)만 존재.
- → 예약 상세 화면이 필요로 하는 단건 조회 API가 없음.
- **반영 위치 제안**: `GET /reservations/{id}` 신설(QR 페이로드 포함).

### 2.13 QR 스캔/픽업 확정용 지점(Staff) 엔드포인트 없음
- PRD §14(p.19): `/admin/reservations`(전체 조회), `PATCH /branches/{id}/rate`(우대율/재고 갱신)만 존재.
- → 직원이 QR을 스캔해 픽업을 확정 처리하는 API가 없음.
- **반영 위치 제안**: 예) `POST /branches/{id}/reservations/{reservationId}/redeem` — QR 스캔 결과로 상태검증 후 COMPLETED 전환.

### 2.14 QR 픽업/상환 시퀀스 다이어그램 없음
- PRD §17(p.22): 17.1(환율비교), 17.2(예약 생성+취소) 두 개뿐.
- → `check_data.md`가 가장 상세히 다루는 QR 픽업/상환(redemption) 흐름에 대한 다이어그램이 없음.
- **반영 위치 제안**: "17.3 Cash Pickup / QR Redemption" 추가 — User QR 제시 → Staff 앱 스캔 → Server 디코드+상태검증+ID대조 → DB status RESERVED→COMPLETED → 승인/거부 응답.

### 2.15 Time-slot 정책 미반영 (부차적, F4 화면 자체엔 슬롯 UI 불필요)
- `PRD_Feedback_v2.md` #3 / `PRD_Feeback_v2.2.md`(Waiting Queue Policy)에서 "30분 단위 Time Slot + 지점별 최대 예약 인원, 초과 시 예약 불가"로 이미 확정됐는데 §19(p.24)·§12 어디에도 반영 안 됨.
- → 참고: F4(My Reservations) 화면 자체엔 슬롯 선택 UI가 없는 게 맞음(슬롯 선택은 F3 Reservation Flow 도메인). 다만 정책 문서 자체엔 반영 필요.
- **반영 위치 제안**: §19에 Time-slot capacity 행 추가, §12 Branch에 슬롯당 최대인원 필드 추가.

### 2.16 §24 Wireframe Requirements 다중 이슈
- PRD §24(p.27): Home/Exchange Comparison/Reservation/AI Recommendation/My Reservations/Staff Confirmation 6개 행뿐.
- → (a) "My Reservations" 행 "pending/complete/cancelled"가 §2.11과 동일 이슈. (b) "countdown to expiry" 요구사항이 있는데 `check_data.md`/MYRSV 어디에도 카운트다운 UI가 없음 — 디자인 누락인지 요구사항이 stale한지 확인 필요. (c) Reservation Complete/Reservation Detail/Exchange History 3개 화면에 대응하는 행 자체가 없음. (d) "Staff Confirmation" 행에 QR 스캔 요소, 거부 상태(만료/이미사용/불일치) UI가 없음.
- **반영 위치 제안**: (a) "Upcoming"으로 정정, (b) 디자이너 확인 후 추가 또는 삭제, (c) 3개 행 신설, (d) QR-scan 요소 + 거부 상태 UI 요구사항 추가.

---

## 3. 화면 ID 현황 — check_data.md 원본 체계 vs MYRSV 체계 (해결 보류, §5 참고)

`check_data.md` 원본은 아래 7개 화면을 정의한다 (§1의 MYRSV-01~03과 별개로 병기 — 대응 관계는 미확정):

```
RSV-01          My Reservations (목록, Upcoming/Completed/Cancelled 탭)
RSV-02          Reservation Complete (신규 예약 제출 직후)
RSV-03          Reservation Detail (상태별 분기: QR/취소/영수증)
RSV-03-1        QR Code View (RSV-03 내 인라인 확장)
RSV-03-2-POPUP  Cancellation Confirm (모달)
RSV-03-3        Cancellation Complete
RSV-04          Exchange History (Completed/Cancelled 전용, 별도 진입점)
```

같은 프로젝트 안에서 `RSV-` 접두어가 이미 두 군데에서 쓰이고 있다: F4(위 원본) 자체, 그리고 `reservation/wireFrame-feature3.md`(F3)도 `RSV-01~04`를 전혀 다른 의미(Reservation Home/Time&Currency Selection/Reservation Detail/My Reservations)로 사용 중. `RateDiscovery/prd_update_checklist.md`(F2)에서도 동일한 충돌을 지적함. 이번에 도입된 `MYRSV-` 접두어가 이 충돌을 해소하려는 시도로 보이나, 위 7개 화면 중 3개(MYRSV-01~03)만 다시 정의되어 있어 나머지 4개(Reservation Complete, QR View, 취소 확인/완료)의 거취가 불명확 — **§5로 넘긴다.**

---

## 4. PRD 문서 반영 제안 위치

| 반영 내용 | PRD 위치 |
|---|---|
| QR 언급(Cash Pickup, Reservation Confirmation) | §6.1, §8 |
| My Reservations / Exchange History 분리 | §6.1, §8 |
| Staff 플로우 QR 스캔 단계 | §7.2 |
| Tourist 플로우 예약관리 흐름 | §7.3 (또는 F4 문서 상호참조) |
| Cash Pickup FR 신설 | §8 |
| 취소 확인 모달 각주 | §8 |
| Download Receipt FR + API | §8, §14 |
| lockedRate/pickedUpAt 필드 | §12 Reservation |
| pickupLocationDetail 필드 | §12 Branch |
| ReservationStatus UI 라벨 매핑 | §12, §24 |
| 예약 단건 조회 API | §14 |
| QR 스캔/픽업확정 API | §14 |
| QR 픽업 시퀀스 다이어그램 | §17 |
| Time-slot capacity 정책 | §19, §12 |
| Reservation Complete/Detail/Exchange History 행 신설 | §24 |
| 화면 ID 체계 재정리 | §24 (팀 확인 후) |

---

## 5. 확인이 필요한 질문 (팀/멘토 논의용)

1. MYRSV-01/02/03은 `check_data.md` 원본의 RSV-01(My Reservations)/RSV-04(Exchange History)/RSV-03(Reservation Detail)에 각각 대응하는가?
2. Reservation Complete(RSV-02), QR 코드 화면(RSV-03-1), 취소 확인 팝업/완료(RSV-03-2-POPUP/RSV-03-3)는 MYRSV 목록에서 빠졌는데 — `wireFrameDomain.md` 기준 "예약 완료(QR 발급)"는 F3(Reservation Flow) 5단계로 되어 있어 F3 도메인으로 옮겨간 것인가, 아니면 F4에 별도 화면(MYRSV-04 등)으로 추가될 예정인가?
3. F3가 이미 `RSV-` 접두어를 쓰고 있고 F4 원본도 `RSV-`를 쓰는데, 최종적으로 F4는 `MYRSV-`로 전면 통일하는 것이 맞는가? (F2 체크리스트에서도 동일 충돌 지적 — 세 도메인 전체의 접두어 규칙을 한 번에 정리하는 게 나을 수 있음)
4. §24 "pending" vs 실제 카피 "Upcoming" 중 무엇이 최종인가?
5. countdown-to-expiry 요소가 실제 디자인(MYRSV-01 또는 03)에 있는가, 아니면 §24 요구사항 쪽을 수정해야 하는가?

---

## 참고 — 문서 간 상충 안내

이전 세션에서 작성한 `prd/planning/v1.2-feedback-review.md`는 등록/QR/웨이팅/플로우/지도 이슈에 대해 **옵션을 제안하는 형태**로 작성했는데, 이후 `prd/feedBack/PRD_Feedback_v2.md` / `PRD_Feeback_v2.2.md`를 확인해보니 이미 다음과 같이 **확정**되어 있었다:
- 환전소 등록: 파트너 셀프신청이 아니라 **Admin 직접 등록 폼** (옵션 A로 확정)
- 웨이팅: 대기열 자동승격이 아니라 **슬롯 정원 초과 시 단순 예약불가** (더 단순한 버전으로 확정)

두 문서가 상충하는 부분은 **`PRD_Feedback_v2.md`/`v2.2.md`(확정본)가 우선**이며, `v1.2-feedback-review.md`의 해당 항목(옵션 B, 웨이팅 자동승격 등)은 참고용 대안으로만 남겨두는 것을 권장한다.
