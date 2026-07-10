# F2: Rate Discovery 플로우 — 와이어프레임 정리 & PRD 반영 제안

작성 목적: 와이어프레임 작업 중 정리된 F2(Rate Discovery) 플로우 내용을 정리하고, `TravelX_PRD_v1.2_EN.pdf` 대비 빠진 부분과 반영 위치를 제안한다. (`reservation/wireFrame-feature3.md`와 동일한 형식)

참고 문서: `MyReservation/prd_update_checklist.md`(F4, 동일 형식), `reservation/wireFrame-feature3.md`(F3), `PRD_Feeback_v2.2.md` #1(환율 비교 재정의 — "통화 먼저 선택 → 그 통화의 지점별 환율을 지도에 표시"는 이미 확정, `check_data.md` 흐름과 방향 일치)

---

## 1. 지금까지 와이어프레임에서 정리된 내용 (F2: Rate Discovery)

| 화면 ID | 역할 | 주요 내용 |
|---|---|---|
| HOME-01 | 전체 사이트 Home | Search bar > "Search a country or currency" → SRCH-03 이동<br>Currency row > "USD" → SRCH-04 이동 |
| SRCH-03 | 통화 검색 | · Recent searches와 popular currencies를 입력 전 기본 노출<br>· Recent searches는 검색 이력 없으면 섹션 자체 숨김<br>· Popular currencies > "USD" → SRCH-04 이동 |
| SRCH-04 | 환율 상세 / AI 추천 지점 | · AI 추천 지점 4곳 리스트 노출<br>· 랭킹 기준: 최근 환율의 moving average·표준편차 (raw rate·distance 아님)<br>· 변동폭이 임계값 초과 시 high volatility 배지 노출<br>· Branch row > "TravelX Myeongdong" → SRCH-02 이동<br>· "Reserve now" → 통화만 프리필된 상태로 예약 플로우 시작 |
| SRCH-02 | 지점 상세 (Branch Detail) | · "Best rate nearby" 배지: 주변 지점 중 최저 매도율일 때만 노출<br>· 오늘 영업시간 강조 표시(옵션)<br>· Map preview → SRCH-01 이동<br>· "Reserve now" → 지점까지 프리필, 예약 플로우에서 지점 선택 단계 스킵 |
| SRCH-01 | 지점 지도 | (SRCH-02의 map preview에서 진입 — 세부 내용 `check_data.md`에 미기술) |

> ※ 위 표는 `check_data.md` 원문을 화면 ID별로 재배열한 것. "역할" 열 중 SRCH-02~04는 원문에 라벨이 없어 본문 내용 기준으로 채워 넣음 — 실제 명칭은 팀 확인 필요.

---

## 2. PRD 대비 빠진 부분

### 2.1 최근 검색어 / 인기 통화 기능이 PRD에 없음
- PRD §8 Functional Requirements(p.10) "Currency Search": Input "currency code or country name" / Process "Query currency list" / Output "Matching currency list"
- → SRCH-03의 "입력 전 recent searches + popular currencies 노출" 기능이 Process/Output 어디에도 없음. 이력 저장용 테이블도 §12에 없음.
- **반영 위치 제안**: §8 Currency Search Process에 "입력 없을 시 최근 검색어(사용자별)+인기 통화(집계 기준) 조회" 추가, §12에 `SearchHistory`(id, userId, currencyId, searchedAt) 신설, "인기 통화" 집계 기준(예: 최근 7일 검색/예약수 상위 N) 정의.

### 2.2 SRCH-04 지점 랭킹 로직이 §18 공식과 다름 (§9 로직과 혼동 가능성)
- PRD §8(p.11)/§18.3(p.23) "AI Exchange Shop Recommendation": score = w1·distance + w2·rate + w3·availability + w4·reservation. §18.1은 "이 기능은 §9(타이밍 추천)와 독립적이며 상점을 랭킹하지 타이밍을 다루지 않는다"고 명시.
- → `check_data.md`는 SRCH-04 지점 랭킹이 "moving average와 표준편차(recent rates)" 기준이라고 명시 — 이는 §9(타이밍)의 변동성 로직이지 §18 공식이 아님. 둘 중 어느 게 실제 의도인지 문서 간 불일치.
- **반영 위치 제안**: 팀 확인 후 (a) §18 공식이 맞다면 디자인 노트 정정, (b) moving average/표준편차가 맞다면 §18.3 공식 개정 또는 제3의 로직으로 §18에 명시. (b)인 경우 §12에 지점별 환율 이력 테이블(`BranchExchangeRateHistory`: branchId, currencyId, rate, recordedAt) 신설 필요 — 현재 `ExchangeRateHistory`는 `branchId`가 없어 통화 전체 이력만 저장.

