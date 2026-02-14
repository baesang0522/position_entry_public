# 🐶 MS Trading Bot (Public Notes)

자동매매/시그널링 시스템을 **설계–검증–라이브 운영**까지 이어가며 얻은 인사이트를 정리한(public-safe) 레포입니다.  
실제 자본으로 운용 중인 시스템이지만, **실거래 키/원천 데이터/구체 전략 파라미터**는 보안 및 경쟁력 보호를 위해 공개하지 않습니다.

> ⚠️ Not financial advice. This repository is for engineering/research documentation only.

---

## What this project explores

### Core idea
**“유리한 장에서만 싸운다.”**  
시장 국면(regime)을 고려해 **롱/숏을 분업**하고, 모델 확신도가 높은 구간에서만 진입하는 방식으로 과도한 거래와 노이즈 구간을 피합니다.

### High-level architecture
- **Dual model setup**: Long-specialist + Short-specialist
- **Regime-aware training**: 각 모델이 유리한 국면에서만 “성공 패턴”을 학습하도록 라벨링/학습을 설계
- **Probability gating**: 확신도가 충분히 높을 때만 진입
- **Risk controls**: 단일 포지션, 재진입 제한, 시간대 필터(변동성 회피), 시간 기반 종료 등
- **Execution realism**: 신호 생성(상위 TF)과 체결/청산 검증(하위 TF)을 분리하여 과대평가를 줄이는 방향의 백테스트 구성

---

## Performance (Backtest snapshot)

> 아래 수치는 “특정 시점의 백테스트 스냅샷”이며, 미래 성과를 보장하지 않습니다.

| Metric | Value |
|---|---:|
| Period | Recent multi-month test window |
| Total Return | **+25.61%** |
| Win Rate | **61.54%** |
| Trades | **26** |
| Robustness | **94%** positive Monte Carlo runs |

---

## Documents
- **model_test_public.md**: 검증 결과/강건성 테스트/리스크 관찰 요약

---

## Disclosure / Safety
- 이 레포에는 **API 키, 실거래 설정, 구체 전략 파라미터, 원천 데이터**가 포함되지 않습니다.
- 공개 내용은 “연구 방법론/검증 프레임/리스크 인지” 중심으로 정리되어 있습니다.
