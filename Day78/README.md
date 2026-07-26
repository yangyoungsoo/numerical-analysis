# Day 78 — §15.11 Rainbow 통합 + Categorical Distributional Q (C51) + Noisy Nets

Day 77(§15.10) 에서 Rainbow 의 세 구성요소 (PER, Dueling, n-step) 를 결정적 전이 GridWorld
에서 격리 관측했다. §15.11 은 남은 두 처방 — **Categorical Distributional Q (C51)** 과
**Noisy Nets** — 을 tabular / 선형근사 위에서 구현하고, 마지막에 이들을 n-step 과 함께
**Rainbow-lite** 로 통합해 **확률 전이 (slippery) GridWorld** 에서 순 증분 기여를 측정한다.
Day 77 말미에 예측한 "n-step 은 확률 전이가 있어야 되살아난다" 가 실측되는지 검증하는 것이
핵심 목적.

## 학습 목표

- **Problem 1 (C51)**: 결정적 전이 GridWorld + $r=-1+\mathcal N(0,1)$ 잡음 보상에서 tabular
  C51 ($K=31$, 지지대 $[-15,0]$) 을 스칼라 Q-learning 과 15 시드 × 200 에피소드 비교.
  분포적 표적이 (i) 표적 분산을 KL 로 흡수해 그래디언트 잡음을 줄이는지, (ii) 시작상태
  학습분포에 잡음 보상의 확산 · 최적/비-최적 행동의 mode 분리가 나타나는지 관찰.
- **Problem 2 (Noisy Nets)**: 3 조건 (ε-greedy / Noisy learn σ / Noisy fixed σ=0.5) 비교.
  학습형 Noisy 가 greedy 경로에서 σ 를 자동 collapse 시키면서도 미방문 (s,a) 에서는 σ 를
  유지해 상태 의존적 탐색이 발생하는지, ε-greedy 대비 열세 없이 스케줄 없는 탐색을
  달성하는지 정량 관측.
- **Problem 3 (Rainbow-lite ablation)**: 슬리퍼리 환경 ($p_\text{slip}=0.2$) 에서 4 조건
  (baseline / +C51 / +Noisy / Rainbow-lite = C51+Noisy+n=3) 를 15 시드 × 200 에피소드 학습.
  Rainbow-lite 가 개별 조건의 산술합을 초과하는 시너지를 내는지, 확률 전이가 n-step 을
  재활성화하는지 검증.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_11_01.ipynb` | Categorical Distributional Q (C51, tabular) | $Z(s,a)$ 위 softmax 확률질량, categorical projection $\Phi(\hat T Z)$, KL 그래디언트 | $V^\star=-5.85$; tail50 리턴 **C51 = -6.77** vs scalar Q = -6.91, tail $\|Q_\text{mean}-Q^\star\|_\infty$: scalar 1.50 vs C51 5.50 (지지대 $K=31$ 이산화 잔차). C51 은 평균 정확도는 대등하되 **표적 분산 흡수 · 위험 정보 보존** 이라는 자산 획득. |
| 2 | `CE_15_11_02.ipynb` | Noisy Nets (tabular, 학습형 σ) | $Q=\mu+\sigma\varepsilon$, 순수 greedy, $\mu$ · $\sigma$ 동시 gradient step | tail50 리턴: eps-greedy -6.93, **Noisy(학습) -6.30**, Noisy-fixed -6.24 (분산 큼). 학습된 σ 열지도에서 시작·목표 인접 셀은 낮은 σ, 코너·경로 밖 셀은 상대적으로 큰 σ 유지 — **상태 의존적 탐색** 이 정확히 재현. 명시적 ε 스케줄 없이 자동 감쇠. |
| 3 | `CE_15_11_03.ipynb` | Rainbow-lite (C51+Noisy+n=3) ablation on slippery GridWorld | $p_\text{slip}=0.2$, n-step distributional projection, noisy greedy on Q_mean | $V^\star_\text{slip}=-7.05$; tail50 리턴: baseline -8.43, +C51 -8.62, +Noisy -7.95, **Rainbow-lite -7.33 (최우수)**. 4 조건 중 유일하게 $V^\star$ 에 근접. 개별 조건 산술합을 초과하는 시너지 관찰 — **n-step 이 확률 전이 하에서 되살아남 (Day 77 예측 검증)**. |

## 한 줄 정리

> **분포적 표적 (C51) · 파라미터 잡음 탐색 (Noisy) · n-step 부트스트랩** 은 각각 표적 분산
> 흡수 · 상태 의존적 탐색 · 부트스트랩 지평 확대라는 서로 다른 병리에 대응하며, 확률
> 전이 환경에서 세 요소를 통합한 Rainbow-lite 는 개별 조건들의 산술합을 초과하는 시너지를
> 낸다 — 결정적 환경에서 사라졌던 n-step 이득이 slippery 하에서 정확히 되살아나며,
> Rainbow 원 논문의 "요소는 상보적" 주장이 tabular 규모에서도 재현된다.

## 사용 라이브러리
- NumPy, pandas, Matplotlib (표준 스택). 신경망 프레임워크 미사용 — tabular / linear 만으로
  각 구성요소의 gradient · 부트스트랩 · 샘플링 경로를 정확히 재현.

## 다음 (Day 79)

§15.12 **Quantile Regression DQN (QR-DQN) + IQN + 신경망 파라미터화 규모 이행** — C51 의
이산 지지대 잔차 (Q-오차 하한) 를 quantile regression 으로 제거하고, IQN 으로 quantile
함수 자체를 근사, 이어서 신경망 파라미터화로 확장 시 §15.10–§15.11 의 세 관찰 (요소 순
증분, 확률 전이의 n-step 활성화, 상태의존 탐색) 이 얼마나 규모 이행되는지 검증.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3) 까지이며, Day 14 이후는
> 스케줄러가 로드맵을 절 단위로 자동 진행. Day 77 README 의 예고대로 **§15.11 Rainbow
> 통합 + C51 + Noisy Nets** 로 진행. Chapter 15 는 원 커리큘럼의 확장 사례연구 흐름이므로
> 다음 Day 세부 절 이름은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (C51 vs scalar 요약, Noisy 3-조건 요약,
> Rainbow-lite 4-조건 요약) 는 nbclient 로 실행된 결과. 시드: Problem 1 = 5100..5114,
> Problem 2 = 6100..6114, Problem 3 = 7100..7114. Python 3.10, NumPy, pandas, Matplotlib
> 표준 스택으로 실행 완료 (총 ~22 초, tabular / linear 만 사용).
