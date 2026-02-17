# Engineering Notes (Public)

이 문서는 “전략 레시피”가 아니라, 시스템을 운영하며 마주친 **엔지니어링 이슈와 해결 방식**을 정리합니다.

---

## 1) Backtest correctness: timing alignment (lookahead risk)
### Issue
백테스트에서 진입 시점(T)과 동일한 시점의 상위 타임프레임 정보를 사용하면,
“진입 당시에는 알 수 없는 정보(동일 봉 종가 등)”가 섞여 **lookahead 위험**이 생길 수 있습니다.

### Root cause (conceptual)
- 신호 계산에 쓰인 상위 타임프레임 row의 timestamp가 entry_time과 동일하게 매핑되는 구조
- 진입 가격이 “진입 시점에 관측 가능한 값”이 아니라, 동일 봉의 집계값(예: 종가)로 설정될 가능성

### Fix
- **Signal time**: T-1 row에서 신호/확률 생성
- **Entry time**: T (다음 구간 시작)에 진입
- **Entry price**: T 시점에 관측 가능한 값(open) 사용
- Probe/Assertion으로 `feature_row_ts < entry_time`을 검증

### Verification
- 로그 probe로 신호 시점과 진입 시점의 strict ordering 확인
- 수정 전/후 결과 비교(성능 수치 변화는 “정합성 반영” 결과로 해석)

---

## 2) Operational reliability: data window / indicator warmup
### Issue
라이브 운영 중 데이터 윈도우가 짧으면 지표 계산이 불안정하거나 NaN이 발생하여,
“신호 없음” 또는 잘못된 입력으로 이어질 수 있음.

### Fix
- 충분한 히스토리 윈도우 확보(지표 워밍업 고려)
- warmup 구간 스킵/가드 로직 추가
- “데이터 수집 실패 → 보수적으로 HOLD” 정책

### Verification
- 지표 NaN 구간에서 자동 스킵 동작 확인
- 데이터 부족 상황을 강제로 재현해 안전하게 동작하는지 점검

---

## 3) Backtest vs Live parity: decision rules (ongoing)
### Issue
백테스트와 라이브가 의사결정/예외처리 규칙이 다르면,
백테스트가 실거래를 정확히 대표하지 못해 결과 해석이 흔들릴 수 있음.

### Mitigation plan
- “동일 조건 입력 → 동일 액션”을 보장하는 parity 체크리스트 구축
- 동시 신호, 시간대 제한, 쿨다운 등 edge-case 시나리오 중심으로 테스트

---

## 4) Disclosure
- 실거래 키, 원천 데이터, 구체 전략 파라미터는 공개하지 않습니다.
- 이 문서는 엔지니어링 관점의 이슈/해결/검증을 공유하기 위한 목적입니다.
