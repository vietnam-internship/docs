# PRD v1.2 수정 체크리스트 — Rate Discovery 기준

> 비교 대상: `check_data.md`(Rate Discovery 화면 노트, SRCH-01~04) vs `TravelX_PRD_v1.2_EN.pdf`(전 28페이지)
> 교차 참고: `prd/feedBack/reservation/wireFrame-feature3.md`(F3 Reservation 플로우 검토 — 같은 방식으로 먼저 작성된 팀 문서), `PRD_Feeback_v2.2.md` #1(환율 비교 재정의: "통화 먼저 선택 → 그 통화의 지점별 환율을 지도에 표시"는 이미 확정됨, `check_data.md`의 흐름과 방향 일치)
>
> 형식: **[PRD 섹션 (페이지)] — 문제 → 수정 제안**. MyReservation 체크리스트와 동일한 원리.

---

## 1. 지금까지 와이어프레임에서 정리된 내용 (F3: Reservation)

| 화면 ID | 역할 | 주요 내용 |
|---|---|---|
| HOME-01 | 전체 사이트 Home | Search bar > "Search a country or currency" → Navigates to the currency search page (SRCH-03)
Currency row > "USD" → Navigates to the rate detail page (SRCH-04) |
| SRCH-03 | | · Recent searches and popular currencies are shown before any input
· Recent searches section is hidden if the user has no search history
Popular currencies > "USD" → Navigates to the rate detail page (SRCH-04) |
| SRCH-04 |  | Shows a list of 4 AI-recommended branches
Branches are ranked using a moving average and standard deviation of recent rates, not raw rate or distance
A high volatility badge appears above a defined swing threshold
Branch row > "TravelX Myeongdong" → Navigates to the branch detail page (SRCH-02)
Button > "Reserve now" → Starts the reservation flow with this currency pre-filled |
| SRCH-02 | 예약 확인/조회 | · "Best rate nearby" badge only shows if this branch has the lowest sell rate among nearby branches
· Hours show today's schedule bolded if the design needs to highlight it

Map preview → Navigates to the branch map page (SRCH-01)
Button > "Reserve now" → Carries this branch forward and skips branch selection in RSV-02 |


## ⚠️ 최우선 이슈 — 화면 ID(RSV-XX) 체계 충돌 (3개 문서가 서로 다름)

`check_data.md` 28번째 줄: `"Reserve now" → Carries this branch forward and skips branch selection in RSV-02` — 여기서 말하는 **RSV-02가 어느 문서의 RSV-02인지 확인이 안 됨.** 현재 존재하는 두 개의 다른 RSV 체계와 전부 충돌한다.

| 문서 | RSV-01 | RSV-02 | RSV-03 | RSV-04 |
|---|---|---|---|---|
| `MyReservation/check_data.md` | My Reservations(목록) | Reservation Complete | Reservation Detail | Exchange History |
| `reservation/wireFrame-feature3.md` | Reservation Home(지도+AI추천) | Time & Currency Selection | Reservation Detail/QR·Cancel | My Reservations(목록) |
| `RateDiscovery/check_data.md`이 암시하는 것 | (불명) | **지점 선택**이 일어나는 화면 | — | — |

`reservation/wireFrame-feature3.md` 자체 구조상 지점 선택은 RSV-01(지도) 또는 신설 제안된 RSV-01-1(Branch Detail)에서 일어나고, 그 문서의 RSV-02는 "시간·통화 선택"이지 "지점 선택"이 아니다. 즉 **RateDiscovery 쪽 설명("skips branch selection in RSV-02")과 F3 문서의 실제 RSV-02 정의가 서로 다르다** — 같은 예약 플로우를 두 팀이 다르게 이해하고 있을 가능성이 높다.
→ **PRD에 반영하기 전에 먼저 팀 내 확인 필요**: (1) RateDiscovery의 SRCH-02(지점 상세)가 F3의 RSV-01-1(Branch Detail, 아직 제안 단계)과 같은 화면인지, (2) "Reserve now"가 실제로 스킵하는 스텝이 F3 기준 몇 번인지. 확인 후 **화면 ID를 도메인별 접두어로 재정리**(예: MyReservation=`MYR-`, Reservation Flow=`RESV-`, Rate Discovery=`SRCH-`)해서 `RSV-`가 두 도메인에서 다른 의미로 재사용되지 않게 하는 것을 제안. (§24 와이어프레임 요구사항 개정 시 이 체계를 반영)

---

## §8 Functional Requirements (p.10) — Currency Search

