# F1/F5: Auth & MyPage 플로우 — 와이어프레임 정리 & PRD 반영 제안

작성 목적: 와이어프레임 작업 중 정리된 F1(Auth)·F5(MyPage & Notification) 플로우 내용을 정리하고,
`TravelX_PRD_v1_2_EN.pdf` 대비 빠진 부분과 반영 위치를 제안한다.

---

## 1. 지금까지 와이어프레임에서 정리된 내용 (F1: Auth / F5: MyPage & Notification)

| 화면 ID | 역할 | 주요 내용 |
|---|---|---|
| LOGIN-01 | 로그인 (진입점) | TravelX 스플래시 / "Continue with Google" → Google 로그인 페이지 / "Sign up for TravelX" → REGISTER-01 |
| REGISTER-01 | 프로필 설정 | Google 인증 후 이메일 읽기전용 자동표시, Full Name·Phone Number 수동 입력, 약관 체크박스, "Sign Up →"로 가입 완료 후 LOGIN-01 복귀 |
| HOME-01 | 홈 (알림 진입점) | Exchange overview(예약 요약), Live rate comparison, AI Report / 알림 벨 아이콘 → HOME-02 |
| HOME-02 | Notification 팝업 | 환율 알림·픽업 리마인더·예약 확정 알림 3종 리스트(아이콘+제목+설명), 바깥 탭 시 HOME-01 복귀 |
| MY-01 | MyPage | 프로필(이름/이메일) / "My Reservation" → (My Reservation 도메인 MYRSV-01 표기) / "Exchange history" → (My Reservation 도메인 MYRSV-02 표기) / "Logout" → LOGIN-01 |

---

## 2. PRD 대비 빠진 부분

### 2.1 프로필 완성 단계(REGISTER-01)가 PRD Auth FR에 없음
- PRD §6.1: Auth 기능은 "Sign in with Google" / "Log out" 두 가지로만 정의
- PRD §8 FR: "Sign In with Google" Process는 "Verify token with Google; **auto-create account on first login** or retrieve existing user; issue JWT" — 자동 생성만 명시, 별도 입력 단계 없음
- → REGISTER-01은 Full Name·Phone Number를 사용자가 직접 입력해야 통과되는 중간 화면인데 PRD 근거가 없음
- **반영 위치 제안**: §8 "Sign In with Google" Process를 2단계로 분리 — ① OAuth 콜백으로 신규/기존 유저 판별 ② 신규 유저는 REGISTER-01(프로필 완성) 경유, 기존 유저는 바로 Home으로 라우팅

### 2.2 User 테이블에 phone 필드 없음
- PRD §12 Database Design: `User(id, name, email, googleId, role)` — phone 필드 없음
- → REGISTER-01에서 수집하는 Phone Number를 저장할 위치가 PRD DB 설계에 없음
- **반영 위치 제안**: §12 User 테이블에 `phone` 필드 추가

### 2.3 REGISTER-01의 phone 수집과 §19.3/§21.3 KYC 인증의 관계 불명확
- PRD §19.3, §21.3: "two-stage verification: **phone verification at reservation time**, and staff ID check at pickup" — 전화 인증은 가입 시점이 아니라 예약 시점에 발생하는 것으로 설계
- → REGISTER-01은 가입 단계에서 이미 Phone Number를 수집하는데, 이 값을 예약 시 그대로 재사용(프리필+재인증)하는지 별도로 다시 받는지 불명
- **반영 위치 제안**: §8 Identity Verification(KYC) FR 옆에 "예약 화면 전화번호는 프로필 값 프리필 + OTP 재인증" 여부 각주 추가

### 2.4 REGISTER-01 체크박스 탭 목적지 오류 추정
- 원본 화면 설명: "Tapping the checkbox area (Terms of Service / Privacy Policy link) navigates to **Google Login Page**"
- → 이미 Google 로그인을 마친 상태에서 약관 링크 탭이 Google 로그인 페이지로 이동하는 것은 플로우상 부자연스러움
- **반영 위치 제안**: 팀 확인 후 와이어프레임 설명 수정 — 약관 링크는 ToS/Privacy Policy 정적 페이지(또는 웹뷰)로 이동하는 것이 맞는지 재확인

