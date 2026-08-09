# Day 91 — §15.24 Long-Horizon Distributional Consolidation · Two-Layer Whitening β-Sweep · Cross-Slip Generalization

Day 90 (§15.23) 는 (i) real-net 4-처방 결합에서 twin head 의 incremental 효과가 0 임을
관측 (+CNR 0.963, +CNRT 0.962), (ii) 2-hidden trunk 의 both-layer whitening 이 marginal
효과 대비 destructive interaction (−0.44) 을 보이는 negative finding, (iii) $p_{\text{slip}}$
sweep 에서 twin+white real-net 이 세 슬립 수준 모두에서 seed_std 를 축소시키는 세 축으로
마감했다. Day 91 (§15.24) 은 그 잔여를 세 축에서 정면 검증한다: (a) 학습 예산을 2000 step
으로 확장했을 때 twin 의 시간의존적 marginal 이 나타나는가, (b) both-layer whitening 의
EMA 감쇠계수 $\beta$ 를 스윕해 destructive interaction 을 완화하는 sweet spot 을 찾을 수
있는가, (c) 훈련-배포 슬립 mismatch 하에서 +CNRT 스택이 baseline 대비 robustness lift 를
유지하는가.

## 학습 목표

- **Problem 1 (long-horizon CNR/CNRT)** — Shared trunk (H=16) + per-action linear head
  (K=11 atoms) 을 stochastic chain MDP ($p_{\text{slip}}=0.10$) 위에서 3 조건 (baseline,
  +CNR, +CNRT) × 4 시드 × 2000 step 학습. 500/1000/1500/2000 step 체크포인트에서 tail-8
  return, seed_std, $\sigma_W$ RMS 를 기록해 twin head 의 **시간의존적 marginal utility**
  를 측정.

- **Problem 2 (both-layer whitening β sweep)** — 2-hidden trunk ($H_1=16, H_2=12$, tanh)
  을 deterministic chain MDP 에 배치하고 whitening 조건 {none, L1_only, L2_only, both} 를
  운영. **both 조건만** $\beta \in \{0.90, 0.95, 0.99, 0.995\}$ 스윕. Interaction
  $\Delta_{\text{int}}(\beta) = \bar R_{\text{both}}(\beta) - \bar R_{\text{none}} - \Delta_{L_1}
  - \Delta_{L_2}$ 를 β 함수로 관측.

- **Problem 3 (cross-slip generalization)** — baseline / +CNRT 를 $p_{\text{train}}=0.10$
  으로 3 시드 × 800 step 학습, 학습된 정책을 **freeze** 하여 $p_{\text{deploy}} \in
  \{0.05, 0.10, 0.20\}$ 각각에서 greedy 배포 (80 에피소드). Deployment tail-8 return 과
  매칭 (matched=0.10) 대비 저하량을 조건별 비교.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_24_01.ipynb` | Long-horizon (2000 step) tail-8 vs 3 조건 | ε-greedy baseline, Cramér closed-form gradient + Noisy Linear + EMA whitening, twin-half EMA | 세 조건 모두 tail-8 ≈ 0.90 근처 수렴. **$\Delta_{CNR}$: {+0.006, −0.001, −0.001, +0.024}**, **$\Delta_T$: {−0.022, +0.009, −0.024, −0.003}** — 노이즈 수준으로 $\Delta_T ≈ 0$ 재확인. 오히려 **+CNRT 는 tail 편차 확대** (seed_std max 0.091 at t=1500) — twin split 후 half-K=6 EMA 가 산발적 진동 유발. |
| 2 | `CE_15_24_02.ipynb` | 2-hidden trunk both-layer whitening β 스윕 | 4 조건 × 4 시드 × 400 step, β ∈ {0.90, 0.95, 0.99, 0.995} 만 both 에 적용 | **Interaction 은 β 에 U-자형**: β=0.90 (τ=10) → **−0.001** (안전), β=0.95 → **−0.358**, β=0.99 → **−0.304** (Day 90 관측 −0.44 재현), β=0.995 (τ=200) → **+0.003** (사실상 no-op). Sweet spot 은 **β≈0.90** (짧은 EMA 로 층 간 상호간섭 시간 부족) 또는 **β≥0.995** (whitening 자체가 상수 divide). |
| 3 | `CE_15_24_03.ipynb` | Cross-slip generalization (train p=0.10, deploy {0.05, 0.10, 0.20}) | freeze-and-deploy, greedy policy, 80-episode 배포 | **Baseline 이 세 슬립 모두 압도적 우위** (0.90 이상), **+CNRT 는 0.49–0.55 로 정체**. 슬립 mismatch 자체는 두 조건 모두 작은 영향 (baseline max 저하 −0.014). **Seed std**: baseline 0.01–0.03, +CNRT 0.62–0.71 — **negative finding**: greedy 배포 + Noisy scale=0 조건에서 +CNRT 스택은 학습 정책을 sub-optimal 로 고정. §15.23 real-net 우위는 배포시 exploration/noise 유지 가정에 의존. |

## 한 줄 정리

> **Long-horizon 에서도 $\Delta_T ≈ 0$ 재확인** (P1), **both-layer whitening interaction 은 β 의
> U-자형** — β≈0.90 또는 β≥0.995 로 옮기면 결합 사용 안전 (P2), **greedy 배포 하에서
> +CNRT 는 baseline 대비 절대성능·robustness 모두 열위** — Noisy Linear 처방이 학습 정책을
> sub-optimal 로 고정 (P3, negative finding).

## 다음 Day (§15.25) 예고

- P3 negative finding 재판정 — 배포시 stochastic policy (temperature-softmax) 또는 훈련시
  exploration 스케줄 unify 하에서 +CNRT 우위가 회복되는지.
- Cramér vs KL vs Wasserstein 세 손실이 twin head 와 결합될 때의 sharpness / robustness
  trade-off.
- Multi-slip curriculum (uniform vs stagewise) 훈련이 cross-slip 일반화를 어떻게 개선하는지.
