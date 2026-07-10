# PRD v1.3 반영 내역 — Rate Discovery (Domain 2, F2)

> 비교: `TravelX_PRD_v1.2_EN.docx` → `TravelX_PRD_v1.3_EN.docx`
> 근거 문서: `RateDiscovery/check_data.md`, `RateDiscovery/prd_update_checklist.md`

---

## 1. 적용된 변경 (PRD v1.3에 반영됨)

| # | PRD 위치 | 이전 | 이후 | 근거 |
|---|---|---|---|---|
| 1 | §8 FR "Currency Search" | Process: "Query currency list" | "Query currency list; if no input given, return recent searches (per user) and popular currencies (aggregated)" | 원본 §2.1 "최근 검색어/인기 통화 기능이 PRD에 없음" |
| 2 | §18.2 Inputs/Outputs | `is_best_rate_nearby` 필드 없음 | 신규 행 추가: "Boolean flag — true if this branch has the lowest sell rate among nearby branches (independent of total_score)" | 원본 §2.4 "'Best rate nearby' 배지가 §18 스코어와 별개 로직인데 PRD에 없음" |
| 3 | §18.2 `top_n` | "default 10" | "default 10; the Rate Detail screen requests top_n=4" | 원본 §2.5 "top_n 기본값 불일치 (4 vs 10)" — 화면별 예외로 각주 처리 |
| 4 | §9.6 Display and Disclaimer | Now/Wait/Neutral 배지만 정의 | High Volatility 배지 문구 추가: "…exceeds a defined threshold… (exact threshold to be confirmed by the team)" | 원본 §2.3 "High volatility 배지가 PRD에 정의되어 있지 않음" — **임계값은 미정으로 명시적으로 남겨둠** |
| 5 | §14 API Specification | 지점 단건 조회 API 없음 | 신규 행 `GET /branches/{id}` — "Retrieve single branch details (address, hours, rates, badges, map preview)" | 원본 §2.7 |
| 6 | §24 Wireframe Requirements | Currency Search/Rate Detail/Branch Detail/Branch Map 화면 행 없음 | 신규 4행: **Currency Search**, **Rate Detail**, **Branch Detail**, **Branch Map** | 원본 §2.8 |

기존 "Exchange Comparison" 행은 **삭제하지 않고 그대로 유지**했다 — 아래 §3-6 참고.

## 2. 이미 확정되어 재작업 불필요 (원본 §참고 그대로 인용)

- `PRD_Feeback_v2.2.md` #1: "통화 먼저 선택 → 지도에 그 통화의 지점별 환율 표시"는 이미 서비스 핵심 가치로 확정 (PRD §1.3, §6.1에 이미 반영되어 있음, 이번에 추가 변경 없음).
- §18.2의 `is_open_now` 필드는 이미 존재하므로 "오늘 영업시간 강조" 요구사항은 표시 로직만 §24에 명시하면 충분 (§24 Branch Detail 행에 "today highlighted" 문구로 반영함).

## 3. 팀 확인이 필요한 사항 (PRD에 반영하지 않음 — 원본 §5 질문 그대로 이월)

아래는 **의도적으로 PRD 본문을 수정하지 않고 남겨둔** 항목이다. 임의로 결정하면 오히려 문서를 오염시킬 수 있는, 서로 다른 로직/화면 대응이 상충하는 지점이기 때문이다.

1. **SRCH-04 지점 랭킹 로직**: §18.3 가중합 공식(distance/rate/availability/reservation)인가, `check_data.md`가 말하는 moving average/표준편차(§9식)인가? — §18.3 본문은 **그대로 유지**했다. 팀 결정 후 (a) §18.3 디자인 노트 정정 또는 (b) §18.3 공식 자체 개정 중 선택 필요.
2. **"Reserve now"가 스킵하는 "RSV-02"**: F3(`reservation/wireFrame-feature3.md`)의 RSV-02(Time & Currency Selection)와 동일 화면인지, F2/F3/F4 세 문서에서 `RSV-` 접두어가 제각각 다른 의미로 쓰이는 문제의 일부. PRD 본문 자체는 화면 ID 표기를 쓰지 않으므로 PRD엔 영향 없으나, 팀의 wireframe 화면 ID 체계 전체 정리가 필요.
3. **SRCH-02(지점 상세)가 F3의 RSV-01-1(Branch Detail)과 같은 화면인지** — 브라우징용(F2)과 예약 플로우 내(F3)로 분리된 별개 화면인지 미확인.
4. **"인기 통화" 집계 기준** (검색수/예약수, 집계 주기, 실시간 vs 배치) — §8에는 "aggregated"로만 명시해 두었고 구체적 알고리즘은 미정.
5. **SRCH-04 실제 디자인에 §9.6/§18.6 디스클레이머 문구가 실제로 포함되어 있는지** — 디자이너 확인 필요.
6. **§24 "Exchange Comparison" 행을 신규 4행과 어떻게 정리할지** — 이번엔 보수적으로 기존 행을 유지하고 4행을 추가하는 방식을 택했다. 팀이 "Exchange Comparison"을 없애고 4행으로 완전히 대체하길 원하면 후속 수정 필요.
