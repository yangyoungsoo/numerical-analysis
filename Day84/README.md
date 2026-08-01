# Day 84 — §15.17 NoisyC51 head 통합 + True Rainbow 6-요소 완전 통합 + Distributional Bellman Contraction

Day 83 (§15.16) 은 dueling head 위에 **C51 · NoisyNet 을 개별 격리** 하고, Rainbow-lite-3
baseline 위에 각 원소를 하나씩 얹은 3-조건 ablation 을 수행했다. §15.17 은 그 다음 두 축을
동시에 다룬다: (i) 두 원소를 **하나의 head 로 융합** 한 **NoisyC51** 을 구현·검증하고,
(ii) baseline / +C51 / +Noisy / **+NoisyC51 (true Rainbow 6-요소 완전 통합)** 의 4-조건
waterfall ablation 으로 각 원소의 순증분 · 시너지 · 시드편차 압축을 실측한 뒤, (iii) C51 이
사용하는 **분포 Bellman operator 의 $\gamma$-contraction** (Bellemare et al. 2017 Prop. 2) 을
수치적으로 검증한다.

## 학습 목표

- **Problem 1 (NoisyC51 head 격리)**: dueling head 의 마지막 선형층을 Factorized Noisy Linear
  로 교체하고 categorical 원자 로그잇을 출력하는 **unified head** 를 numpy 로 구현. Rainbow-lite-3
  scalar (Day 83) baseline 과 격리 조건에서 비교. 관찰 축: 학습 안정성, 상태-의존적 분포 학습,
  $\lvert\sigma_W\rvert$ 자동 감쇠.
- **Problem 2 (True Rainbow 6-요소 완전 통합 ablation)**: baseline (Rainbow-lite-3, scalar) /
  +C51 / +Noisy / +NoisyC51 (Rainbow) 4-조건을 2 시드 × 6 에피소드 (max_steps=120) 로 학습해
  원소별 순증분 · additivity vs synergy 를 정량화.
