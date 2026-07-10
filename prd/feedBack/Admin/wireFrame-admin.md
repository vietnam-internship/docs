# F4: Admin 기능 — 와이어프레임 정리 & PRD 반영 제안
**담당**: 김두현 (기획/Admin 담당) · **상태**: 조사 완료 (초안)

---

## 1. 지금까지 와이어프레임에서 정리된 내용 (F4: Admin)

| 화면 ID | 역할 | 주요 내용 |
|---|---|---|
| ADM-1 | Admin Login | 사전 배포된 지점용 계정으로 로그인 (회원가입 없음) |
| ADM-2 | Dashboard page | 오늘 누적 예약 수, 대기 예약 건수, 예약 순위 및 최근 확정된 예약 목록 대시보드 조회 |
| ADM-03 | Exchange Rates page | 지점의 환율 우대율(Fee) 및 살 때/팔 때 환율 편집 및 신규 통화 쌍 추가 |
| ADM-04 | Inventory page | 환전소별 통화 재고 조회, 재고 조정(Adjust stock)을 통한 수량 설정 |
| ADM-05 | Reservations page | 예약 내역 검색/필터링(ALL, Completed, Cancelled, Pending) 및 상세 조회 페이지네이션 |
| ADM-6-1 | QR Scan - Checking page | 카메라 스캔 영역 제공, QR 코드를 대어 예약 건 확인 단계로 진입 |
| ADM-6-2 | QR Scan - Confirmation page | 스캔된 예약의 상세 정보(금액, 장소, 시간 등) 확인 및 완료(Complete)/거절(Reject) 처리 버튼 노출 |
| ADM-6-3 | QR Scan - Reservation Complete page | 최종 완료 상태(그린 체크) 및 거래 번호가 요약된 완료 화면 노출 |
| ADM-6-4 | QR Scan - Reservation Rejected page | 거절 상태(레드 크로스) 및 거절 안내 문구 요약 화면 노출 |
| ADM-07-01 | Settings page for the first time | 최초 로그인 시 지점의 기본 정보(이름, 좌표, 영업시간, 시간당 예약 한도) 필수 등록 화면 강제 리다이렉트 |
| ADM-07-02 | Settings page | 설정 탭을 통해 상점 기본 정보 수정 및 저장 |

---

## 2. PRD 대비 빠진 부분

### 2.1 관리자 전용 독립 로그인 체계 및 계정 정책 누락
- **내용**: PRD §10 Security 및 §14 API에는 구글 OAuth2 인증 방식만 명시되어 있으나, 사설 환전소 지점 관리자는 회원가입 절차 없이 **사전에 시스템에서 배포한 전용 로컬 계정**을 통해 로그인해야 함.
- **반영 위치 제안**: 
  - `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L210-L223)` §10 Security 섹션 및 `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L403-L413)` §14 API Specification에 "지점 관리자 전용 ID/PW 기반 인증 체계 및 API" 명시 추가.

### 2.2 시스템 총괄 관리자(System Admin)와 지점 관리자(Branch Admin)의 권한 및 역할 분리 부재
- **내용**: PRD §6.2 Admin Features에서는 플랫폼 전체 관리와 개별 환전소 지점 관리가 혼재되어 있으나, 와이어프레임은 **개별 지점 관리자(Branch Admin)** 관점으로 설계됨. 플랫폼 총괄 관리자와 지점 직원 간의 데이터 접근 권한 분리(RBAC)가 필요함.
- **반영 위치 제안**: 
  - `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L86-L92)` §6.2 및 `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L244-L245)` §12 DB Design `User` 테이블의 `role` 컬럼에 `SYSTEM_ADMIN`과 `BRANCH_ADMIN` 역할 세분화 추가.

### 2.3 최초 로그인 시 상점 기본정보 등록(ADM-07-01) 강제 프로세스 누락
- **내용**: 지점 계정으로 최초 로그인(First-time Login) 시, 상점 활성화를 위해 지점 정보(좌표, 운영 시간 등)를 필수 입력해야 하는 리다이렉트 흐름 및 화면 정의가 누락됨.
- **반영 위치 제안**: 
  - `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L97-L100)` §7.1 Userflow Admin에 최초 진입점 리다이렉트 룰 추가 및 `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L146-L149)` §8 Branch Management (Admin)에 조건부 프로세스 명시.

### 2.4 QR 코드 기반 예약 처리(Complete/Reject) 및 거절 상태(Rejected)의 라이프사이클 부재
- **내용**: PRD §24 에서는 단순한 "버튼 클릭식 스태프 컨펌"만 정의되어 있으나, 와이어프레임에서는 모바일/웹 카메라를 활용한 QR 스캔 화면(ADM-6-1) 및 그에 따른 완료/거절(Reject) 처리와 거절 알림 발송 플로우가 구체화됨. 특히 거절(Reject) 처리에 따른 데이터베이스 상태 변화 및 유저 단 안내 명세가 없음.
- **반영 위치 제안**: 
  - `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L325-L328)` §12 ReservationStatus Enum에 `REJECTED` 상태 추가.
  - `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L387-L393)` §14 API Specification에 `POST /admin/reservations/{id}/complete` 및 `POST /admin/reservations/{id}/reject` API 세분화.

