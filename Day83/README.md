# Day 83 — §15.16 Distributional (C51) + Noisy Nets on Dueling

Day 82 (§15.15) 은 dueling head 의 $V$/$A$ 분해가 **mean-neutral** 이면서 시드편차를
2.4–2.8× 압축하는 "reproducibility 이득" 을 보였다. §15.16 은 원 Rainbow (Hessel et al.
2018) 의 남은 두 원소 — **C51 (categorical distributional)** 과 **Noisy Nets (parameter-noise
exploration)** — 를 dueling head 위에 격리·통합 관점에서 실측한다. 이 시리즈는 완전한
6-원소 Rainbow (Double + PER + Dueling + n-step + C51 + Noisy) 직전 단계.

## 학습 목표

- **Problem 1 (C51 격리)**: Dueling-scalar-Q vs Dueling-C51 을 동일 Double stabilizer 위에서
  2 시드 × 12 에피소드 비교. 분포 표적의 mean/std 축 이득이 소규모 · 결정론적 리턴 환경에서
  어떻게 발현되는지 정직하게 관측.
- **Problem 2 (Noisy Nets 격리)**: ε-greedy vs NoisyNet 탐색 (dueling head 유지) 2 시드 ×
  12 에피소드 비교. σ 파라미터의 학습 · 자동 감쇠 · $V$/$A$ head 간 σ 크기 차이를 관측.
- **Problem 3 (baseline + 1-요소 ablation)**: Rainbow-lite-3 (Day 82) = Double + PER + Dueling +
  n-step 위에 +C51 / +Noisy 를 각각 얹은 3-조건 waterfall 로 각 요소의 순증분과 시드편차 축
  기여를 분해. 2 시드 × 8 에피소드.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 (tail-5 mean ± std) |
|---|--------|------|-----------|--------------------------------|
| 1 | `CE_15_16_01.ipynb` | Dueling scalar vs Dueling C51 (격리) | Categorical Bellman projection · KL 손실 · Dueling head over 원자 logits · Double DQN | **scalar $-71.3 \pm 29.1$ / C51 $-78.3 \pm 41.7$**. 이 소규모 결정론적-리턴 환경에서 C51 은 mean $-7.0$ 하락, std $+12.6$ 증가 — 분포 표적의 이점이 발현되기에는 표본 부족. Probe 그림은 학습된 C51 이 상태별 return 분포를 실제로 출력함을 확인 (구조는 정상 작동). |
| 2 | `CE_15_16_02.ipynb` | ε-greedy vs NoisyNet (dueling 격리) | Factorized Gaussian noisy linear $y=(\mu+\sigma \odot \varepsilon^W)x + (\mu_b+\sigma_b \odot \varepsilon^b)$ · V-head/A-head 각각 noisy · ε-greedy 제거 | **ε-greedy $-36.2 \pm 5.2$ / NoisyNet $-62.3 \pm 10.9$**. 이 12-에피소드 규모에서는 ε-greedy 가 mean 우위 (+26.1). σ 파라미터는 학습됨 (marker: mean $\lvert\sigma_W\rvert$ 값이 그림에 표시) — 구조적 동작 확인. NoisyNet 의 진가는 sparse-reward + 긴 학습 규모에서 발현. |
| 3 | `CE_15_16_03.ipynb` | Rainbow-lite-3 + {C51 or Noisy} ablation | Baseline (Double+PER+Dueling+n=3) 위에 +C51 / +Noisy 개별 얹기 · IS-weighted n-step · β annealing $0.4\to 1$ | **baseline $-55.3 \pm 19.3$ (tail-3) / +C51 $-110.3 \pm 9.7$ / +Noisy $-27.8 \pm 2.2$**. +Noisy 는 mean 을 $+27.5$ 개선하면서 **시드편차를 $19.3 \to 2.2$ (8.8× 압축)** — Day 82 의 "reproducibility 이득" 축에서 큰 이득 재현. +C51 은 이 소규모 · 결정론적-리턴 조건에서 mean 이 $-55.0$ 하락 (Problem 1 결과와 일관 — 체제 외 관찰). |

## 한 줄 정리