### 2.3 High volatility 배지가 PRD에 정의되어 있지 않음
- PRD §9.6(p.13)은 Now/Wait/Neutral 배지만 정의, §18.6(p.24)도 변동성 배지 언급 없음.
- → SRCH-04의 "swing threshold 초과 시 volatility 배지"에 대응하는 요구사항이 없음.
- **반영 위치 제안**: §18.2(Inputs/Outputs 표) 또는 §9에 volatility 배지·threshold 값 정의 추가.

### 2.4 "Best rate nearby" 배지가 §18 스코어와 별개 로직인데 PRD에 없음
- PRD §18.2(p.23) Output: `total_score`, `breakdown`, `is_open_now`, `reservation_available`.
- → SRCH-02의 "주변 최저 매도율일 때만 노출"은 가중합산 `total_score`와 무관한 순수 최저가 판정 — 대응 필드 없음.
- **반영 위치 제안**: §18.2에 `is_best_rate_nearby`(boolean) 필드 추가.

### 2.5 top_n 기본값 불일치 (4 vs 10)
- PRD §18.2(p.23): `top_n` 기본값 10.
- → SRCH-04는 정확히 "4개" 고정 노출.
- **반영 위치 제안**: §14 API 문서에 SRCH-04 호출 시 `top_n=4` 예시 명시, 또는 화면별 기본값 차이를 각주로 정리.

### 2.6 디스클레이머 노출 여부 확인 필요
- PRD §9.6/§18.6: "not investment advice" 디스클레이머 필수 표시 규정.
- → `check_data.md` SRCH-04 노트엔 디스클레이머 언급 없음(노트가 축약형이라 누락일 가능성).
- **반영 위치 제안**: 디자인 확인 후 없으면 §24에 필수 요소로 명시.

### 2.7 지점 단건 상세 조회 API 없음
- PRD §14(p.19): `GET /branches`(목록)만 존재.
- → SRCH-02(주소·영업시간·환율·배지·지도 프리뷰)를 채울 단건 조회 API가 없음.
- **반영 위치 제안**: `GET /branches/{id}` 신설.

### 2.8 §24 와이어프레임 요구사항에 4개 화면 행이 통째로 없음
- PRD §24(p.27): Home / Exchange Comparison / Reservation / AI Recommendation / My Reservations / Staff Confirmation 6개 행뿐.
- → Currency Search(SRCH-03), Rate Detail(SRCH-04), Branch Detail(SRCH-02), Branch Map(SRCH-01) 4개 화면에 대응하는 행이 없음. 기존 "Exchange Comparison" 행이 이 중 일부와 겹치는지도 불명확.
- **반영 위치 제안**: 4개 행 추가 또는 기존 "Exchange Comparison" 행을 세분화. (§5 질문 참고)

### 2.9 예약 플로우 진입 시 프리필 상태가 §8 Reservation FR에 없음
- PRD §8(p.10) "Reservation" FR: Input "currency, amount, branch, date/time" — 단일 트랜잭션처럼 기술.
- → SRCH-04(통화만 프리필)와 SRCH-02(지점까지 프리필, 지점선택 스킵)처럼 진입 경로별로 프리필 상태가 다름. `reservation/wireFrame-feature3.md` §2.6·§2.7의 미해결 질문과 같은 맥락.
- **반영 위치 제안**: §8 Reservation FR 또는 §7 User Flow에 "예약 플로우는 통화만/통화+지점 모두 프리필된 상태로 진입 가능하며, 프리필값에 해당하는 단계는 스킵" 각주 추가.

---

## 3. 화면 ID 충돌 현황 (해결은 보류 — §5 참고)