- [ ] 현재: Input "currency code or country name" / Process "Query currency list" / Output "Matching currency list" — **"Recent searches"(최근 검색어)와 "Popular currencies"(인기 통화)를 입력 전에 보여주는 기능이 전혀 없음.** `check_data.md` SRCH-03은 이 두 섹션이 검색 입력 전 기본 노출임을 명시.
  → **추가**: Process에 "입력 없을 시 최근 검색어(사용자별) + 인기 통화(집계 기준) 조회" 추가. "인기 통화"의 집계 기준(예: 최근 7일 검색/예약 건수 상위 N개)을 명시해야 구현 가능.
- [ ] **"최근 검색어"를 저장할 DB 테이블이 없음** (§12 참고, 아래 항목).

## §8 Functional Requirements (p.11) — AI Exchange Shop Recommendation

- [ ] 현재 Process: "Score branches (distance/rate/availability/reservation) & rank" (§18.3의 w1~w4 공식 그대로). 그런데 `check_data.md` 16번째 줄은 "Branches are ranked using a **moving average and standard deviation of recent rates**, not raw rate or distance"라고 명시 — **이건 §18의 공식이 아니라 §9(AI Exchange Timing Recommendation)의 변동성 로직**이다. §18.1은 "이 기능은 §9(타이밍)와 독립적이며, 상점을 랭킹하지 타이밍을 다루지 않는다"고 명시적으로 선을 긋는데, 화면 설계 노트는 그 둘을 섞어서 쓰고 있음.
  → **확인 필요(우선순위 최상)**: SRCH-04의 "4 AI-recommended branches"가 실제로 어떤 알고리즘을 쓰는지 팀 확인 — (a) §18 공식대로 distance/rate/availability/reservation 조합이 맞다면 `check_data.md` 설명이 오기이므로 디자인 노트 정정, (b) 실제로 moving average/표준편차 기반이 맞다면 **§18.3 공식 자체를 개정**하거나 이 화면 전용의 제3의 로직(§9·§18과 별도)으로 §18에 명시.
- [ ] 위 (b)가 맞을 경우 **§12 DB 설계에 이를 뒷받침할 테이블이 없음.** `ExchangeRateHistory`는 `currencyId`만 갖고 `branchId`가 없어 통화 전체의 이력이지 **지점별 이력이 아님**. 지점별 moving average/표준편차를 계산하려면 지점별 시계열 데이터가 필요.
  → **추가(조건부)**: `BranchExchangeRateHistory`(branchId, currencyId, rate, recordedAt) 같은 테이블 신설 검토.
- [ ] **"High volatility badge"(swing threshold 초과 시 표시)가 PRD 어디에도 없음.** §9.6은 Now/Wait/Neutral 배지만 정의, §18.6도 별도 변동성 배지 언급 없음.
  → **추가**: §18.2 Inputs/Outputs 표(또는 §9)에 volatility badge 및 swing threshold 값 정의 추가.
- [ ] **"Best rate nearby" 배지 로직이 없음.** `check_data.md`(SRCH-02): "이 지점이 주변 지점 중 최저 매도율(sell rate)일 때만 표시" — 이는 §18의 **가중합산 total_score**와 다른, "순수 최저가 여부"만 보는 별도 boolean 플래그. 현재 §18.2 Output엔 `total_score`, `breakdown`, `is_open_now`, `reservation_available`만 있고 이 플래그가 없음.
  → **추가**: §18.2에 `is_best_rate_nearby` (boolean) 필드 추가.
- [ ] `check_data.md`는 "4개" 지점을 고정 노출하는데(SRCH-04), §18.2 `top_n`은 옵션이며 기본값 10.
  → **확인/수정**: SRCH-04 컨텍스트에서는 `top_n=4`로 호출한다는 것을 API 문서에 예시로 명시하거나, 기본값을 화면별로 다르게 쓴다는 점을 §14에 각주.

## §9 / §18 Display and Disclaimer (p.13, p.24)

- [ ] §9.6·§18.6은 "This recommendation is not investment advice..." 디스클레이머를 **필수 표시 요소**로 규정. `check_data.md`의 SRCH-04(AI 추천 지점 리스트), volatility badge 관련 노트엔 디스클레이머 언급이 없음(노트가 축약형이라 누락일 수 있음).
  → **확인 필요**: 디자인에 디스클레이머가 실제로 있는지 확인, 없으면 §24에 필수 요소로 재명시.

---

## §12 Database Design (p.15–16)

- [ ] **"최근 검색어(recent searches)" 저장 테이블 없음.** User/Currency/Branch/BranchCurrencyRate/ExchangeRateHistory/Reservation/Notification 중 검색 이력을 남기는 테이블이 없음.
  → **추가 테이블**: `SearchHistory`(id, userId, currencyId, searchedAt) 등.
- [ ] **"인기 통화" 산출 근거가 되는 집계 데이터/뷰가 없음.** 실시간 집계인지 배치인지도 불명.
  → **추가**: 인기 통화 산출 방식(예: 최근 N일 검색/예약수 상위 K개, 배치로 캐싱) §9 또는 §12에 명시.
