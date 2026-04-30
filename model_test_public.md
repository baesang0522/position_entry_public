# Model & Backtest Summary (Public)

## 1) Goal
이 프로젝트는 “항상 예측”이 아니라, **유리한 구간에서만 진입**하는 의사결정 시스템을 목표로 합니다.  
롱/숏 모델을 분리하고, 확신도가 충분할 때만 진입하도록 설계해 **노이즈 구간의 과진입을 줄이는 방향**을 채택했습니다.

> 전략 세부 규칙(지표/임계값/파라미터)과 원천 데이터는 공개하지 않습니다.

---

## 2) High-level design
- **Dual specialist models**: Long specialist / Short specialist
- **Regime-aware training**: 불리한 국면에서는 “기회 없음”에 가까운 확률이 나오도록 학습 설계
- **Probability gating**: 고확신 구간만 진입
- **Risk controls**: 단일 포지션, 쿨다운, 시간 기반 종료 등

---

## Latest backtest snapshot (public)
- Total Return: **21.53%**
- Win Rate: **60.53%**
- Trades: **38**
- Final Capital: **$12,152.69**
- Max Drawdown: **-27.07%**


### Important note (correctness)
최근 공개 수치는 **백테스트 정합성 개선 이후** 기준입니다.

- **Timing alignment fix**: 신호 생성(T-1)과 진입(T)을 분리
- **Entry price**: T 시점에 관측 가능한 값(open) 사용
- 목적: 동일 봉 정보(특히 종가)를 이용해 “이미 아는 미래”로 진입하는 형태의 **lookahead 위험**을 줄이기 위함

---

## 4) Stress / robustness test (public-safe summary)
실거래에서는 체결 지연, 진입 타이밍 오차, 더 보수적인 운영 조건이 존재하므로  
기본 백테스트 외에 **스트레스 테스트 / robustness 테스트**도 함께 확인했습니다.

### What was stressed
- 더 보수적인 진입 조건
- 더 불리한 슬리피지 가정
- 더 긴 쿨다운
- 더 짧은 시간 청산(time cut)
- 진입 타이밍 흔들림(shift)

### Key takeaway
- 일반 백테스트와 스트레스 테스트의 결과 차이가 꽤 크게 날 수 있음을 확인했습니다.
- 이는 전략이 **좋은 조건에서는 성과를 내더라도, 실행 타이밍과 운영 가정 변화에 민감할 수 있음**을 보여줍니다.
- 따라서 이 프로젝트는 단순 수익률 숫자보다도, **parity / robustness / operational safety**를 함께 보는 방향으로 개선 중입니다.

> 스트레스 테스트의 세부 파라미터와 분포는 비공개입니다.

---

## 5) Known failure modes (where it can break)
- **횡보 + 급등락(whipsaw)**: 연속 손절/과진입 위험
- **변동성 구조 변화**: 변동성 기반 종료 규칙이 시장과 어긋날 수 있음
- **급변 이벤트**: 짧은 시간 리프라이싱에서 체결 품질/슬리피지로 성과 왜곡 가능
- **Timing sensitivity**: 진입 타이밍이 조금만 흔들려도 결과가 악화될 수 있음

---

## 6) Current validation philosophy
이 프로젝트는 아래 원칙을 기준으로 결과를 해석합니다.

- **Backtest correctness first**: 결과보다 정합성 우선
- **Backtest vs Live parity**: 백테스트와 라이브 의사결정 불일치 최소화
- **Robustness over vanity metrics**: 좋은 숫자 하나보다, 흔들렸을 때 얼마나 버티는지 중시
- **Operational visibility**: 실제 운영 로그를 통해 행동을 검증

---

## 7) Next steps (public roadmap)
- Backtest vs Live decision parity 강화
- Probability calibration / threshold stability checks
- Stress test 해석 체계 정리
- 운영 로그 기반 incident review 루프 고도화