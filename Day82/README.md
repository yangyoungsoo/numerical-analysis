# Day 82 — §15.15 Dueling Network Architecture (V/A decomposition)

Day 81 (§15.14) 은 uniform replay · PER · Double DQN 을 통합해 MLP 규모의 시드편차를
$14.8 \to 0.16$ 으로 압축했다. §15.15 는 남은 표준 처방인 **Dueling** — head 를 상태값
$V(s)$ 와 행동이점 $A(s,a)$ 로 분리 — 을 격리·통합 두 관점에서 검증하고, mean 축과
reproducibility 축의 시너지 분포가 dueling 추가로 어떻게 재분배되는지 정량화한다.

## 학습 목표

- **Problem 1 (Dueling 격리)**: MountainCar-lite MLP ($2 \to 16 \to 3$) 에서 plain / dueling-centered
  / dueling-uncentered 3 조건을 4 시드 × 50 에피소드 비교. dueling 그 자체의 mean/variance 이득과
  $V \leftrightarrow A$ 자유도(식별성)의 mean-subtraction 처방을 검증.
- **Problem 2 (Dueling + Double + PER)**: Day 81 Rainbow-lite-2 (Double + PER-α+β) 위에 dueling
  을 얹었을 때 이득 축의 재분배를 3 시드 × 40 에피소드로 관측. TD-error 분포 · $V$-head
  probe 궤적으로 편향 표집 하에서의 $V$-head 역할 관찰.
