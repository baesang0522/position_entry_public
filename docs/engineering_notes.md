# Engineering Notes (Public)

이 문서는 “전략 레시피”가 아니라, 시스템을 운영하며 마주친 **엔지니어링 이슈와 해결 방식**을 정리합니다.

---

## 1) Backtest correctness: timing alignment (lookahead risk)

### Issue
백테스트에서 진입 시점(T)과 동일한 시점의 상위 타임프레임 정보를 사용하면,
“진입 당시에는 알 수 없는 정보(동일 봉 종가 등)”가 섞여 **lookahead 위험**이 생길 수 있음

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
- 수정 전/후 결과 비교 시, 성능 수치 변화보다 **정합성 반영 여부**를 우선 해석

---

## 2) Stress testing: robustness over single-number backtests

### Issue
일반 백테스트만 보면 성과가 양호해 보여도,
실거래에서는 진입 타이밍 오차, 더 보수적인 운영 조건, 슬리피지 확대 등으로 결과가 크게 달라질 수 있음

### Fix
- 기본 백테스트와 별도로 **stress / robustness test**를 운용
- 보수적 threshold, slippage, cooldown, time-cut 조건을 따로 점검
- 진입 타이밍을 흔드는 시뮬레이션으로 timing sensitivity 확인

### Verification
- 일반 백테스트와 스트레스 테스트 결과 차이를 함께 해석
- “잘 나오는가”보다 “얼마나 쉽게 무너지는가”를 같이 확인
- 전략 평가 기준을 단일 수익률 숫자에서 robustness 관점으로 확장

---

## 3) Operational reliability: data window / indicator warmup

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

## 4) Live execution safety: TP/SL placement & duplicate protection orders

### Issue
라이브에서 포지션 진입은 정상인데, TP/SL 보호주문이 **누락되거나 중복으로 누적**되어 운영 문제가 발생할 수 있음.

- “주문이 있는데도 청산이 안 됨”
- “포지션 닫으려면 기존 주문을 먼저 취소하라”는 거래소 경고

특히 Binance Futures에서 조건부(stop) 주문은 UI에서 `Untriggered Orders`로 분리되어 보이며,
일부 환경에서는 `fetch_open_orders()`가 이 주문들을 항상 포함하지 않아 “미등록으로 오판 → 재발행 → 중복 누적”이 발생할 수 있음.

### Root cause (conceptual)
- SL을 조건부 주문(STOP_MARKET)으로 생성하면 거래소가 `Untriggered Orders`로 분리 보관할 수 있어,
  일반 오픈오더 조회만으로는 “등록 여부”를 정확히 판단하기 어려움
- 보호주문 재시도(rescue) 로직이 “오픈오더 미검출”을 실패로 판단할 경우,
  동일한 reduce-only stop 주문이 여러 개 쌓이는 현상이 발생할 수 있음
- reduce-only 주문 수량이 포지션 수량을 초과하거나,
  동일 방향의 reduce-only 보호주문이 여러 개 존재하면,
  거래소가 “reduce-only 위반 가능성(포지션 반전 위험)”을 이유로 체결/청산을 제한할 수 있음

### Fix
- SL(STOP_MARKET)은 **order id 기반으로만 등록 여부를 확인**
- id 확보 이후에는 “오픈오더 미검출”만으로 **재발행하지 않도록** 변경
- TP(LIMIT reduceOnly)는 기존 검증/재시도 흐름을 유지하되,
  보호주문 판정을 강화하여 중복 누적을 줄임
- 운영 가이드:
  - SL은 UI에서 `Untriggered Orders`에 보일 수 있으며 이는 정상 동작
  - TP(LIMIT)는 `Normal Orders`에 보이며 “Trigger condition” 컬럼이 비어있는 것이 정상

