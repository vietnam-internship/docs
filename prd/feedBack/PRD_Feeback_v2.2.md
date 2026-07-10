# PRD Revision - Feedback Reflection (2nd Review)
# PRD 수정 사항 - 2차 피드백 반영

---

# 1. Exchange Rate Comparison (환율 비교)

## Feedback
The core value of the service is allowing users to compare **exchange rates for the same currency across different exchange branches**, not comparing different currencies.

> 서비스의 핵심 가치는 서로 다른 통화를 비교하는 것이 아니라, **동일한 통화의 환율을 여러 환전소에서 비교할 수 있는 기능**입니다.

### Decision (반영 내용)

- Users will first select a currency (USD, JPY, EUR, etc.).
- The map will display exchange rates of the selected currency from nearby exchange branches.
- This comparison experience will become one of the main features highlighted in both the UI and PRD.

- 사용자는 먼저 환전할 통화(USD, JPY, EUR 등)를 선택합니다.
- 지도에는 선택한 통화에 대한 각 환전소의 환율이 표시됩니다.
- 동일 통화의 환율을 여러 환전소에서 비교하는 것이 서비스의 핵심 가치가 되도록 UI와 PRD를 수정합니다.

---

# 2. Branch Management (지점 관리)

## Feedback

Admin currently allows exchange rate management, but there is no functionality to create or manage exchange branches themselves.

> 현재 Admin에서는 환율 관리만 가능하며, 환전소(지점) 자체를 생성하거나 관리하는 기능이 없습니다.

### Decision (반영 내용)

A new **Branch Management** module will be added to the Admin system.

관리자(Admin)에 **Branch Management** 기능을 추가합니다.

### Features (기능)

- Create branch
- Edit branch
- Delete / Deactivate branch
- Manage:
  - Branch name
  - Address
  - Business hours
  - Contact information
  - Supported currencies
  - Branch status

관리자는 다음 정보를 직접 관리할 수 있습니다.

- 환전소 생성
- 환전소 수정
- 환전소 삭제 또는 비활성화
- 환전소 이름
- 주소
- 운영 시간
- 연락처
- 취급 통화
- 운영 상태(Open / Closed)

---

# 3. Minimum Exchange Amount (최소 환전 금액)

## Feedback

Support a configurable minimum exchange amount.

> 최소 환전 가능 금액을 설정할 수 있도록 기능을 추가합니다.

### Decision (반영 내용)

The minimum exchange amount will be:

**10,000 VND**

- Reservations below the minimum amount cannot be completed.

최소 환전 금액은

**10,000 VND**로 설정합니다.

- 최소 금액 미만은 예약할 수 없습니다.
- 예약 단계에서 검증됩니다.

---

# 4. Branch Location Management (지도 위치 설정)

## Feedback

Although users can search nearby branches on the map, the Admin flow for configuring branch locations is missing.

> 사용자 지도에는 환전소가 표시되지만, 관리자가 위치를 등록하는 기능이 없습니다.

### Decision (반영 내용)

The Admin system will allow branch location management.

관리자는 각 환전소의 위치를 직접 설정할 수 있습니다.

### Features

- Set latitude & longitude
- Edit location
- (Optional) Select location directly from map

관리자는

- 위도(Latitude)
- 경도(Longitude)

를 입력하여 환전소 위치를 등록하며,

등록된 좌표를 기반으로 사용자 지도에서 환전소를 표시합니다.

향후에는 지도에서 직접 위치를 선택하는 기능도 고려합니다.

---

# 5. Online Payment (온라인 결제)

## Feedback

Allow customers to complete payment online before arriving so that they only need to present a QR code at pickup.

> 예약 시 온라인 결제를 완료하고, 방문 시 QR 코드만 제시하여 빠르게 환전할 수 있도록 제안되었습니다.

### Our Response (팀 답변)

We believe online payment is a valuable feature.

However, our target users are international travelers arriving in Vietnam from many different countries.

Because banking systems and payment policies vary significantly across countries, we would appreciate recommendations regarding:

- Global payment APIs
- Cross-border payment solutions
- Sandbox payment APIs suitable for demonstrations

Additionally, most payment gateways require a registered business license before production integration.

For these reasons, online payment will be considered as a future enhancement rather than an MVP feature.

온라인 결제는 매우 좋은 기능이라고 생각합니다.

다만, 본 서비스의 주요 사용자는 여러 국가에서 베트남으로 입국하는 해외 여행객이므로 국가별 결제 시스템과 정책이 크게 다릅니다.

또한 대부분의 결제 API는 실제 사업자 등록이 필요하기 때문에 MVP 단계에서 구현하기에는 현실적인 제약이 있습니다.

따라서

- 국제 결제 API 추천
- Cross-border Payment Solution
- Sandbox 환경

등에 대한 멘토님의 의견을 참고하여 추후 기능으로 확장할 예정입니다.

---

# Additional Improvements (추가 개선 사항)

## QR Reservation Verification (QR 예약 인증)

To prevent reservation misuse:

- Every reservation generates a unique QR code.
- Exchange staff verify reservations by scanning the QR code.
- QR codes become invalid immediately after use.

예약 이미지 캡처 등의 악용을 방지하기 위해

- 예약 완료 시 고유 QR 코드 생성
- 직원이 QR 코드를 스캔하여 예약 확인
- 사용 완료된 QR은 즉시 만료

방식으로 운영합니다.

---

## Waiting Queue Policy (대기 정책)

To avoid congestion:

- Reservation slots are managed every **30 minutes**.
- Each exchange branch defines the maximum number of reservations per time slot.
- Fully booked slots become unavailable.

혼잡을 방지하기 위해

- 예약은 **30분 단위**로 운영합니다.
- 환전소마다 시간대별 최대 예약 가능 인원을 설정합니다.
- 최대 인원을 초과하면 해당 시간은 예약이 불가능합니다.

---

# MVP Priority (우선 개발 범위)

Following the mentor's recommendation, we will focus on completing the Reservation Flow before implementing additional features.

멘토님의 의견에 따라 다양한 부가 기능보다 **예약 프로세스 완성도**를 최우선 목표로 개발합니다.

Priority

1. Branch Management
2. Exchange Rate Comparison
3. Reservation Flow
4. QR Verification
5. Waiting Queue
6. Map Integration
7. Online Payment (Future Enhancement)

우선순위

1. 환전소 관리
2. 동일 통화 환율 비교
3. 예약 기능
4. QR 인증
5. 대기열 관리
6. 지도 연동
7. 온라인 결제 (향후 기능)
