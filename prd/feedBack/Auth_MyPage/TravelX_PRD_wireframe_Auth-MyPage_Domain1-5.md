# TravelX — 인증 & 마이페이지·알림 화면 설계 문서 (PRD)

## 1. 개요
사용자가 앱에 진입하기 위해 구글 계정으로 로그인하고, 자신의 프로필 정보를 확인하며, 예약/픽업 관련 알림을 조회하는 플로우.
디자인 기준 파일: *(Figma 파일명 입력)* / 정적 export는 `figma-export/` 폴더 참고


## 2. 데이터 모델

### User (사용자)
| 필드 | 설명 |
|---|---|
| id | PK |
| name | 이름 (구글 프로필에서 가져옴) |
| email | 이메일 (구글 프로필에서 가져옴) |
| googleId | 구글 계정 고유 ID (OAuth2 "sub" claim) |
| role | USER / ADMIN |

### Notification (알림 1건)
| 필드 | 설명 |
|---|---|
| id | 알림 ID |
| type | reservation_completed / reservation_cancelled / pickup_reminder |
| message | 알림 본문 |
| isRead | 읽음 여부 |
| createdAt | 생성 시각 |

## 3. 화면 목록 (Screen ID 체계)

### AUTH-01 — Login (로그인)
- 진입: 앱 최초 실행 시 진입하는 첫 화면 (미로그인 상태의 최상위 화면)
- 헤더 없음 (뒤로가기 없음, 진입점)
- 로고 + 앱 타이틀("TravelX") + 서비스 한 줄 소개
- "Sign in with Google" 버튼 (단일 로그인 수단)
- 하단 안내문: 로그인 시 이용약관/개인정보처리방침 동의로 간주된다는 문구
- **버튼 동작**
  - "Sign in with Google" 탭 → AUTH-02 (OAuth Redirect)로 이동

> 별도의 "회원가입" / "비밀번호 찾기" 화면은 없음. 구글 OAuth2 특성상 최초 로그인 시 계정이 자동 생성되고, 로컬 비밀번호를 사용하지 않으므로 비밀번호 재설정 플로우가 존재하지 않음.

### AUTH-02 — OAuth Redirect / Loading (인증 처리 중)
- 진입: AUTH-01에서 구글 로그인 버튼 탭 직후
- 전체 화면 로딩 스피너 + "Signing you in…" 텍스트
- 실제로는 시스템 브라우저/웹뷰로 구글 인증 화면이 뜨고, 인증 완료 후 콜백으로 복귀하는 구간을 앱 내에서 표현한 화면
- **분기**
  - 인증 성공 → RSV-01 (My Reservations, 예약 도메인의 앱 첫 진입 화면)로 이동
  - 인증 실패/취소 → AUTH-03 (Login Error)로 이동

### AUTH-03 — Login Error (로그인 실패)
- 진입: AUTH-02에서 인증 실패 또는 사용자가 구글 인증 과정에서 취소한 경우
- 경고 아이콘(연레드 배경) + "Sign-in failed" + 안내문 ("다시 시도해 주세요" 등)
- **버튼**
  - "Try again" → AUTH-01로 복귀

---

### MYPAGE-01 — My Page (마이페이지 / 프로필)
- 진입: 하단 탭바 "Profile" 탭
- 헤더: "TravelX" (뒤로가기 없음, 탭 최상위 화면)
- 프로필 카드: 이름, 이메일 (구글 프로필 기반, 읽기 전용)
- 메뉴 리스트
  - "Notifications" → 우측에 안 읽은 알림 개수 배지
  - "Exchange history" → RSV-04 (예약 도메인의 Exchange History)로 딥링크
  - "Log out"
- 하단 탭바: Home / Maps / Exchange / Profile(active)
- **동작**
  - "Notifications" 탭 → MYPAGE-02 (Notification List)로 이동
  - "Log out" 탭 → MYPAGE-01-POPUP (로그아웃 확인 팝업) 노출