### Verification
- 진입 직후 SL이 `Untriggered Orders`에 1개만 유지되는지 확인
- TP가 `Normal Orders`에 1개만 유지되는지 확인
- rescue 동작 시 SL 재발행이 발생하지 않는지 로그/주문내역으로 확인
- “포지션 종료 시 기존 주문 취소 요구” 경고가 재발하지 않는지 운영 관찰

---

## 5) Backtest vs Live parity: time-based exit (time cut)

### Issue
백테스트에서만 “시간 청산(time_cut)”이 적용되고 라이브에는 없으면,
백테스트가 실거래를 대표하지 못해 결과 해석이 흔들릴 수 있음.

### Fix
- 라이브 설정에 `time_cut_hours`를 도입
- 라이브 실행 루프에서 **보유 시간이 임계값을 초과하면 주문 정리 후 포지션 종료** 수행
  - 순서: `cancel_all_orders(symbol)` → `close_position(symbol)`
- 운영 진단을 위해 시간청산 발생 시 `[TIME_CUT]` 태그 로그를 남기고,
  내부 상태에 종료 이벤트를 기록

### Verification
- 보유 시간이 임계값에 도달한 포지션에서 time_cut이 정상 발동하는지 확인
- 발동 시 로그가 정확히 남는지 확인
- 발동 후 신규 진입 흐름이 정상적으로 재개되는지 확인

---

## 6) Observability: EC2 local logs → CloudWatch Logs

### Issue
운영 중 발생하는 warning / decision / position 상태 로그가 EC2 내부 파일에만 있으면,
장애 원인 분석과 장기 추적이 번거롭고 원격 확인성이 떨어질 수 있음.

### Fix
- EC2 내부 `trader.log`를 AWS CloudWatch Logs로 수집하도록 연결
- IAM 역할 권한과 CloudWatch Agent 설정을 정리
- 로그 그룹/스트림 기준으로 운영 로그를 중앙에서 조회 가능하게 구성

### Verification
- CloudWatch Logs에서 실제 운영 로그가 지속적으로 적재되는지 확인
- warning, decision, position 상태 로그가 원격에서 조회 가능한지 점검
- 운영 이슈 분석 편의성 확보

---

## 7) Deployment automation: GitHub push → AWS restart

### Issue
수동 배포는 누락, 경로 실수, 재시작 누락 등으로 이어질 수 있어
실운영 반영 속도와 재현성이 떨어질 수 있음.

### Fix
- GitHub push 이후 AWS 쪽 배포 흐름이 자동으로 이어지도록 구성
- SSM 기반 실행/재시작 흐름을 사용하여 서버 반영 절차를 자동화
- 로컬 수동 반영보다 “같은 절차를 반복 가능하게” 만드는 방향으로 정리

### Verification
- push 이후 서버에 최신 코드가 반영되는지 확인
- 서비스 재시작이 자동으로 수행되는지 확인

---

## 8) Maintainability: live loop refactor (guard extraction)

### Issue
라이브 실행 로직이 단일 파일에 과도하게 집중되면(수백 라인),
버그 수정/검증이 어려워지고 운영 안정성 개선 속도가 느려질 수 있음.

### Fix
- 포지션 보유시간/시간청산/상태관리 블록을 `position_guard` 모듈로 분리하여
  `execute_logic()`의 책임을 “오케스트레이션” 중심으로 얇게 유지
- 리팩토링은 동작 변경 없이 코드 이동/래핑 중심으로 수행
- 단일 소스 원칙 유지: `pos`/`has_position`은 상위에서 계산한 값을 guard에 전달

### Verification
- 분리 전/후 동일 입력에서 동일한 종료 신호/로그/반환 경로가 유지되는지 확인
- 포지션 조회 호출 횟수가 증가하지 않는지 확인

---

## 9) Disclosure
- 실거래 키, 원천 데이터, 구체 전략 파라미터는 공개하지 않습니다.
- 이 문서는 엔지니어링 관점의 이슈/해결/검증을 공유하기 위한 목적입니다.