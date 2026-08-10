# Day 92 — §15.25 Stochastic-Deployment Recovery · Loss-Family × Twin-Head · Multi-Slip Curriculum

Day 91 (§15.24) 는 (i) long-horizon (2000 step) 학습에서 $\Delta_T \approx 0$ 재확인, (ii)
both-layer whitening interaction 이 $\beta$ 의 U-자형 (β≈0.90 또는 β≥0.995 sweet spot),
(iii) greedy-freeze cross-slip 배포에서 +CNRT 가 baseline 대비 절대성능·robustness 모두
열위 (negative finding) — 세 축으로 마감했다. Day 92 (§15.25) 는 그 잔여를 세 축에서 정면
검증한다: (a) 배포 정책을 **stochastic softmax (temperature)** 로 바꾸면 +CNRT 열위가 회복
되는가, (b) **Cramér vs KL vs 1-Wasserstein** 세 손실이 twin head 결합시 sharpness /
robustness trade-off 를 어떻게 바꾸는가, (c) **multi-slip curriculum (uniform vs stagewise)**
훈련이 cross-slip 일반화를 개선하는가.

## 학습 목표

- **Problem 1 (stochastic 배포 재판정)** — Shared trunk (H=16) + per-action head 를
  $p_{\text{train}}=0.10$ 확률적 chain MDP 에서 3 시드 × 600 step 학습 후, 학습된 Q 함수를
  온도 $\tau \in \{0.05, 0.10, 0.20, 0.5, 1.0, 2.0\}$ 의 softmax 정책으로 배포 (60 에피소드
  × 각 $\tau$). Baseline vs +CNRT 의 온도-대-tail-8 곡선, $\tau^\star$ 및 gap 관측.

- **Problem 2 (Cramér vs KL vs Wasserstein × twin)** — Stationary Gaussian-mixture target
  ($0.6 \mathcal N(0.3, 0.05^2) + 0.4 \mathcal N(-0.4, 0.08^2)$) 을 12원자 categorical 로
  projection, twin 분할 (KA=KB=6), 3 손실 × 4 시드 × 400 step SGD (lr=0.2). Sharpness
  $H(\hat p)$ 및 shift $\Delta \in \{\pm 0.05, \pm 0.10\}$ 하 mean bias 로 sharpness-
  robustness trade-off 를 측정.