### 2.5 Full Name 필드 프리필 여부 모순
- 원본 설명: "Full Name and Phone Number are **entered manually** by the user"
- 실제 목업: Full Name 필드에 "John Doe"가 이미 채워져 있음(Google 프로필 값으로 추정)
- → 설명 텍스트(수동 입력)와 목업(프리필 표시)이 서로 다름
- **반영 위치 제안**: REGISTER-01 설명 재정리 — Google 프로필 이름을 프리필 + 수정 가능으로 할지, 완전 수동 입력으로 할지 확정 후 §8 FR에도 반영

### 2.6 HOME-02 알림에 MVP 범위 밖 알림 유형이 포함됨
- PRD §6.1, §8 Notifications FR: MVP 알림은 예약 완료/취소/픽업 리마인더 3종만 정의
- PRD §16 Future Features: "Exchange rate alerts (notify when target rate is reached)"는 **MVP 이후 기능**으로 명시
- → HOME-02 목업의 3개 알림 중 "JPY rate is 3.2% lower than its 30-day average" 항목은 §16에 못박힌 환율 알림(rate alert)에 해당 — 나머지 2개(픽업 리마인더/예약 확정)만 PRD와 일치
- **반영 위치 제안**: §6.1/§8/§16 중 하나에서 범위를 먼저 확정 — (a) 목업에서 해당 알림 제거, 또는 (b) MVP로 승격 시 §9 이동평균 로직 재사용한 트리거 조건과 배치 작업을 §8/§9에 추가

### 2.7 Notification 테이블 type enum 미정의
- PRD §12 Notification 테이블: `type` 필드가 값 정의 없이 존재
- → HOME-02에서 아이콘이 다른 3종(하향 화살표/시계/핀)이 노출되는데, 각 알림의 `type` 값이 무엇인지, 아이콘이 type에서 매핑되는지 불명확
- **반영 위치 제안**: §12 또는 §8에 `type` enum 값 명시 (예: `PICKUP_REMINDER`, `RESERVATION_CONFIRMED`, `RESERVATION_CANCELLED`, 그리고 §2.6 결정에 따라 `RATE_ALERT` 포함 여부)

### 2.8 §24 와이어프레임 요구사항 표에 Login/Register, Notification, MyPage 행이 없음
- PRD §24: `Home, Exchange Comparison, Reservation, AI Recommendation, My Reservations, Staff Confirmation` 6개 행만 존재
- → Auth(Login/Register), Notification(HOME-02), MyPage(MY-01)에 대응하는 행이 하나도 없음
- **반영 위치 제안**: §24에 3개 행 추가
  - Login/Register — "Google sign-in entry point, post-OAuth profile completion form(Full Name/Phone), ToS/Privacy consent"
  - Notification — "Bell icon entry point, alert list with icon/title/description, dismiss-on-outside-tap"
  - MyPage — "Profile display(name/email), entry points to My Reservation & Exchange History, logout"

### 2.9 HOME-01의 예약 요약 노출이 §24 Home 행 정의에 없음
- PRD §24 Home 행: "Currency selector, amount input, entry point to the rate-comparison list"만 정의
- → HOME-01 상단 "Exchange overview"(현재 진행 중인 픽업 예약 요약)를 홈에 표시한다는 내용이 없음
- **반영 위치 제안**: §24 Home 행에 "가장 임박한 예약 요약(있을 경우)" 항목 추가, My Reservation 도메인과의 데이터 의존성을 §7 User Flow에 명시

### 2.10 AI Report의 "지속 기간" 문구가 §9.3 로직 범위 밖
- PRD §9.3: 회귀 + 이동평균/표준편차 기반 Now/Wait/Neutral 배지 — "다음날 예측값 vs 현재값 비교"까지만 정의
- → HOME-01 AI Report 문구 "Rates like this have historically lasted 2-4 days before reversing"는 과거 유사 패턴의 **지속 기간 통계**로, §9.3~9.5 어디에도 없는 별도 분석
- **반영 위치 제안**: §9에 과거 유사 이탈 구간의 평균 지속일수 산출 로직 추가, 또는 목업이 예시 텍스트일 뿐이라면 실제 구현 문구로 교체

