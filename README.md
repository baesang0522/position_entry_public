# Position Entry System (Public Portfolio)

자동매매/시그널링 시스템을 **설계 → 검증 → 라이브 운영**까지 이어가며 얻은 엔지니어링/리서치 인사이트를 정리한(public-safe) 레포입니다.  
실제 자본으로 운영 중이지만, **실거래 키·원천 데이터·전략의 세부 규칙(지표/임계값/파라미터/시간대 정책)** 은 보안 및 엣지 보호를 위해 비공개로 유지합니다.

> ⚠️ Trading involves substantial risk of financial loss. This repository is for engineering/research documentation only and is not financial advice. :contentReference[oaicite:0]{index=0}

---

## What I built
- **Dual-direction modeling**: 롱/숏을 분업하는 구조(각각 유리한 국면에서 더 잘 동작하도록 설계)
- **Signal gating**: 확신도가 충분히 높을 때만 진입하는 보수적 운영
- **Risk controls**: 단일 포지션, 재진입 제한(쿨다운), 시간 기반 종료 등 안전장치
- **Backtest realism**: 상위 타임프레임(신호)과 하위 타임프레임(청산 판정)을 분리하여 과대평가를 줄이는 방향

---

## Latest backtest snapshot (public)
- Total Return: **26.95%**
- Win Rate: **59.26%**
- Trades: **27**
- Final Capital: **$12,694.63**

> Note: The latest numbers reflect a **backtest timing-alignment fix** to reduce lookahead risk (signal from T-1, entry at T open).  
> Detailed strategy parameters remain intentionally redacted.

---

## Key engineering highlights
- **Backtest correctness fix (timing alignment / lookahead prevention)**  
  신호 생성 시점(T-1)과 진입 시점(T)을 분리하고, 진입가는 T 시점에 관측 가능한 값(open)을 사용하도록 정합성을 강화했습니다.
- **Robustness evaluation**  
  체결 타이밍 흔들림(지연/오차)을 가정한 스트레스 테스트로 신호의 안정성을 점검했습니다.
- **Operational safety learnings**  
  라이브 환경에서 발생 가능한 데이터 부족/지표 계산 실패/운영 제약을 진단하고 방어 로직을 추가했습니다(세부 규칙은 비공개).

---

## Docs
- `docs/model_test_public.md` — 검증 요약(공개용)
- `docs/engineering_notes.md` — 버그 수정/검증 포인트(포트폴리오 핵심)
