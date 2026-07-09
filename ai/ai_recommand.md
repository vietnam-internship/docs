# 환전소 위치 추천 시스템

## 1. 개요
환전소 추천 기능은 위치·환율·수수료·영업시간·예약 가능 여부를 종합적으로 평가하는 점수 기반 추천 모델이다. 사용자가 지도에서 주변 환전소를 확인할 때 조건이 좋은 환전소를 우선적으로 보여주며, 최종 환전소 결정 여부는 사용자가 판단한다.

## 2. 기능 요구사항

### 2.1 입력
| 항목 | 설명 | 필수 여부 |
|---|---|---|
| latitude, longitude | 사용자 현재 위치 | 필수 |
| currency | 환전 희망 통화 | 필수 |
| radius_km | 검색 반경 (기본값 5km) | 선택 |
| top_n | 반환할 최대 환전소 수 (기본값 10) | 선택 |

### 2.2 출력
정렬된 환전소 리스트를 반환하며, 각 항목은 다음을 포함한다.

| 항목 | 설명 |
|---|---|
| office_id, name, latitude, longitude | 환전소 기본 정보 |
| total_score | 최종 추천 점수 (0~1) |
| breakdown | 변수별 기여도 (distance_score, rate_score, availability_score, reservation_score) |
| is_open_now, reservation_available | 상태 정보 (UI 배지 표시용) |

### 2.3 정렬 및 표시 기준
- total_score 내림차순 정렬
- 동점 처리: distance_score 우선순위로 2차 정렬
- 지도 마커에는 상위 스코어 환전소에 하이라이트/뱃지 표시

## 3. 알고리즘 방식

**채택 방식: Rule-based Scoring**

score = w1⋅distance_score + w2⋅fee_score + w3⋅availability_score + w4⋅reservation_score

| 변수 | 정의 
|---|---|
| distance_score | 사용자-환전소 간 거리 | 
| fee_score | 수수료율 (%) — 낮을수록 유리 
| availability_score | 영업 여부 및 마감 임박도 
| reservation_score | 예약 가능 여부

**초기 가중치(예시):** w1=0.35, w2=0.35, w3=0.20, w4=0.10
가중치는 서비스 정책에 따라 조정 가능한 설정값으로 관리

## 4. 설계 근거 (Appendix)

MVP 단계에서 Rule-based Scoring을 채택한 이유:

| 방식 | 미채택 사유 |
|---|---|
| Learning-to-Rank | 모델 해석이 어렵고 랭킹 품질이 로그 데이터 양·질에 크게 의존 |
| 추천 시스템(협업 필터링 등) | 개인화 수준은 높으나 모델 복잡도가 크고, 실시간 위치 필터링과 결합 시 추가 설계 필요 |

**Rule-based Scoring** : 변수 간 관계가 단순하고 방향성이 명확해 가중합만으로 합리적 순위 산출 가능. 변수별 기여도 분리 제공으로 설명가능성 확보

향후 사용자 선택 로그 축적 시 Learning-to-Rank 기반 개인화 랭킹으로 고도화 검토.