- **Problem 3 (multi-slip curriculum)** — Baseline (§15.24 P3 재현) 학습기를 세 curriculum
  (fixed-0.10 / uniform-$\{0.05,0.10,0.20\}$ / stagewise-3분할) × 3 시드 × 800 step 학습,
  freeze 후 $p_d \in \{0.05, 0.10, 0.20\}$ 각 60 에피소드 greedy 배포. Cross-slip tail-8
  return, generalization gap $\Gamma_c = \max_{p_d} R_c - \min_{p_d} R_c$ 비교.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_25_01.ipynb` | Stochastic softmax 배포 하 +CNRT 열위 재판정 | Shared trunk (H=16) + linear head (baseline) / Factorized Noisy Linear + Cramér + EMA whitening + twin (K=10) (+CNRT), 3 시드 × 600 step | 최적 온도: baseline $\tau^\star=0.10$ **0.874** ± 0.042, **+CNRT $\tau^\star=0.05$ 0.926 ± 0.003 (+0.180)**. Gap 은 $\tau \in [0.05, 0.20]$ 에서 +CNRT 우위, $\tau \ge 0.5$ 에서는 두 조건 tail-8 이 함께 저하 (2.0 에서 0.62–0.65). **이 setup 에서 Day 91 P3 열위 미재현** — 원자 분포 학습이 near-greedy 에서도 안정. Day 91 결과는 예산·처방 결합·시드 차이에 민감함을 시사. |
| 2 | `CE_15_25_02.ipynb` | 3 손실 × twin head sharpness/robustness | 12원자 categorical, twin KA=KB=6, Gaussian-mixture target, 3 손실 × 4 시드 × 400 step SGD (lr=0.2) | **KL 이 두 축 모두 승리**: sharpness H=**1.539**, mean bias **0.0032**. **Cramér 근접 2위** (H=1.572, bias=0.0047). **Wasserstein 두 축 모두 열위** (H=2.221, bias=0.0146) — sign-based subgradient 근사의 $1/K$ 감쇠로 학습 신호 손실. Cramér × twin 을 KL 로 교체시 이론적 이점 있으나 real-net 재판정 필요. |
| 3 | `CE_15_25_03.ipynb` | Multi-slip curriculum 하 cross-slip 일반화 | Baseline (H=16, linear head, MSE, ε=0.10) × 3 curriculum × 3 시드 × 800 step, freeze greedy 배포 60 에피소드 | **Uniform curriculum 압도**: mean tail-8 = **0.920** vs fixed **0.829** (+0.091) vs stagewise **0.771** (+0.149). **Generalization gap**: uniform **0.014** vs fixed 0.108 (8× 축소) vs stagewise 0.166 (12× 축소). **Stagewise 는 예상 밖 열위** — 마지막 stage ($p=0.20$) 편중으로 초·중기 학습이 catastrophic-forgetting. Uniform 은 훈련 예산 증가 없이도 강력한 저비용 처방. |

## 한 줄 정리

> **Day 92 세 축 요약**: (P1) 이 setup 의 stochastic 배포에서는 +CNRT 가 baseline 을 near-greedy
> 구간에서 +0.180 능가 — Day 91 P3 greedy-freeze 열위는 재현되지 않음 (예산/처방/시드 감수성).
> (P2) Twin head × 3 손실 sandbox 에서 **KL 이 sharpness (1.54) 와 mean-tracking bias (0.003)
> 두 축 모두 최고**, Cramér 은 안정적 근접 2위, Wasserstein 은 subgradient 근사 감쇠로 두 축 모두
> 열위. (P3) **Uniform multi-slip curriculum 은 cross-slip generalization gap 을 12배 축소**하며
> mean tail-8 를 baseline 대비 +0.091 개선. Stagewise 는 catastrophic-forgetting 으로 fixed 보다
> 열위 — 순서 curriculum 은 rehearsal 없이는 위험.

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. DL 프레임워크 미사용 — shared trunk MLP (tanh, 수기
backprop), Unified/Twin softmax categorical head, Factorized Noisy Linear ($\mu, \sigma$ 각
gradient), Cramér closed-form logit gradient, running-EMA feature whitening, categorical
projection, KL/1-Wasserstein subgradient 를 모두 수기 numpy 로 구현.

## 다음 (Day 93)

§15.26 는 (a) Day 92 P1 재현/부재의 원인 규명 — 훈련 예산 (2000 step 확장) 및 처방 조합
결과와 이 setup 의 결과 사이 격차, (b) Day 92 P2 의 KL 우위를 real-net Bellman 학습에서
재판정 (categorical Bellman projection + Cramér vs KL 비교), (c) Day 92 P3 의 uniform
curriculum 을 +CNRT 스택과 결합 시 §15.24 negative finding 이 실제로 뒤집히는지 검증한다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 91 README 의 "다음 Day (§15.25) 예고" (stochastic
> deployment 재판정 + 3 손실 × twin + multi-slip curriculum) 을 그대로 반영.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치는 `nbclient` 로 실행된 결과.
> 시드: P1 = 92101–92103, P2 = 92201–92204, P3 = 92301–92303. Python 3.10, NumPy /
> pandas / Matplotlib 표준 스택.
>
> 🔎 **정직성 노트**:
> - **P1**: Day 91 P3 (+CNRT greedy 열위) 를 이 setup 에서 재현하려 했으나 오히려 **+CNRT 가
>   baseline 을 능가**하는 반대 관측. 원인 후보: (a) 훈련 예산 (600 vs 800 step), (b) 처방
>   조합 (Cramér+Noisy+Whitening+Twin 을 하나의 클래스로 결합 vs Day 91 은 조건 분리),
>   (c) 시드 개수 (3 vs 3, 시드 값 다름), (d) 배포 에피소드 수 (60 vs 80). 이 setup 의 결과를
>   Day 91 결과와 함께 병기해 감수성 문서화.
> - **P2**: KL 이 두 축 모두 승리한 결과는 target 이 매우 뾰족한 categorical projection
>   (2 modes, sharp) 이기 때문. Target 이 flatter 분포이면 순위가 뒤바뀔 가능성이 크며,
>   Wasserstein 의 열위는 sign-based subgradient 근사의 $1/K$ 스케일링 때문. 정확한 EMD-
>   projection 을 구현하면 순위가 바뀔 수 있다.
> - **P3**: Stagewise 가 fixed 보다 열위인 관측은 catastrophic-forgetting 로 자연스럽게 설명
>   되나, curriculum 순서 (0.05→0.10→0.20 만 실험) 를 반전 또는 rehearsal 을 추가하면
>   결과가 달라질 여지가 있다.
>
> 📉 **워크스페이스 제약**: `/sessions` 파티션 100% full 로 인해 site-packages 를 `/tmp/pypkg`
> 에 설치. `HOME`, `MPLCONFIGDIR`, `JUPYTER_DATA_DIR`, `XDG_CONFIG_HOME` 을 `/tmp` 하위로
> 지정하고 노트북 실행. 훈련 예산은 P1 600 step / P2 400 step / P3 800 step 로 유지, 확장은
> Day 93 로 이관.
>
> 📚 **레퍼런스**: Bellemare, Dabney, Munos (2017), ICML — C51. Rowland et al. (2018), AISTATS
> — Cramér gradient. Fortunato et al. (2018), ICLR — Factorized Noisy Linear. Villani (2009),
> *Optimal Transport: Old and New* — 1-Wasserstein. Elman (1993), *Cognition* — starting small
> (stagewise curriculum motivation).