### 2.11 §9.6 디스클레이머 누락
- PRD §9.6: "This recommendation is not investment advice and is provided for reference only." 표시가 필수 요소로 규정됨
- → HOME-01 AI Report 목업에는 해당 디스클레이머가 보이지 않음
- **반영 위치 제안**: 화면에 디스클레이머 문구 추가 확인

### 2.12 "Bank-level security" 문구 근거 없음
- PRD §10 Non-functional Requirements: 보안 항목은 "JWT authentication (Google OAuth2 login only), role-based authorization" 정도만 명시
- → LOGIN-01 하단 "Bank-level security" 문구를 뒷받침할 근거가 PRD에 없고, §21 Legal & Compliance는 인턴십 단계에서 환전업 등록 자체가 면제됨을 명시하고 있어 과장 문구로 오해 소지가 있음
- **반영 위치 제안**: §10에 근거(TLS, 암호화 표준 등) 추가하거나, 문구 유지 여부 팀 재확인

### 2.13 MY-01의 화면 ID 참조가 My Reservation 도메인 실제 체계와 다름
- MY-01 설명: "My Reservation" → MYRSV-01, "Exchange history" → MYRSV-02로 표기
- My Reservation 도메인 문서 기준 실제 체계: `RSV-01`(My Reservations 목록), `RSV-04`(Exchange History) — `MYRSV-` 접두어 자체가 그 도메인에 존재하지 않음
- **반영 위치 제안**: MY-01 설명의 화면 ID를 `RSV-01`/`RSV-04`로 수정하거나, 도메인 간 접두어를 통일(`MYR-` vs `RSV-`)해서 재정리

---

## 3. 종합 제안 — 화면 ID 재정리 (초안)

```
LOGIN-01       Login / Splash (Google 로그인 진입)
REGISTER-01    Profile Setup (Full Name 프리필 여부 확정, Phone 수동 입력) ※ 항목 보강
REGISTER-01-1  ToS/Privacy Policy 정적 페이지 ※ 신규 검토 (체크박스 탭 목적지 오류 수정)
HOME-01        Home (Exchange overview, Live rate, AI Report + §9.6 디스클레이머) ※ 항목 보강
HOME-02        Notification Popup ※ 알림 유형 MVP 범위 확정 필요 (rate alert 포함 여부)
MY-01          MyPage (Profile) → RSV-01(My Reservation) / RSV-04(Exchange History) ※ ID 표기 수정
```

---

## 4. PRD 문서 반영 제안 위치

| 반영 내용 | PRD 위치 |
|---|---|
| 프로필 완성 단계(REGISTER-01) 존재 명시 | §8 Sign In with Google FR 갱신 |
| User.phone 필드 추가 | §12 User 테이블 |
| 가입 시 phone vs 예약 시 KYC phone 관계 각주 | §8 KYC FR / §19.3 |
| MVP 알림 유형 범위 확정(환율 알림 포함 여부) | §6.1, §8 Notifications FR, §16 Future Features |
| Notification.type enum 값 명시 | §12 Notification 테이블 |
| Login/Register, Notification, MyPage 행 추가 | §24 와이어프레임 요구사항 표 |
| Home 화면 예약 요약 노출 근거 | §24 Home 행 |
| AI Report 지속기간 로직 추가 또는 문구 교체 | §9.3~9.6 |
| MY-01 화면 ID 표기 수정 | §24 / My Reservation 도메인 문서와 교차 정리 |

---

## 5. 확인이 필요한 질문 (팀/멘토 논의용)

1. REGISTER-01의 Full Name은 Google 프로필 값 프리필 + 수정 가능인가, 완전 수동 입력인가?
2. 체크박스/약관 링크 탭 시 실제 이동 목적지는 Google Login Page가 맞는가, 오기인가?
3. 가입 시 수집한 Phone Number를 예약 시점 KYC 인증에서 재사용(프리필)하는가, 완전히 별도로 다시 받는가?
4. HOME-02의 환율 변동 알림(rate alert)을 MVP에 포함할 것인가, §16 Future Features대로 이번 범위에서 제외할 것인가?
5. MY-01에서 "My Reservation"/"Exchange history"가 가리키는 실제 화면 ID는 RSV-01/RSV-04가 맞는가?
6. HOME-01 AI Report의 "2-4일 지속" 문구는 실제 구현 목표(추가 통계 로직 필요)인가, 목업 예시 텍스트일 뿐인가?