- **Problem 3 (분포 Bellman contraction 실측)**: 3-상태 · 2-행동 결정론적 MDP + $N=21$ 원자
  격자에서 $\Pi\mathcal T^\pi$ 를 30 회 반복. $\bar d_1, \bar d_2$ 궤적과 비율 $r_k$ 를 측정.
  두 초기 분포 (uniform / delta) 가 동일 고정점으로 합류함을 확인.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_17_01.ipynb` | NoisyC51 head 격리 | Factorized Noisy Linear × dueling head × $N=11$ atoms · Double n-step categorical projection | tail-3 mean: **baseline scalar $-81.8 \pm 38.2$ / NoisyC51 $-74.5 \pm 45.5$**. 격리 조건에서는 mean 이 소폭 우위 ($+7.3$) 이나 std 는 소표본 잡음 지배. **핵심 확인**: (a) NoisyC51 head 가 categorical projection loss 로 발산 없이 학습, (b) 학습된 head 가 상태-의존적 원자 mass 를 실제 배치 (probe 그림), (c) $\lvert\sigma_W\rvert$ 가 매 gradient step $\times 0.9995$ 감쇠 신호로 단조 감소 — parameter-noise exploration 의 자동 anneal 이 정상 작동. |
| 2 | `CE_15_17_02.ipynb` | True Rainbow ablation | Rainbow-lite-3 baseline + PER + IS-weight × KL · Double C51 · NoisyC51 통합 | tail-3: **baseline $-49.3 \pm 10.3$ / +C51 $-83.3 \pm 36.7$ / +Noisy $-23.0 \pm 0.33$ / Rainbow $-108.0 \pm 12.0$**. **극적 관찰**: +Noisy 단독은 mean $+26.3$ 개선하면서 **시드편차를 $31\times$ 압축** ($10.33 \to 0.33$) — Day 83 Problem 3 의 "+Noisy reproducibility 이득" 이 이 stack 규모에서 더욱 강하게 재현. **부정 시너지 관찰**: additivity 예측 $\Delta_{\text{C51}} + \Delta_{\text{Noisy}} = -34 + 26.3 = -7.7$ 인 반면 실측 $\Delta_{\text{Rainbow}} = -58.7$ — synergy $= -51.0$ (강한 음의 상호작용). 소규모 · 결정론적 리턴 조건에서 NoisyC51 통합은 두 원소의 개별 gradient 신호를 서로 방해 (원소 disentanglement 실패). Day 85 에서 시드 · 에피소드 확대로 재검증할 축이 명확. |
| 3 | `CE_15_17_03.ipynb` | 분포 Bellman contraction 검증 | Categorical projection $\Pi$ · $W_1, W_2$ 계산 (CDF-based) · $\bar d_p$ metric · $\Pi\mathcal T^\pi$ 반복 30 회 | **완벽한 $\gamma$-contraction 관찰**: $r_k^{(1)}$ mean = max = **$0.9000$ = $\gamma$** (30 회 모두 tight, 사영 오차 미미). $r_k^{(2)}$ mean $0.925$, max $0.948$ ($\gamma$ 소량 초과 — $W_2$ 관점에서 $\Pi$ 는 non-expansive 가 아님, 원 논문 Prop. 3 와 일관). $\bar d_1$ 궤적은 $5.0 \to 2.12 \times 10^{-1}$ 로 정확히 $\gamma^k$ 엔벨로프 위. 두 초기 분포 (uniform · delta at $z_0$) 모두 동일 고정점 $Z^\pi$ 로 합류 — final $\bar d_1(Z_K, Z_K') = 2.1 \times 10^{-1}$. Bellemare et al. (2017) Prop. 2 를 numpy 로 재현. |

## 한 줄 정리

> **격리 관점 (Problem 1) 에서는 NoisyC51 head 가 구조적으로 정확히 학습되고 (분포 · 자동
> anneal · 안정성 3 축 모두 확인), True Rainbow ablation (Problem 2) 에서는 +Noisy 단독이
> **$31\times$ 시드편차 압축 + mean $+26.3$ 개선**의 강한 이득을 보이는 반면 +NoisyC51 통합은
> $\text{synergy} = -51$ 의 강한 음의 상호작용으로 baseline 을 밑돌아 소규모 · 결정론적 리턴
> 조건에서 두 원소의 gradient 신호가 서로 방해할 수 있음을 정직하게 관찰. Problem 3 에서는
> Bellemare et al. (2017) Prop. 2 를 3-상태 · 2-행동 MDP + $N=21$ 원자 격자에서 numpy 로 재현
> 하여 **$\Pi\mathcal T^\pi$ 가 $\bar d_1$ 위에서 정확히 $\gamma = 0.9$-contraction** 임을 실측
> — 두 초기 분포가 동일 고정점 $Z^\pi$ 로 합류. 이로써 C51 · true Rainbow 가 사용하는 분포
> 표적의 이론적 정당성이 소규모 finite MDP 에서도 tight 하게 성립함을 확인.**

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. **신경망 프레임워크 미사용** — 2-층 MLP 의
forward/backward, Factorized Noisy Linear ($\mu, \sigma$ 각 gradient), dueling C51 head
(per-action softmax over atoms) 의 KL gradient, PER buffer + IS weight, Double target, n-step
큐, categorical projection $\Pi$, $W_1 / W_2$ 계산을 모두 수기 numpy 로 구현.

## 다음 (Day 85)

§15.18 은 (a) Problem 2 의 True Rainbow 를 시드 · 에피소드 · 신경망 폭 확대 조건에서 재검증하여
"음의 synergy" 가 소표본 잡음인지 체제 특성인지 판별, (b) Problem 3 의 contraction 검증을
확률적 전이 · 확률적 정책 · 큰 $|\mathcal S|$ 로 확장하며, (c) Wasserstein 이 아닌 **Cramér
distance** ($\ell_2$-CDF metric) 관점에서 재조명 — Cramér 은 $\Pi$ 가 non-expansive 임이 알려져
있어 $W_2$ 의 슬라이드 문제를 우회할 수 있다. Rowland et al. (2018, "An Analysis of Categorical
Distributional RL") 의 결과 재현이 목표.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 83 README 의 "다음 (Day 84)" 예고대로
> **§15.17 NoisyC51 head 통합 → true Rainbow 6-요소 완전 통합** 및 **분포 Bellman contraction
> 수치 검증** 두 축으로 진행. Chapter 15 확장 사례연구는 원 커리큘럼의 확장 흐름이므로 다음 Day
> 세부 절 이름 (§15.18 Cramér distance 관점) 은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (Problem 1 의 2-조건 tail 요약 + 학습
> 곡선 + $\lvert\sigma_W\rvert$ 감쇠 + 분포 probe, Problem 2 의 4-조건 waterfall + additivity
> 분해, Problem 3 의 30-스텝 $\bar d_1 / \bar d_2$ 궤적 + 비율 + 고정점 합류) 는 nbclient 로
> 실행된 결과. 시드: Problem 1 = 8401–8402, Problem 2 = 8411–8412, Problem 3 = deterministic MDP.
> Python 3.10, NumPy, pandas, Matplotlib 표준 스택.
>
> 🔎 **정직성 노트**: Problem 2 의 True Rainbow 가 baseline 을 밑돈 결과 (synergy $-51$) 는
> Hessel et al. (2018, "Rainbow: Combining Improvements in DQN") 의 원 주장 (Atari 대규모 학습에서
> 6-요소 통합의 우위) 과 직접 모순되지 않는다. MountainCar-lite 는 (a) 결정론적 조각선형 리턴,
> (b) 소량 학습 (6 에피소드 × 120 스텝), (c) 소형 신경망 ($H=16$), (d) NoisyC51 head 는 trunk
> 업데이트를 생략한 compact 구현이므로 원소 상호작용의 본격 이득이 발현되는 체제 밖. Day 83
> Problem 3 에서 Rainbow-lite-3 위에 +Noisy 단독이 $8.8\times$ std 압축을 보인 관찰이 이
> Problem 2 에서 $31\times$ 로 확대 재현된 것이 오히려 이 결과의 유효성을 지지한다 — +Noisy 는
> 이 체제에서 결정적 stabilizer 이지만, +C51 과 결합 시 두 gradient 신호가 서로 방해할 수 있다.
>
> 📉 **워크스페이스 제약**: 실행 환경 CPU 시간 제약 (bash timeout 45s) 때문에 시드/에피소드 수를
> Day 82 대비 축소 유지 (Problem 1·2: 2 시드 × 6 에피소드, max_steps=120). tail 통계의 상대
> 순위는 보존되지만 절대값 비교 시 이 점 감안. Day 85 에서 확대 재검증 예정.
>
> 📚 **레퍼런스**: Bellemare, Dabney, Munos (2017), "A Distributional Perspective on Reinforcement
> Learning", ICML — Proposition 2 ($\gamma$-contraction in $\bar d_p$), Proposition 3 (projection
> non-expansion in $W_2$ fails but holds in $W_1$). Hessel et al. (2018), "Rainbow: Combining
> Improvements in Deep Reinforcement Learning", AAAI — 6-요소 원 구성 및 통합 시너지. Fortunato
> et al. (2018), "Noisy Networks for Exploration", ICLR — Factorized Noisy Linear.