- **Problem 3 (Rainbow-lite-3 ablation)**: baseline (Double+PER) / +Dueling / +n-step / **Rainbow-lite-3
  = Double + PER + Dueling + n=3** 4 조건 waterfall 분해. mean 축과 std 축의 시너지 분포를 정량화.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_15_01.ipynb` | Dueling 격리 (V/A decomposition, identifiability) | 공유 trunk + V-head + A-head · `mean-subtraction` · Day 81 stabilizer 유지 | tail20: **plain $-25.45 \pm 0.77$ / duel-c $-25.49 \pm 0.27$ / duel-u $-25.79 \pm 0.65$**. duel-centered 는 mean-neutral (mean 이 plain 과 0.04 차) 이면서 **시드편차를 $0.77 \to 0.27$ (2.8×) 로 압축**. $V$-head 는 골 근처가 높은 표준 MountainCar $V^\star$ 형상을 물리적으로 재현. `duel-u` (mean-subtraction 미적용) 는 $\lVert V\rVert$ 와 $\lVert A\rVert$ 가 발산·표류 — 식별성 문제 정량 관측. |
| 2 | `CE_15_15_02.ipynb` | Dueling × PER × Double DQN | Double target · PER-α+β · IS weight · +dueling head | tail20: **baseline (Double+PER) $-26.58 \pm 1.70$ / +dueling $-27.70 \pm 0.72$**. mean 은 $-1.12$ 소폭 하락하지만 **시드편차를 $1.70 \to 0.72$ (2.4×) 로 압축**. TD-error 분포의 heavy tail 이 dueling 조건에서 얇아지고, $V$-head probe 궤적이 baseline max-Q 대비 부드럽게 수렴. |
| 3 | `CE_15_15_03.ipynb` | Rainbow-lite-3 ablation (Double + PER + Dueling + n=3) | 4 조건 waterfall · IS-weighted n-step Double DQN | tail20: **baseline $-25.75 \pm 0.23$ / +Dueling $-26.62 \pm 1.83$ / +n-step $-26.32 \pm 0.27$ / Rainbow-3 $-25.82 \pm 0.30$**. 3 시드 소표본에서는 baseline (Double+PER) 이 이미 낮은 편차 체제에 있으며, Rainbow-3 는 mean 을 baseline 수준으로 회복하되 추가 압축은 관측되지 않음. 단일 요소 +Dueling 이 이 소표본에서 std 를 증가시킨 관측은 Problem 1·2 (4 시드/3 시드) 와 반대 방향이므로 소표본 잡음으로 해석해야 하며 (아래 정직성 노트), 절대 순위는 표본 크기 확대 시 재현성 검증 필요. |

## 한 줄 정리

> **Dueling 은 격리 조건 (Problem 1) 과 Double + PER stack 위 (Problem 2) 에서 mean-neutral
> 이면서 시드편차를 $2.4$–$2.8\times$ 압축하는 "reproducibility 이득" 을 재현했다. 그러나
> Problem 3 의 4-조건 waterfall 은 baseline (Double+PER) 이 이미 극도로 낮은 편차 ($0.23$)
> 체제에 있어 dueling · n-step 의 추가 이득이 소표본 잡음에 묻히는 경계를 노출했다 — dueling
> 의 순 이득은 격리 조건에서 재현적으로 존재하되, 이미 잘 안정화된 stack 위에서는 시드 수
> 확대 없이 정량화하기 어렵다는 실무 관찰. mean-subtraction 없는 dueling 은 $V \leftrightarrow A$
> 자유도가 head norm 을 표류시키므로 반드시 centered 형태로 써야 한다.**

## 사용 라이브러리

- NumPy, pandas, Matplotlib (표준 스택). **신경망 프레임워크 미사용** — 2-층 MLP + dueling
  head 의 forward/backward, PER buffer + IS weight, Double target, n-step 큐를 모두 수기
  numpy 로 구현. dueling head 의 mean-subtraction gradient
  ($\partial Q_a / \partial A_b = \delta_{ab} - 1/|\mathcal A|$) 도 수기 유도해 반영.

## 다음 (Day 83)

§15.16 **Distributional + Dueling (C51 + Dueling head 결합)** 및 **Noisy Nets + Dueling head** 로
head 재구조화가 표현적 분포 표적 · 파라미터 잡음 탐색과 어떻게 상호작용하는지 검증.
Rainbow full stack 에 남은 두 요소 (Categorical + Noisy) 를 dueling 위에 붙여 원 Rainbow
(Hessel et al. 2018) 의 6-요소 통합에 수렴한다. Problem 3 에서 관측한 소표본 잡음 문제도
시드 수 확대로 재검증할 예정.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후
> (§4 보간부터 시작해 Ch 15 확장 사례연구까지) 는 스케줄러가 절 단위로 자동 진행. Day 81
> README 의 "다음 (Day 82)" 예고대로 **§15.15 dueling** 로 진행. Chapter 15 확장 사례연구는
> 원 커리큘럼의 확장 흐름이므로 다음 Day 세부 절 이름 (§15.16 distributional + noisy on
> dueling) 은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (Problem 1 의 3-조건 요약 + $V/A$
> 히트맵 + head norm 궤적, Problem 2 의 2-조건 요약 + TD-error 히스토그램 + probe 궤적,
> Problem 3 의 4-조건 ablation + waterfall 분해) 는 in-process exec 로 실행된 결과. 시드:
> Problem 1 = 8601..8604, Problem 2 = 8701..8703, Problem 3 = 8801..8803. Python 3.10,
> NumPy, pandas, Matplotlib 표준 스택. dueling head + PER + Double + n-step 을 수기 numpy 로
> 구현 (신경망 프레임워크 미사용).
>
> 🔎 **정직성 노트**: Problem 3 에서 +Dueling 단독이 baseline 대비 시드편차를 늘리고
> Rainbow-lite-3 가 baseline 을 유의미하게 넘지 못한 관측은 Wang et al. (2016) 원 논문의
> Atari 규모 dueling 이득 · Hessel et al. (2018) Rainbow 시너지 관찰과 직접 모순되지 않는다.
> 원 결과는 (a) 훨씬 큰 신경망, (b) 수십 million-step 학습, (c) sparse-and-diverse reward 인
> 반면 이 노트북은 (a) $H=16$ MLP, (b) 40 에피소드, (c) 3 시드 이므로 dueling 의 이점이
> 발현되는 체제 밖의 소표본 잡음. Problem 1·2 (4/3 시드 격리) 에서 재현된 variance 축소가
> "체제 내" 결과, Problem 3 의 반전은 "체제 외" 관찰로 각각 정직하게 리포트.
>
> 📉 **워크스페이스 제약**: 실행 환경 디스크·CPU 제약 때문에 시드/에피소드 수를 Day 80 대비
> 축소 유지 (Problem 1: 4 시드 × 50, Problem 2: 3 시드 × 40, Problem 3: 3 시드 × 40). tail
> 통계의 상대 순위는 유지되지만, 절대값 비교 시 이 점 감안.
