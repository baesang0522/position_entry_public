# Model & Backtest Summary (Public)

## 1) Goal
이 프로젝트는 “항상 예측”이 아니라, **유리한 구간에서만 진입**하는 의사결정 시스템을 목표로 합니다.  
롱/숏 모델을 분업하고, 확신도가 충분할 때만 진입해 노이즈/횡보 구간의 과진입을 줄이는 방향을 채택했습니다.

> 전략 세부 규칙(지표/임계값/파라미터)과 원천 데이터는 공개하지 않습니다.

---

## 2) High-level design
- **Dual specialist models**: Long specialist / Short specialist
- **Regime-aware training**: 불리한 국면에서는 “기회 없음”에 가까운 확률이 나오도록 학습 설계
- **Probability gating**: 고확신 구간만 진입
- **Risk controls**: 단일 포지션, 쿨다운, 시간 기반 종료 등

---

## 3) Backtest snapshot (public)
- Total Return: **26.95%**
- Win Rate: **59.26%**
- Trades: **27**
- Final Capital: **$12,694.63**

### Important note (correctness)
최근 결과는 백테스트 정합성 개선을 반영합니다.

- **Timing alignment fix**: 신호 생성(T-1)과 진입(T)을 분리  
- **Entry price**: T 시점에 관측 가능한 값(open)을 사용  
- 목적: 동일 봉 정보(특히 종가)를 이용해 “이미 아는 미래”로 진입하는 형태의 **lookahead 위험**을 줄이기 위함

---

## 4) Robustness test (conceptual)
실거래에서는 체결 지연/타이밍 오차가 존재하므로, 진입 타이밍을 랜덤하게 흔들어 반복 실행하는 스트레스 테스트를 수행했습니다.

- 목적: “타이밍이 조금 틀려도 전략 성과가 급격히 붕괴하지 않는가?”
- 결과: 다수의 시뮬레이션에서 성과가 양(positive)으로 유지되는 경향을 확인  
  (세부 분포/파라미터는 비공개)

---

## 5) Known failure modes (where it can break)
- **횡보 + 급등락(whipsaw)**: 연속 손절/과진입 위험
- **변동성 구조 변화**: 변동성 기반 종료 규칙이 시장과 어긋날 수 있음
- **급변 이벤트**: 짧은 시간 리프라이싱에서 체결 품질/슬리피지로 성과 왜곡 가능

---

## 6) Next steps (public roadmap)
- Backtest vs Live decision parity (동일 의사결정 보장)
- Probability calibration / threshold stability checks
- Slippage/fee sensitivity analysis with measured distributions (redacted)