### 2.5 대시보드 통계 및 실시간 모니터링 데이터 항목 구체화 미흡
- **내용**: PRD §8 에서는 대시보드가 단순 통계 조회로만 정의되어 있으나, 와이어프레임(ADM-2)에서는 당일 총 활성 고객, 대기 중인 예약 건수, 예약 건수 기반 실시간 통화 랭킹, 최근 승인된 예약 리스트 등의 즉시 대응용 정보 제공이 추가됨.
- **반영 위치 제안**: 
  - `→ [TravelX_PRD_v1.2_EN.docx](file:///Users/doohyun/VietnamInternship/docs/prd/TravelX_PRD_v1.2_EN.docx#L162-L165)` §8 Admin Dashboard 기능 처리 요건 및 §24의 Admin Dashboard 와이어프레임 요구사항에 실시간 핵심 지표 명시.

### 2.6 설정 페이지의 저장 버튼 오기(Adjust Stock) 수정 제안
- **내용**: 와이어프레임 ADM-07-01 및 ADM-07-02에서 설정 저장용 하단 버튼 명이 "Adjust stock"으로 표기되어 상점 정보 설정 수정의 문맥과 불일치함.
- **반영 위치 제안**: 
  - UI 상 "Save settings" 또는 "Save changes"로 명칭을 변경하도록 와이어프레임 설계 수정 및 PRD 반영 시 기재 명칭 정정 제안.

---

## 3. 종합 제안 — ADM 화면 ID 재정리 (초안)

```
ADM-01      Admin Login (지점 전용 계정 로그인)
ADM-02      Dashboard (실시간 대기 예약 및 당일 통계 모니터링)
ADM-03      Exchange Rates (지점별 살 때/팔 때 환율 및 수수료 설정)
ADM-04      Inventory (지점 통화별 모바일 전용 재고 실시간 조정)
ADM-05      Reservations (예약 내역 필터링 검색 및 상세 정보 확인)
ADM-06-1    QR Scan - Checking (카메라 기반 고객 QR 코드 리딩)
ADM-06-2    QR Scan - Confirmation (스캔 완료 후 상세 대조 및 승인/거절 판단)
ADM-06-3    QR Scan - Complete (트랜잭션 완료 처리 완료 안내)
ADM-06-4    QR Scan - Rejected (트랜잭션 거절 처리 완료 안내)
ADM-07-01   Settings - First Time (최초 로그인 시 상점 기본정보 등록 강제 페이지)
ADM-07-02   Settings - Modify (상점 영업 시간 및 슬롯 예약 제한 등 변경 설정)
```

---

## 4. PRD 문서 반영 제안 위치

| 반영 내용 | PRD 위치 |
|---|---|
| 지점 전용 계정 로그인 및 API | §8 Sign In (Admin 용도 추가), §10 Security, §14 API 명세 갱신 |
| System Admin vs Branch Admin 권한 롤 분리 | §6.2 Admin Features 기능 영역 분할 및 §12 DB User Table `role` enum 값 추가 |
| 최초 로그인 강제 리다이렉트 흐름 | §7.1 Userflow Admin 플로우 차트 및 §8 Branch Management 처리 조건에 명시 |
| QR 코드 기반 예약 승인/거절 플로우 및 REJECTED 상태 | §8 Identity Verification, §12 ReservationStatus Enum, §14 API 명세 추가 |
| 대시보드 내 실시간 데이터 항목 목록 구체화 | §8 Admin Dashboard, §24 Wireframe Requirements |
| 설정 페이지 버튼 텍스트 정정 및 저장 API 연동 | §24 Staff Confirmation/Settings, §14 API 명세 반영 |

---

## 5. 확인이 필요한 질문 (팀/멘토 논의용)

1. 플랫폼 전체를 관리하는 총괄 어드민(System Admin)을 위한 별도의 화면 플로우(예: 회원 관리, 지점 입점 승인 등)를 이번 스프린트 내에 추가 개발해야 하는가?
2. 최초 로그인 시 지점 정보 입력 화면(ADM-07-01)에서 입력을 누락하거나 취소하고 다른 탭으로 이동하려 할 때, 이동을 전면 차단(Lock)하고 강제해야 하는가?
3. QR 코드 스캔 거절(Reject) 처리 시, 유저에게 발송되는 알림 외에 해당 유저의 노쇼(No-Show) 이력 페널티 점수에 반영하는 룰을 적용해야 하는가?
4. 지점 설정의 "Available Reservation per One Time Slot"(타임슬롯당 예약 가능 수량) 제한이 초과되었을 때, 유저 화면단에서는 어떤 에러 메시지를 띄워야 하는가?

---

## 관련 문서
- [admin_descriptions.md](./admin_descriptions.md)
- [wireFrame-feature3.md](../reservation/wireFrame-feature3.md)
