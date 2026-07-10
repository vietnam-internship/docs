# 온라인 환전 예약 시스템 — 화면 설계 문서 (PRD)

## 1. 개요
사용자가 공항/항구 등에서 환전을 미리 예약하고, 예약 내역을 조회·취소하며, 픽업 시 QR코드로 인증하는 모바일 앱 플로우.
디자인 기준 파일: `Currency Exchange Reservations.dc.html` (Figma 참고용 정적 export는 `figma-export/` 폴더)

- 플랫폼: iOS 모바일 (402×874, iPhone 기준)
- 톤: 미니멀, 화이트 배경 + 네이비(#1B3A5C) 포인트 컬러
- 상태 배지 컬러: Upcoming(연블루 #E9F0FA/네이비), Completed(연그린 #E8F5EA/그린 #2F7D3C), Cancelled(연그레이 #F1F2F4/그레이 #6B7280)

## 2. 데이터 모델 (예약 1건)
| 필드 | 설명 |
|---|---|
| id | 예약번호 (예: TX-20240501) |
| location / locationDetail | 픽업 장소명 / 상세 위치 (층, 부스 등) |
| date / time | 픽업 날짜 / 시간 |
| amountFrom / amountTo | 환전 전/후 금액 (통화 포함) |
| status | upcoming / completed / cancelled |

## 3. 화면 목록 (Screen ID 체계)

### RSV-01 — My Reservations (예약목록)
- 진입: 앱 첫 진입 화면(현재 플로우의 기본 시작점)
- 상단 헤더: "TravelX" 로고만 (뒤로가기 없음, 최상위 화면)
- "My reservations" 타이틀 + 우측 "Exchange history →" 링크
- 상태 탭: Upcoming / Completed / Cancelled (세그먼트 컨트롤)
- 예약 카드 리스트: 장소, 날짜·시간, 상태 배지, 환전 후 금액
- 하단 탭바: Home / Maps / Exchange(active) / Profile
- **탭 동작**
  - 탭 전환 → 해당 상태의 예약만 필터링
  - 카드 탭 → RSV-03 (Reservation Detail)로 이동, 해당 예약 선택
  - "Exchange history" 탭 → RSV-04 (Exchange History)로 이동

### RSV-02 — Reservation Complete (예약완료)
- 신규 예약 제출 직후 확인 화면
- 체크 아이콘(연그린 배경) + "Reservation complete" + 안내문
- 예약번호 카드
- 하단 요약: Amount / Location / Date
- **버튼**
  - "View reservation" → RSV-03 (Reservation Detail)
  - "Back to home" → RSV-01 (My Reservations)

### RSV-03 — Reservation Detail (예약상세)
- 헤더: 뒤로가기(← RSV-01) + "TravelX"
- 타이틀 "Reservation details" + 상태 배지 (우측 상단)
- 카드: Pickup location / Pickup time / Currency(from → to) / Reservation number
- 상태별 분기:
  - **upcoming**: "View QR code" 버튼 노출 → 탭 시 QR코드 섹션(RSV-03-1) 인라인으로 펼쳐짐 / "Cancel reservation" 버튼 노출
  - **completed**: "This exchange has been completed." 안내 카드 + "Download receipt" 버튼
  - **cancelled**: "This reservation has been cancelled." 안내 카드만 (액션 없음)
- 하단 고정 안내문: 요율 변동 및 신분증 지참 안내 (upcoming일 때만)

### RSV-03-1 — QR Code View (QR코드 보기)
- RSV-03 내부의 확장 섹션 (별도 화면 전환 아님)
- "View QR code" 탭 시 노출: QR 패턴(11×11) + 예약번호 + "Hide QR code"로 다시 접기
- upcoming 상태에서만 접근 가능

### RSV-03-2-POPUP — Cancellation Confirm (취소 확인 팝업)
- RSV-03 위에 뜨는 모달 (반투명 딤 배경)
- "Cancel this reservation?" + 경고 문구
- **버튼**
  - "Keep reservation" → 팝업 닫고 RSV-03 유지
  - "Cancel reservation" → 예약 status를 cancelled로 변경 후 RSV-03-3 (Cancellation Complete)로 이동

### RSV-03-3 — Cancellation Complete (취소완료)
- X 아이콘(연레드 배경) + "Reservation cancelled" + 안내문
- 예약번호 카드 + 요약(Amount/Location/Date)
- **버튼**
  - "Back to list" → RSV-01 (My Reservations)
  - "Back to home" → RSV-02 상당 지점(앱 홈, 이 기능 범위 밖)

### RSV-04 — Exchange History (환전 이력)
- 헤더: 뒤로가기(← RSV-01) + "TravelX"
- 타이틀 "Exchange history" + 부연 설명
- 탭: Completed / Cancelled (RSV-01의 탭과 별도 상태)
- 리스트 카드: 장소, 날짜·시간, 환전 전→후 금액, chevron
- **동작**
  - 카드 탭 → RSV-03 (Reservation Detail)로 이동 (completed/cancelled 상태의 read-only 상세)

## 4. 화면 전환 다이어그램 (텍스트)
```
RSV-02 (예약완료)
 ├─ View reservation ──────────────▶ RSV-03
 └─ Back to home ──────────────────▶ RSV-01

RSV-01 (예약목록)
 ├─ 카드 탭 ────────────────────────▶ RSV-03 (선택된 예약)
 └─ Exchange history ──────────────▶ RSV-04

RSV-03 (예약상세)
 ├─ [upcoming] View QR code ───────▶ RSV-03-1 (인라인 확장)
 ├─ [upcoming] Cancel reservation ─▶ RSV-03-2-POPUP
 ├─ [completed] Download receipt ──▶ (다운로드 액션, 화면전환 없음)
 └─ 뒤로가기 ───────────────────────▶ RSV-01 or RSV-04 (진입 경로에 따름)

RSV-03-2-POPUP (취소 확인)
 ├─ Keep reservation ──────────────▶ RSV-03로 복귀
 └─ Cancel reservation ────────────▶ RSV-03-3

RSV-03-3 (취소완료)
 ├─ Back to list ───────────────────▶ RSV-01
 └─ Back to home ───────────────────▶ (앱 홈, 범위 밖)

RSV-04 (환전 이력)
 ├─ 탭 전환 (Completed/Cancelled) ──▶ 리스트 필터링
 └─ 카드 탭 ────────────────────────▶ RSV-03 (read-only)
```

## 5. 범위 밖 (Out of scope)
- 실제 결제/환율 API 연동
- 로그인/인증 플로우
- Home, Maps, Profile 탭의 실제 내용 (본 문서는 Exchange 탭 하위 플로우만 다룸)
- 다국어 지원 (현재 전부 영문 카피, 원본 참고 디자인 기준)

## 6. 검증 시 확인 포인트
- [ ] RSV-01~RSV-04 각 화면의 필드/버튼이 위 명세와 일치하는지
- [ ] 상태(upcoming/completed/cancelled)별 RSV-03 분기 로직이 정확한지
- [ ] QR코드 노출/숨김 토글이 upcoming 상태에서만 가능한지
- [ ] 취소 플로우가 확인 팝업을 반드시 거치는지 (즉시 취소 금지)
- [ ] Exchange history 탭이 RSV-01의 상태 탭과 독립적으로 동작하는지