- [ ] (위 §18 항목과 연결) 지점별 변동성 계산이 필요하다면 `BranchExchangeRateHistory` 테이블 신설 필요.

---

## §14 API Specification (p.19)

- [ ] `GET /currencies`에 **최근 검색어/인기 통화를 반환하는 파라미터나 별도 엔드포인트가 없음.**
  → **추가**: 예) `GET /currencies/recent`(로그인 사용자 최근 검색), `GET /currencies/popular`(인기 통화 랭킹).
- [ ] `GET /branches/recommend`(§18.2 대응)에 **`is_best_rate_nearby`, volatility badge 관련 필드가 응답 스펙에 없음** (위 §18 항목과 동일 이슈, API 레벨에서도 반영 필요).
- [ ] **지점 단건 상세 조회 API가 없음.** `GET /branches`(목록)만 있고, SRCH-02(지점 상세: 주소·영업시간·환율·"Best rate nearby" 배지·지도 프리뷰)를 채울 단건 API가 없음.
  → **추가**: `GET /branches/{id}` — 지점 상세(주소/영업시간/현재 환율/배지 플래그) 조회.

---

## §24 Wireframe Requirements (p.27)

현재 표엔 `Home`, `Exchange Comparison`, `Reservation`, `AI Recommendation`, `My Reservations`, `Staff Confirmation` 6개 행뿐이다. `check_data.md`가 명시하는 아래 4개 화면 중 어느 것도 대응하는 행이 없다.

- [ ] **`Currency Search`(SRCH-03) 행 없음** — 검색창, 최근 검색어(이력 없으면 섹션 자체 숨김), 인기 통화 리스트.
  → **추가 행**: Currency Search — "Search input, recent searches(조건부 숨김), popular currencies list"
- [ ] **`Rate Detail`(SRCH-04) 행 없음** — 현재 "Exchange Comparison" 행이 이 화면과 사실상 겹치는지, 아니면 별개 화면인지 불명확. `check_data.md` 기준으론 SRCH-04에 AI 추천 지점 4곳 리스트 + volatility 배지가 포함되므로 "Exchange Comparison"보다 구체적임.
  → **추가 행 또는 기존 행 세분화**: Rate Detail — "AI-recommended branch list(4), volatility badge, Reserve now 버튼"
- [ ] **`Branch Detail`(SRCH-02) 행 없음** — "Best rate nearby" 배지, 오늘 영업시간(강조 옵션), 지도 프리뷰, Reserve now(지점 프리필).
  → **추가 행**: Branch Detail — 위 요소 + "skips branch selection in reservation flow" 동작 명시.
- [ ] **`Branch Map`(SRCH-01) 행 없음** — 지점 지도 화면 자체가 §24에 없음 (§18.6 "top-scoring branches highlighted on the map"이 유일한 관련 언급).
  → **추가 행**: Branch Map — "지점 핀 표시, 탭 시 Branch Detail 이동".

---

## §8 Reservation FR (p.10) — 참고, Reservation 도메인과 겹치는 부분

- [ ] `check_data.md`: "Reserve now"가 SRCH-04에서는 통화만 프리필, SRCH-02(지점 상세)에서는 **지점까지 프리필해 지점 선택 스텝을 스킵**함 — 즉 예약 플로우 진입 시점에 따라 사전 채워지는 값이 다름. 현재 §8 "Reservation" FR(Input: currency, amount, branch, date/time)은 한 번에 다 받는 단일 트랜잭션처럼 기술되어 있어 **진입 경로별 프리필 상태**를 반영하지 않음.
  → **추가**: §8 Reservation FR 또는 §7 User Flow에 "예약 플로우는 통화만/통화+지점 모두 프리필된 상태로 진입 가능하며, 프리필된 값에 해당하는 단계는 스킵한다"는 각주 추가. (`reservation/wireFrame-feature3.md`의 §2.6·§2.7 미해결 질문과 동일 맥락 — 두 문서를 함께 검토 권장)

---

## 참고 — 이미 확정된 사항 (재작업 불필요)

- `PRD_Feeback_v2.2.md` #1: "통화 먼저 선택 → 지도에 그 통화의 지점별 환율 표시"가 이미 서비스 핵심 가치로 확정됨 — `check_data.md`의 전체 흐름(검색/선택 → 지점 리스트/지도)과 방향이 일치하므로 이 부분은 추가 확인 불필요.
- §18.2의 `is_open_now` 필드는 이미 존재하므로 `check_data.md`의 "오늘 영업시간 강조" 요구사항 자체는 커버됨 (표시 로직만 §24에 명시하면 됨).
