# TravelX PRD v1.2 → v1.3 변경 요약 (재검토용 인덱스)

> 산출물: `prd/TravelX_PRD_v1.3_EN.docx` (원본 `prd/TravelX_PRD_v1.2_EN.docx`는 보존됨)
> 작성: AI(Claude) / 대상: `prd/feedBack/` 아래 모든 와이어프레임 피드백 문서 + `be/PRD_Feedback_v2.md`, `be/PRD_Feeback_v2.2.md`(2차 피드백 확정본)
> 각 도메인별 상세 변경 내역은 아래 표에서 링크된 개별 문서를 참고.

## 도메인별 변경 내역서

| 도메인 | 문서 | 주요 내용 |
|---|---|---|
| 인증 & 마이페이지/알림 | [`Auth_MyPage/prd_v1.3_changes.md`](Auth_MyPage/prd_v1.3_changes.md) | Profile 조회 기능 신설, 알림 조회/읽음처리 기능 신설, 관련 API/wireframe 행 추가 |
| 환율 탐색 (Rate Discovery) | [`RateDiscovery/prd_v1.3_changes.md`](RateDiscovery/prd_v1.3_changes.md) | 최근검색/인기통화, best-rate-nearby 배지, high-volatility 배지, 지점 단건조회 API, wireframe 4행 신설 |
| 예약 프로세스 (Reservation Flow) | [`reservation/prd_v1.3_changes.md`](reservation/prd_v1.3_changes.md) | 시간대(Time-slot) 정원 정책 신설, QR 픽업 시퀀스(§17.3), 유저플로우 다이어그램 텍스트 노트 |
| 내 예약 관리 (My Reservations) | [`MyReservation/prd_v1.3_changes.md`](MyReservation/prd_v1.3_changes.md) | QR 단일사용 픽업 인증, Cash Pickup/Download Receipt FR 신설, DB 필드/상태라벨 정리 — **변경 폭 최대** |
| 관리자 (Admin) | [`Admin/prd_v1.3_changes.md`](Admin/prd_v1.3_changes.md) | Branch 직접등록 CRUD, 파트너 셀프온보딩 vs Admin직접등록 포크 해소, 최소환전금액 정책 |

## 전체 변경의 성격 — 세 개 버킷

각 도메인 문서는 변경 사항을 다음 세 가지로 분류해 표시한다 (재검토 시 이 구분을 기준으로 봐주면 된다):

1. **적용 — 확정 결정 반영**: `be/PRD_Feedback_v2.md` / `PRD_Feeback_v2.2.md`에 이미 팀이 확정한 내용(QR 단일사용, 30분 Time-slot + 지점별 정원, Admin 직접 등록, 최소환전금액, 지도=등록지점만 표시)인데 PRD 본문 텍스트에는 반영되지 않았던 공백을 메운 것. 여기는 **재논의 대상이 아니라 확인용**이다.
2. **적용 — 안전한 보강(safe-additive)**: 와이어프레임에는 이미 존재하지만 PRD 문서 어디에도 대응 항목이 없었던 것(API 엔드포인트, DB 필드, wireframe 참조 표 행 등). 틀릴 위험이 낮아 바로 반영했다.
3. **미적용 — 팀 결정 필요**: 여러 문서가 서로 다른 답을 암시하거나(화면 ID 접두어 충돌, 지점 랭킹 로직 두 가지 후보), 확정된 근거가 없는 항목(countdown-to-expiry 요소 존재 여부, 디스클레이머 노출 여부). **PRD 본문을 임의로 고치지 않고 각 도메인 문서 마지막 섹션에 원본 그대로 이월**했다.

## 도메인을 가로지르는 이슈 (여러 문서에 걸쳐 반복 언급됨)

- **QR 픽업 인증**: MyReservation·Reservation Flow 도메인 양쪽에 걸쳐 있다. §6.1 Cash Pickup, §8 FR(Cash Pickup 신설), §12(pickedUpAt), §14(redeem API), §17.3(신규 시퀀스)에 반영. **§17.3의 렌더링된 다이어그램 이미지는 아직 없음** — mermaid 텍스트 소스만 반영, §17.1/17.2 스타일로 이미지 삽입 작업이 남아있다.
- **유저플로우 다이어그램(§7.1/7.2/7.3) 재작도 필요**: 세 그림 모두 래스터 PNG라 이번 작업에서 그림 자체는 고치지 못했다. 각 그림 아래에 "[v1.3 note — diagram not yet redrawn]" 텍스트로 정확한 델타를 적어두었으니, 디자이너가 그 문구를 보고 그림을 다시 그리면 된다.
  - §7.1: 파트너 신청 모델 → Admin 직접등록 모델로 변경
  - §7.2: "Scan customer's QR code → look up reservation" 단계 삽입
  - §7.3: "Show ID for verification" → "Show QR code + ID for verification"
- **화면 ID 접두어(`RSV-`) 충돌**: F2(Rate Discovery)·F3(Reservation Flow)·F4(My Reservations) 세 문서가 `RSV-01~04`를 서로 다른 화면에 사용 중이다. PRD 본문 자체는 화면 ID를 쓰지 않으므로 PRD엔 영향이 없지만, 팀의 wireframe 문서 전체를 대상으로 접두어를 한 번에 정리하는 것을 권장한다 (개별 도메인 문서마다 반복 언급된 이슈).
- **ReservationStatus 라벨 통일**: enum(`RESERVED`/`COMPLETED`/`CANCELLED`) ↔ §24 wireframe 표기(과거 "pending") ↔ 실제 화면 카피("Upcoming") 세 곳이 달랐던 것을, 실제 완성된 와이어프레임 카피 기준으로 "Upcoming"으로 통일하고 §12에 매핑 각주를 추가했다.

## 전체 개정판 부가 사항

- 표지: **Version 1.2 → 1.3**, 날짜 **July 10, 2026**으로 갱신.
- 표지 바로 뒤에 **Revision History** 섹션(신규) 추가 — v1.2/v1.3 요약과 이 인덱스 문서로의 참조 포함.
- 원본 `TravelX_PRD_v1.2_EN.docx`는 그대로 보존되어 있으며, `TravelX_PRD_v1.3_EN.docx`는 이를 복사한 뒤 수정한 별도 파일이다.

## 다음 단계 제안

1. 각 도메인 문서의 **"팀 확인이 필요한 사항"** 섹션을 먼저 검토 — 여기 답이 나와야 §7 다이어그램 재작도, §18.3 랭킹 로직, 화면 ID 접두어 통일 등 후속 작업 범위가 확정된다.
2. §7.1/7.2/7.3, §17.3 다이어그램 이미지 재작도를 디자이너에게 배정.
3. 확인 완료된 항목부터 순서대로 반영하고, 이 문서(및 각 도메인 문서)를 v1.4 체크리스트로 갱신.
