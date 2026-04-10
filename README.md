# Position Entry System (Public Portfolio)

자동매매/시그널링 시스템을 **설계 → 검증 → 라이브 운영**까지 이어가며 얻은 엔지니어링/리서치 인사이트를 정리한 public-safe 레포입니다.  
실제 자본으로 운영 중이지만, **실거래 키·원천 데이터·전략의 세부 규칙(지표/임계값/파라미터/시간대 정책)** 은 보안 및 엣지 보호를 위해 비공개로 유지합니다.

---

## What I built
- **Dual-direction modeling**: 롱/숏을 분업하는 구조
- **Signal gating**: 확신도가 충분히 높을 때만 진입하는 보수적 운영
- **Risk controls**: 단일 포지션, 재진입 제한(쿨다운), 시간 기반 종료 등 안전장치
- **Backtest realism**: 상위 타임프레임(신호)과 하위 타임프레임(청산 판정)을 분리하여 과대평가를 줄이는 방향
- **Live Time-Cut Exit**: 일정 시간 이상 보유 시 강제 청산
- **Operational logging**: 운영 로그를 수집하고 추적 가능한 형태로 관리
- **Deployment automation**: GitHub push 이후 AWS 쪽 자동 반영/재시작 흐름 구축

---

## Latest backtest snapshot (public)
- Total Return: **26.95%**
- Win Rate: **59.26%**
- Trades: **27**
- Final Capital: **$12,694.63**

---

## Key engineering highlights

### 1) Backtest correctness fix (timing alignment / lookahead prevention)
신호 생성 시점(T-1)과 진입 시점(T)을 분리하고,  
진입가는 T 시점에 관측 가능한 값(open)을 사용하도록 정합성을 강화했습니다.

### 2) Robustness evaluation
기본 백테스트 외에도 체결 타이밍 흔들림과 더 보수적인 운영 조건을 가정한  
스트레스 테스트를 통해 **전략의 timing sensitivity**를 점검했습니다.

### 3) Operational safety learnings
라이브 환경에서 발생 가능한 데이터 부족, 지표 계산 warning, 주문 보호 로직 문제를 진단하고  
보수적 가드 로직을 추가했습니다(세부 규칙은 비공개).

### 4) CloudWatch-based monitoring
EC2 내부 로그 파일(`trader.log`)을 AWS CloudWatch Logs로 수집하도록 연결하여,  
운영 중 발생하는 warning / decision / position 상태를 중앙에서 확인할 수 있게 했습니다.

### 5) GitHub → AWS deployment automation
GitHub에 push하면 AWS 쪽 배포 흐름이 자동으로 이어지고,  
서버에서 최신 코드 기준으로 서비스가 재시작되도록 구성했습니다.  
이를 통해 수동 배포보다 **반영 속도와 재현성**을 높였습니다.

---

## Current operating philosophy
이 프로젝트는 단순히 “백테스트 수익률이 높은가?”만 보지 않습니다.

- **정합성이 맞는가**
- **라이브와 같은 의사결정을 하는가**
- **운영 중 문제를 빨리 볼 수 있는가**
- **배포가 반복 가능하고 안전한가**

이 네 가지를 함께 보는 구조로 개선하고 있습니다.

---

## Docs
- `docs/model_test_public.md` — 공개용 검증 요약
- `docs/engineering_notes.md` — 버그 수정 / 운영 안정화 / 배포 관련 메모