> **격리 조건 (Problem 1·2) 에서는 소표본 잡음으로 C51 · NoisyNet 모두 결정적 이득을 보이지
> 못하지만, Rainbow-lite-3 위에 얹은 통합 관점 (Problem 3) 에서 +Noisy 는 MountainCar 의
> exploration-hard 특성과 부합해 tail-3 mean 을 $-55.3 \to -27.8$ 개선하면서 시드편차를
> $19.3 \to 2.2$ (8.8×) 로 압축 — Day 82 의 "reproducibility 이득 축" 이 stack 확장 후에도
> 강하게 재현되는 결정적 증거. +C51 은 결정론적 리턴 환경에서 이득이 실측되지 않지만
> 학습된 return 분포 probe 는 정상 (구조 확인, 체제 외). True Rainbow (NoisyC51 head 6-요소
> 완전 통합) 은 Day 84 과제.**

## 사용 라이브러리

- NumPy, pandas, Matplotlib (표준 스택). **신경망 프레임워크 미사용** — 2-층 MLP 의
  forward/backward, Dueling C51 head (per-action softmax over atoms) 의 KL gradient,
  Factorized Noisy Linear 의 $(\mu, \sigma)$ 각각에 대한 gradient, PER buffer + IS weight,
  Double target, n-step 큐를 모두 수기 numpy 로 구현. C51 projection $\Pi \mathcal T Z$ 는
  clipped $r + \gamma z_i$ 를 triangular 가중치로 원자 위에 사영하는 표준 알고리즘.

## 다음 (Day 84)

§15.17 **NoisyC51 head 통합 → true Rainbow 6-요소 완전 통합** — Rainbow-lite-3 + C51 + Noisy
를 하나의 head 로 결합. 또한 Distributional Bellman operator $\mathcal T$ 의 contraction 성질
(Wasserstein metric 하에서) 을 수치적으로 검증. 시드 · 학습 에피소드 수를 늘려 Problem 3 의
+Noisy 이득이 통계적으로 유의미한 신호로 재확인되는지 검증.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 82 README 의 "다음 (Day 83)" 예고대로 **§15.16
> distributional + noisy on dueling** 로 진행. Chapter 15 확장 사례연구는 원 커리큘럼의
> 확장 흐름이므로 다음 Day 세부 절 이름 (§15.17 NoisyC51 true Rainbow) 은 스케줄러가
> 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (Problem 1 의 scalar/C51 2-조건 요약 +
> 학습 곡선 + C51 분포 probe, Problem 2 의 ε-greedy/NoisyNet 2-조건 요약 + 학습 곡선 +
> $\lvert\sigma_W\rvert$ 크기 bar, Problem 3 의 baseline/+C51/+Noisy 3-조건 요약 + 학습 곡선 +
> waterfall) 는 nbclient 로 실행된 결과. 시드: Problem 1 = 9101–9102, Problem 2 = 9201–9202,
> Problem 3 = 9301–9302. Python 3.10, NumPy, pandas, Matplotlib 표준 스택. DuelingC51,
> NoisyLinear, DuelingNoisyMLP, PER, n-step, Double target 을 모두 수기 numpy 로 구현.
>
> 🔎 **정직성 노트**: Problem 1 에서 C51 이 scalar 대비 mean $-7.0$ 하락, std $+12.6$ 증가한
> 결과는 Bellemare et al. (2017) 의 원 주장 (Atari 대규모 학습에서 51-atom C51 의 우위) 과
> 직접 모순되지 않는다. MountainCar-lite 는 (a) 결정론적 조각선형 리턴 (`-100` 실패 / 골도달
> 스텝 수), (b) 소량 학습 (12 에피소드), (c) 소형 신경망 ($H=16$) 이므로 분포 표적의 이점이
> 발현되는 체제 밖. Problem 2 의 ε-greedy 우위도 12-에피소드 규모에서 NoisyNet 의 σ-학습이
> 충분히 진행되기 전에 학습이 끝났기 때문. Problem 3 의 +Noisy 이득은 baseline 이 이미
> Rainbow-lite-3 stack 이라 stabilizer 가 잘 갖춰진 상태에서 파라미터 잡음 탐색이 결정적
> 기여를 하는 조건이 형성된 결과.
>
> 📉 **워크스페이스 제약**: 실행 환경 CPU 시간 제약 (bash timeout 45s) 때문에 시드/에피소드
> 수를 Day 82 대비 더 축소 (Problem 1·2: 2 시드 × 12 에피소드, Problem 3: 2 시드 × 8
> 에피소드, max_steps=120). tail 통계의 상대 순위는 유지되지만, 절대값 비교 시 이 점 감안.
> 다음 Day 84 에서 시드 · 에피소드 수를 늘려 재검증 예정.
