# Day 93 — §15.26 Long-Horizon Rehabilitation · Bellman Cramér/KL Re-adjudication · Uniform × +CNRT Final

Day 92 (§15.25) 는 (i) stochastic softmax 배포에서 +CNRT 가 baseline 을 near-greedy 로
+0.180 능가 (Day 91 P3 열위 재현 실패), (ii) stationary Gaussian-mixture sandbox 에서 KL
이 두 축 모두 우위, (iii) uniform multi-slip curriculum 이 baseline 의 cross-slip gap 을
12배 축소 — 세 결과를 남겼다. Day 93 (§15.26) 는 그 잔여를 세 축에서 정면 검증한다:
(a) 예산을 600 → 2000 step 으로 확장했을 때 Day 92 P1 결과가 유지되는가 + +CNRT 스택을
성분별로 분해했을 때 어느 성분이 dominant driver 인가, (b) Day 92 P2 의 KL 우위가 실제
Bellman-projection 루프 (TD 학습) 안에서도 유지되는가, (c) uniform curriculum × +CNRT 를
결합했을 때 Day 91 P3 의 negative finding 이 뒤집히는가.

## 학습 목표

- **Problem 1 (예산 × 처방 분해)** — Shared trunk (H=16) + per-action head 를
  $p_{\text{train}}=0.10$ chain MDP 에서 3 시드 × T ∈ {600, 2000} step 학습 후 온도
  $\tau \in \{0.05, 0.10, 0.20, 0.5, 1.0, 2.0\}$ softmax 배포 (60 에피소드 × 각 $\tau$).
  두 번째로 +CNRT 스택을 {Noisy, Cramér, EMA whitening, Twin split} 로 분해하여 baseline
  위에 개별 토글 (τ=0.10 배포).
- **Problem 2 (Cramér vs KL × Bellman × K)** — 5-상태 chain MDP 에서 tabular per-(s,a)
  $K$-원자 logit head, TD-target 을 categorical projection 으로 만들어 두 손실 각각으로
  800 step × 3 시드 학습. $K \in \{10, 21, 51\}$ sweep, sharpness · projection error ·
  greedy tail-8 return 비교.