### MYPAGE-01-POPUP — Logout Confirm (로그아웃 확인 팝업)
- MYPAGE-01 위에 뜨는 모달 (반투명 딤 배경)
- "Log out of TravelX?" + 안내 문구
- **버튼**
  - "Cancel" → 팝업 닫고 MYPAGE-01 유지
  - "Log out" → 세션/JWT 무효화 후 AUTH-01 (Login)로 이동

### MYPAGE-02 — Notification List (알림함)
- 헤더: 뒤로가기(← MYPAGE-01) + "Notifications"
- 상단: "Mark all as read" 링크 (우측)
- 리스트 카드: 타입 아이콘/컬러, 메시지 요약, 발생 시각, 안 읽음 표시(점)
- 빈 상태: 알림이 없을 때 안내 일러스트 + "No notifications yet" 문구
- **동작**
  - 카드 탭 → 해당 알림 isRead 처리 + 관련 예약이 있으면 RSV-03 (Reservation Detail)로 이동
  - "Mark all as read" 탭 → 전체 isRead 처리 (화면 전환 없음, 인라인 상태만 갱신)

## 4. 화면 전환 다이어그램 (텍스트)
```
AUTH-01 (로그인)
 └─ Sign in with Google ───────────▶ AUTH-02

AUTH-02 (인증 처리 중)
 ├─ 성공 ──────────────────────────▶ RSV-01 (My Reservations, 앱 첫 진입)
 └─ 실패/취소 ─────────────────────▶ AUTH-03

AUTH-03 (로그인 실패)
 └─ Try again ─────────────────────▶ AUTH-01

MYPAGE-01 (마이페이지)
 ├─ Notifications ─────────────────▶ MYPAGE-02
 ├─ Exchange history ──────────────▶ RSV-04 (예약 도메인)
 └─ Log out ───────────────────────▶ MYPAGE-01-POPUP

MYPAGE-01-POPUP (로그아웃 확인)
 ├─ Cancel ────────────────────────▶ MYPAGE-01로 복귀
 └─ Log out ───────────────────────▶ AUTH-01

MYPAGE-02 (알림함)
 ├─ 카드 탭 ────────────────────────▶ RSV-03 (관련 예약이 있는 경우)
 └─ Mark all as read ──────────────▶ 리스트 내 상태만 갱신 (전환 없음)
```


## 7. PRD 반영 

### 7.1 인증 (AUTH) 관련

| PRD 위치 | 기존 내용 | 이번 화면설계 반영/수정 사항 |
|---|---|---|
| §6.1 User Features > Authentication | "Sign in with Google", "Log out" 두 줄만 존재 | AUTH-01(로그인 화면 상세: 로고/소개/버튼/약관 안내), AUTH-02(OAuth 리다이렉트 로딩), AUTH-03(로그인 실패 화면)으로 **화면 단위 세분화**. PRD 문구 자체는 유지, 화면 스펙만 구체화 |
| §8 Functional Requirements — `Sign In with Google` 행 | Input: Google OAuth2 ID token / Output: 계정 생성·조회, 세션 시작 | 이 흐름이 AUTH-01→AUTH-02(성공 시 RSV-01)로 매핑됨을 확인. |
| §24 Wireframe Requirements 표 | Auth 관련 화면 행 없음 |

### 7.2 마이페이지 & 알림 (MYPAGE) 관련

| PRD 위치 | 기존 내용 | 이번 화면설계 반영/수정 사항 |
|---|---|---|
| §6.1 User Features | 프로필 조회 항목 자체가 없음 | **신규 추가 필요**: "프로필 조회" 항목. MYPAGE-01 화면 스펙(이름/이메일 읽기전용 카드)으로 구체화 |
| §8 Functional Requirements | `Notifications` 행은 알림 "생성" 트리거만 정의, 조회/읽음처리 없음 | MYPAGE-02(알림함 리스트, 개별 읽음 처리, 전체 읽음 처리) 스펙 추가 → PRD에 `Notification List/Read` 행 **신규 추가 필요** |
| §24 Wireframe Requirements 표 | My Page/Notification 화면 행 없음 | **추가 필요**: My Page / Notification List 2행 |

