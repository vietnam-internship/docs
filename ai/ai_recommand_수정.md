# 환전소 추천 알고리즘(수정) — 가중치 최적화 방안

## 1. 목적 (Purpose)

현재 Rule-based Scoring 방식으로 운영 중인 환전소 추천 알고리즘의 가중치(w1~w4)를, 사용자 클릭률/전환율 데이터를 기반으로 데이터 기반(data-driven) 방식으로 최적화하기 위한 방안을 정의한다.

## 2. 배경 (Background)

### 2-1. 현재 구조

```
score = w1·distance_score + w2·fee_score + w3·availability_score + w4·reservation_score
```

| 변수 | 정의 |
|---|---|
| distance_score | 사용자-환전소 간 거리 |
| fee_score | 수수료율 (%) — 낮을수록 유리 |
| availability_score | 영업 여부 및 마감 임박도 |
| reservation_score | 예약 가능 여부 |

**초기 가중치(예시):** w1=0.35, w2=0.35, w3=0.20, w4=0.10
가중치는 서비스 정책에 따라 조정 가능한 설정값으로 관리.

### 2-2. 문제 정의

초기 가중치는 임의로 설정된 값이며, 실제 사용자 행동(클릭/전환) 데이터가 축적되면 이를 반영해 가중치를 지속적으로 개선할 필요가 있다.

## 3. 상세 내용 (Details)

### 3-1. 채택 방식: Logistic Regression (클릭 예측 → 가중치 역산)

score 공식이 이미 `w1·x1 + w2·x2 + w3·x3 + w4·x4` 형태이므로, 클릭 여부(0/1)를 라벨로 하고 4개 sub-score를 feature로 하는 로지스틱 회귀를 학습하면 계수가 곧 w1~w4가 된다.


**장점**
- 기존 rule-based 구조를 그대로 유지 → 설명 가능성(explainability) 유지
- 데이터가 적어도(수백~수천 건) 안정적으로 수렴
- 온라인 서비스 적용 후 배치로 주기적 재학습 가능 (예: Spring Scheduler로 일/주 단위 재학습 트리거)

## 4. 고려사항 (Considerations)

- **Selection bias / Position bias**: 사용자가 애초에 상위 노출된 환전소만 볼 확률이 높으므로, 노출 순서를 feature에 추가하거나 IPS(Inverse Propensity Scoring) 보정을 고려해야 한다.
- **전환율 반영**: 클릭이 아닌 실제 전환(예약/이용)까지 반영하려면 label을 전환 여부로 바꾸거나, 클릭 모델과 전환 모델을 곱하는 2단계 모델 구조를 검토한다.
- **단계적 도입**: MVP 단계에서는 복잡도가 낮은 로지스틱 회귀부터 도입하고, 데이터 축적 및 서비스 성숙도에 따라 Learning to Rank, Contextual Bandit 순으로 확장을 검토한다.

## 5. 결론 (Recommendation)

MVP 단계 권장 조합:

1. **로지스틱 회귀로 시작**
2. **학습된 계수를 w1~w4로 정규화**
3. **배치 재학습 (Spring Scheduler로 주기적 트리거)**