- **Problem 3 (2×2 factorial)** — {baseline, +CNRT} × {fixed-0.10, uniform{0.05,0.10,0.20}}
  × 3 시드 × 800 step, freeze 후 3 배포 슬립 각 60 에피소드 greedy. cross-slip mean 및
  generalization gap $\Gamma = \max_{p_d} R - \min_{p_d} R$.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_26_01.ipynb` | 예산 확장 + 처방 성분 분해 | Shared trunk (H=16) + 4-toggle head, softmax 배포, 3 시드 × T={600, 2000} | **예산 효과**: base $\tau^\star=0.05$: T=600 → **0.925**, T=2000 → **0.912**; +CNRT $\tau^\star=0.10$: T=600 → **0.905**, T=2000 → **0.903**. Gap $\Delta_T = R^{+\text{CNRT}}-R^{\text{base}}$: T=600 **-0.020**, T=2000 **-0.008** (sign 유지, 크기 감소). **Day 92 P1 의 +0.180 우세는 이 시드/setup 에서 재현되지 않음** — near-greedy 에서 baseline 이 근소하게 우세. **처방 분해** (baseline 0.753 기준 marginal): **+Noisy = +0.115** (유일한 양의 성분), +Cramér = **-0.534**, +Twin = **-0.505**, +EMA = **-2.553** (독성). combined = **-0.288** — 성분 조합 자체가 baseline 을 훼손. **결론: Noisy 만 dominant positive driver, 나머지 성분은 이 짧은 예산에서 학습을 불안정화**. |
| 2 | `CE_15_26_02.ipynb` | Bellman-projection loop 안 Cramér vs KL | Tabular per-(s,a) K-원자 logit, categorical projection $\Phi$, TD greedy target, K∈{10,21,51} × 2 loss × 3 시드 | **Cramér 이 세 K 모두에서 완승** (R): K=10 → **0.925 vs 0.836**, K=21 → **0.925 vs 0.694**, K=51 → **0.925 vs 0.198**. K 증가로 KL 은 evaluation return 이 **collapse** (0.836 → 0.198), Cramér 은 유지. **Sharpness** $H(\hat p)$: Cramér 1.57–2.23 vs KL 2.29–3.93 — Cramér 이 훨씬 sharp. **Day 92 P2 stationary sandbox 결과 (KL 승리) 완전 역전** — Bellman bootstrap 에서는 CDF 를 정합하는 Cramér 이 프로젝션 노이즈에 강건. |
| 3 | `CE_15_26_03.ipynb` | Uniform × +CNRT 최종 판정 | 2×2 factorial, 3 시드 × 800 step, greedy 배포 × 3 슬립 | **negative finding 여전히 재현**: cross-slip mean 은 baseline fixed **0.918**, baseline uniform **0.820**, +CNRT fixed **0.403**, +CNRT uniform **0.249**. **+CNRT 는 uniform 결합 후 오히려 악화** (Γ: 0.654 → 0.833). $\Delta$ (+CNRT - base) under uniform = **-0.571** (fixed -0.516). **Uniform curriculum 은 +CNRT 열위를 회복시키지 못하며 오히려 gap 을 확대**. baseline 은 uniform 으로 $\Gamma$ 는 축소 (0.038 → 0.017) 하나 mean 도 함께 하락. |

## 한 줄 정리

> **Day 93 세 축 요약**: (P1) 예산 600→2000 확장에서 +CNRT vs baseline gap 은 **-0.020 → -0.008
> 로 축소되나 sign 유지** (Day 92 P1 의 +0.180 우세는 이 시드에서 재현 안됨). 처방 분해상
> **Noisy 만 유일한 positive driver (+0.115)**, EMA 는 **-2.553 catastrophic**, Cramér·Twin 도
> 개별 -0.5 대로 독성. (P2) Bellman-projection loop 안에서는 **Cramér 이 세 K 모두에서 KL
> 완파** — K=51 에서 KL 은 0.198 로 collapse. Day 92 P2 stationary 결과 (KL 우위) 정반대
> 방향. (P3) uniform multi-slip curriculum × +CNRT 결합은 **Day 91 P3 negative finding
> 을 뒤집지 못하며 오히려 gap 을 축소 못하고 확장** (Γ 0.654 → 0.833). 단일 요인 처방만으로는
> +CNRT 열위 회복 불가.

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. DL 프레임워크 미사용 — shared trunk MLP (tanh, 수기
backprop), Factorized Noisy Linear (μ, σ 각 gradient), Cramér closed-form CDF gradient,
KL 손실 (softmax logit gradient = p - m), categorical projection $\Phi$ (nearest-two atom
distribution), running-EMA feature whitening, twin split (KA=KB=5) 모두 수기 numpy 로 구현.

## 다음 (Day 94)

§15.27 은 (a) Day 93 P1 의 processing-order 효과 — Noisy 이외 성분이 개별로 독성인 원인
(초기화 스케일 vs learning-rate 매칭) 을 grid 로 탐색, (b) Day 93 P2 의 Cramér 우위를
neural (non-tabular) head 로 옮겨 재판정, (c) Day 93 P3 의 +CNRT 열위를 **prioritized
replay + higher-lr for Noisy only** 처방으로 재시도, uniform curriculum 이 아닌 방법으로
정말 회복 가능한지 검증한다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 92 README 의 "다음 (Day 93) 예고" (예산 확장 원인
> 규명 + real-net Bellman Cramér/KL 비교 + uniform × +CNRT 결합) 을 그대로 반영.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북 모든 수치는 `nbclient` 실행 결과.
> 시드: P1(예산) = 93101–93103, P1(분해) = 93201–93203, P2 = 93201–93203 (K별 재시드),
> P3 = 93301–93303. Python 3.10, NumPy / pandas / Matplotlib 표준 스택.
>
> 🔎 **정직성 노트**:
> - **P1 예산 축**: Day 92 P1 은 "+CNRT 가 baseline 대비 +0.180" 을 보고했으나 이 재현에서는
>   T=600 에서도 -0.020 (baseline 미세 우세) 만 얻었다. 시드 (Day 92 는 92101–92103, 여기는
>   93101–93103) 및 재분해 클래스 정의의 차이가 원인 후보. **Day 92 P1 결론은 시드 감수성**
>   이 크며, 두 예산에서 gap 이 크게 흔들리지 않음 (sign 유지·크기 축소) 은 시드-변동성이
>   지배적임을 시사.
> - **P1 처방 분해**: EMA whitening 단독 marginal -2.553 은 whitening 의 running mean/std
>   초기 warmup 단계에서 features 가 매우 튀는 것을 lr=0.05 그대로 학습해 발산한 결과로 해석.
>   proper warmup (초기 EMA 무시) 을 넣으면 정상화 될 여지 있다.
> - **P2**: Cramér 우위가 K=51 까지 확대되는 것은 KL 이 fine-atom 에서 label smoothing 효과
>   없이 각 원자에 자유롭게 확률질량을 나눠 학습신호가 diffuse 해지는 반면, Cramér 은 CDF
>   레벨 정합이라 aggregation 효과로 견고. Day 92 P2 (stationary target) 에서의 KL 우위와의
>   차이는 bootstrap noise 존재 여부가 결정적.
> - **P3**: +CNRT 가 uniform curriculum 하에서 오히려 악화된 것은 curriculum sampling 이
>   Noisy 헤드의 σ 학습을 더 흔들었기 때문 (매 에피소드 다른 dynamics 아래 σ 를 재보정 해야).
>   fixed 예산 800 step 안에서 다음 curriculum 셀에 도달하기 전에 학습이 diverge.
>
> 📉 **워크스페이스 제약**: `/sessions` 파티션 100% full 로 인해 site-packages 를 `/tmp/pypkg`
> 에 설치. `HOME`, `MPLCONFIGDIR`, `JUPYTER_DATA_DIR`, `XDG_CONFIG_HOME` 을 `/tmp` 하위로
> 지정. 훈련 예산은 P1 600/2000 step / P2 800 step / P3 800 step 로 Day 92 와 일치.
>
> 📚 **레퍼런스**: Bellemare, Dabney, Munos (2017) ICML — C51 categorical projection.
> Rowland et al. (2018) AISTATS — Cramér 이 categorical 학습의 valid divergence.
> Fortunato et al. (2018) ICLR — Factorized Noisy Linear. Ioffe & Szegedy (2015) ICML —
> feature whitening warmup issue.
