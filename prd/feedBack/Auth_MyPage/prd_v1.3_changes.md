# PRD v1.3 반영 내역 — Auth & My Page/Notification (Domain 1 + 5)

> 비교: `TravelX_PRD_v1.2_EN.docx` → `TravelX_PRD_v1.3_EN.docx`
> 근거 문서: `Auth_MyPage/TravelX_PRD_wireframe_Auth-MyPage_Domain1-5.md`
> 이 도메인은 대부분 "PRD에 아예 없던 항목을 추가"하는 성격이라 확정/미확정 구분보다 **적용(safe-additive)** 위주다.

---

## 1. 적용된 변경 (PRD v1.3에 반영됨)

| # | PRD 위치 | 이전 | 이후 | 근거 |
|---|---|---|---|---|
| 1 | §6.1 User Features | "Profile" 서브섹션 자체가 없었음 (프로필 조회 기능 누락) | **Profile** 서브섹션 신설 + bullet "View profile (name, email — read-only, sourced from Google account)" | 원본 §7.2 "PRD에 프로필 조회 항목 자체가 없음" 지적 반영 |
| 2 | §6.1 Notifications | "Reservation completed/cancelled/pre-pickup reminder notification" 3개 — 알림 **생성(수신)** 만 정의, 조회/읽음처리 없음 | bullet 2개 추가: "View notification list", "Mark notifications as read" | 원본 §7.2 "Notifications 행은 생성 트리거만 정의, 조회/읽음처리 없음" 지적 반영 |
| 3 | §14 API Specification | 알림/프로필 조회 API 없음 | 신규 3행: `GET /notifications`, `PATCH /notifications/read`, `GET /users/me` | 위 1, 2번 기능을 뒷받침하는 API |
| 4 | §24 Wireframe Requirements | My Page / Notification 화면 행 없음 | 신규 행 **"My Page / Notifications"** — "Profile card (read-only), menu links (Notifications, My Reservation, Exchange history, Log out); notification list with mark-all-as-read and empty state" | 원본 §7.2 "My Page/Notification 화면 행 없음" 지적 반영 |

## 2. 적용하지 않은 것 (의도적으로 보류)

- **AUTH-01/02/03 화면 단위 세분화** (로그인 화면 상세, OAuth 리다이렉트 로딩, 로그인 실패 화면): 원본 문서 자체가 "PRD 문구는 유지, 화면 스펙만 구체화"라고 명시했으므로 **PRD 본문 텍스트 변경 없음**. §6.1 Authentication의 "Sign in with Google / Log out" 두 줄은 그대로 둠. 필요하면 `Auth_MyPage/` 와이어프레임 문서가 화면 단위 상세 스펙을 이미 보유하고 있음.
- **MYPAGE-01의 "My Reservation"/"Exchange history" 딥링크**: My Reservations 도메인(F4) 쪽 변경사항과 중복되므로 이번 반영은 MyReservation 도메인 문서(`MyReservation/prd_v1.3_changes.md`)에서 다룸.

## 3. 팀 확인이 필요한 사항 (PRD에 반영하지 않고 원문 그대로 둠)

이 도메인 원본 문서(`TravelX_PRD_wireframe_Auth-MyPage_Domain1-5.md`)에는 별도의 "확인 필요 질문" 섹션이 없었음 — 발견된 갭이 대부분 순수 추가(additive)라 결정이 필요한 항목이 없었다. 다만 아래는 이번 반영 과정에서 새로 발견된 확인 포인트:

1. `GET /users/me`는 이번에 새로 추가한 API인데, 로그인(Sign in with Google) 응답에 이미 프로필 정보가 포함되는지(§8 FR "Sign In with Google" Output: "User account created/retrieved, session started") 확인 후 — 세션에서 재조회가 필요 없다면 이 엔드포인트를 제거하고 클라이언트 캐시로 대체할 수도 있음.
2. 알림 읽음 처리 API를 `PATCH /notifications/read`(전체/단건 겸용)로 단일화했는데, "Mark all as read"(전체)와 개별 항목 읽음 처리를 분리된 엔드포인트로 원하는지 팀 확인 필요.