`check_data.md` 마지막 줄: `"Reserve now" → Carries this branch forward and skips branch selection in RSV-02` — 이 "RSV-02"가 어느 문서 기준인지 불명확하다. 이미 존재하는 두 RSV 체계와 대조하면 전부 다르다.

| 문서 | RSV-01 | RSV-02 | RSV-03 | RSV-04 |
|---|---|---|---|---|
| `MyReservation/check_data.md` | My Reservations(목록) | Reservation Complete | Reservation Detail | Exchange History |
| `reservation/wireFrame-feature3.md`(F3) | Reservation Home(지도+AI추천) | Time & Currency Selection | Reservation Detail/QR·Cancel | My Reservations(목록) |
| `RateDiscovery/check_data.md`이 암시 | (불명) | **지점 선택**이 일어나는 화면 | — | — |

F3 문서 구조상 지점 선택은 RSV-01(지도) 또는 제안 단계인 RSV-01-1(Branch Detail)에서 일어나고, F3의 RSV-02는 "시간·통화 선택"이지 "지점 선택"이 아니다. 즉 이 문서의 설명과 F3의 실제 정의가 어긋난다 — **결론은 내지 않고 §5로 넘긴다.**

이 문서(F2) 내부에서 쓰는 `SRCH-` 접두어 자체는 다른 도메인과 충돌하지 않으므로 유지 가능. 문제는 F2 문서가 F3의 화면을 "RSV-02"로 지칭하는 부분뿐.

---

## 4. PRD 문서 반영 제안 위치

| 반영 내용 | PRD 위치 |
|---|---|
| 최근 검색어/인기 통화 기능·데이터 | §8 Currency Search, §12(SearchHistory), §14 |
| SRCH-04 지점 랭킹 로직 정정/개정 | §18.3, (조건부) §12 BranchExchangeRateHistory |
| High volatility 배지 정의 | §9 또는 §18.2 |
| "Best rate nearby" 필드 | §18.2 |
| top_n=4 예시/각주 | §14 |
| 디스클레이머 확인 | §24 |
| 지점 단건 조회 API | §14 (`GET /branches/{id}`) |
| Currency Search/Rate Detail/Branch Detail/Branch Map 행 추가 | §24 |
| 예약 플로우 프리필 상태 각주 | §8 Reservation FR, §7 User Flow |
| 화면 ID 체계 재정리 | §24 (팀 확인 후) |

---

## 5. 확인이 필요한 질문 (팀/멘토 논의용)

1. SRCH-04의 "4 AI-recommended branches" 랭킹은 §18(distance/rate/availability/reservation) 공식인가, moving average/표준편차(§9식) 기반인가?
2. "Reserve now"가 스킵한다는 "RSV-02"는 `reservation/wireFrame-feature3.md`의 RSV-02(Time & Currency Selection)와 같은 화면인가? F3 쪽 "지점 선택"은 RSV-01/RSV-01-1에서 일어나는 것으로 보이는데, 이 문서(F2)가 말하는 "RSV-02의 지점 선택"은 어느 화면을 가리키는가?
3. SRCH-02(지점 상세)는 F3의 RSV-01-1(Branch Detail, 아직 제안 단계)과 같은 화면인가, 아니면 브라우징용(F2)과 예약플로우 내(F3)로 분리된 별개 화면인가?
4. "인기 통화"의 집계 기준(검색수/예약수, 집계 주기, 실시간 vs 배치)은?
5. SRCH-04 실제 디자인에 §9.6/§18.6 디스클레이머 문구가 포함되어 있는가?
6. §24의 "Exchange Comparison" 행을 Currency Search/Rate Detail/Branch Detail/Branch Map으로 세분화할지, 별도 행을 추가할지?

---

## 참고 — 이미 확정된 사항 (재작업 불필요)

- `PRD_Feeback_v2.2.md` #1: "통화 먼저 선택 → 지도에 그 통화의 지점별 환율 표시"가 이미 서비스 핵심 가치로 확정됨 — `check_data.md`의 전체 흐름(검색/선택 → 지점 리스트/지도)과 방향이 일치하므로 이 부분은 추가 확인 불필요.
- §18.2의 `is_open_now` 필드는 이미 존재하므로 `check_data.md`의 "오늘 영업시간 강조" 요구사항 자체는 커버됨 (표시 로직만 §24에 명시하면 됨).
