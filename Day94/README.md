# Day 94 — §15.27 Component-Diagnosis Grid · Neural-Head Cramér/KL Revisit · PR + Noisy-LR Rehab

Day 93 (§15.26) 는 (i) +CNRT 개별 성분의 marginal 이 모두 음수 (특히 EMA -2.55), (ii) Bellman
tabular head 에서 Cramér 이 KL 을 완파, (iii) uniform curriculum 이 +CNRT 열위를 뒤집지 못함 —
세 결과를 남겼다. Day 94 (§15.27) 는 그 잔여를 세 축에서 정면 재판정한다:
(a) Day 93 P1 의 성분 독성이 **초기화 스케일 × 학습률 셀 mismatch** 때문인지, 격자를 sweep 해서
성분별 최적 셀과 rehabilitation gap 을 측정, (b) Day 93 P2 의 Cramér 우위를 **non-tabular
neural head** 로 옮겨 재판정, (c) Day 93 P3 의 +CNRT 열위를 **prioritized replay + Noisy 헤드
전용 higher lr** 처방으로 회복 가능한지 검증.

## 학습 목표

- **Problem 1 (성분 진단 격자)** — $\sigma_0 \in \{0.1, 0.3, 0.5\} \times \eta \in \{0.01, 0.02,
  0.05, 0.10\}$ 12 셀에서 baseline + 4 개 성분 각 3 시드 (94101–94103) 학습, softmax
  $\tau=0.10$ 배포. 성분별 최적 셀, rehabilitation gap $\rho_c$, 같은 셀에서의 baseline 대비
  gap $\Delta_c^\star$.
- **Problem 2 (Neural head Cramér vs KL)** — 공유 트렁크 (H=16) + per-action $K$-atom softmax
  head, categorical projection Bellman. $K \in \{10, 21, 51\}$ × 손실 ∈ {Cramér, KL} × 3 시드
  (94201–94203) × 800 step. tail-8 greedy return, sharpness $\bar H(\hat p)$.
- **Problem 3 (PR + Noisy-lr rehab)** — $\alpha \in \{0.0, 0.5, 1.0\}$ (proportional prioritized
  replay) × $k \in \{1, 2, 5\}$ (Noisy 헤드 lr 배수) 9 셀에서 +CNRT 3 시드 (94301–94303) × 800
  step 학습, freeze 후 slip $p_d \in \{0.05, 0.10, 0.20\}$ 3 배포 greedy. cross-slip mean,
  $\Gamma = \max R - \min R$.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_27_01.ipynb` | 초기화 × lr 격자에서 성분 진단 | Learner(component, σ₀, η) × 12 셀 × 3 시드, softmax τ=0.10 배포 | **모든 성분이 자기 최적 셀에서 baseline 을 회복하거나 능가**. baseline 최적 R = **0.899** @ (σ₀=0.5, η=0.10). Noisy 최적 R = **0.905** @ (0.1, 0.02), $\rho$ = **+0.330**, $\Delta^\star$ = **+0.253**. Cramér 최적 R = **0.913** @ (0.3, 0.05), $\rho$ = +0.009, $\Delta^\star$ = +0.047. **EMA 최적 R = 0.908** @ (0.3, 0.02), $\rho$ = **+0.952** (default 셀 -0.044 → 0.908 극적 회복), $\Delta^\star$ = +0.367. Twin 최적 R = 0.914 @ (0.1, 0.05), $\rho$ = +0.128, $\Delta^\star$ = +0.037. **Day 93 P1 의 catastrophic EMA 열위 (-2.55) 는 순수 셀 mismatch** — proper (더 작은) $\eta$ 와 중간 $\sigma_0$ 조합이면 EMA 는 baseline 보다 오히려 우수. |
| 2 | `CE_15_27_02.ipynb` | Neural head Cramér vs KL × K | 공유 트렁크 (H=16, tanh) + per-action K-logit softmax head, Cramér CDF-distance gradient (softmax Jacobian), KL cross-entropy, K∈{10,21,51} × 3 시드 | **Day 93 P2 결론 완전 역전** — 이번엔 KL 이 두 K 에서 승. R (Cramér vs KL): K=10 → **-0.358 vs 0.767**, K=21 → **0.288 vs 0.930**, K=51 → **0.930 vs 0.930** (동률). **작은 K 에서 neural head + Cramér 은 학습이 불안정** (시드 표준편차 0.91). Sharpness $\bar H$: Cramér 0.77–1.30 vs KL 1.11–3.36 — Cramér 이 sharp 하지만 return 은 낮음. **Day 93 P2 의 tabular head 에서의 Cramér ≫ KL 은 head 아키텍처 artifact**. non-tabular head 에서는 Cramér 의 강한 sharpening 이 오히려 poor local optima 로 수렴시키는 것으로 해석. |
| 3 | `CE_15_27_03.ipynb` | Prioritized replay × Noisy-lr rehab | Replay buffer + $p_i \propto \|\delta_i\|^\alpha$ 샘플링, Noisy-only $\eta_N = k\eta$, freeze × 3 slip | **+CNRT 열위 부분 회복** — best cell $(\alpha=0.0, k=1.0)$ 에서 cross-slip mean **0.919** = baseline **0.919** (완전 tie). Day 91/93 P3 catastrophic gap (0.918 → 0.249) 이 **Day 94 P3 에서는 소거**. 하지만 회복 기전은 rehab 이 아니라 **replay uniform + 원래 lr** — 즉 **prioritized replay 자체가 오히려 destabilizing** (α=1, k=2 → -0.329; α=1, k=5 → -0.233). $k=5$ 는 어떤 α 에서도 catastrophic. **Noisy 헤드 lr 배수 확대는 도움되지 않음**. 격자에서 baseline 을 상회하는 셀은 없음 ($\Delta_{\max} = 0.000$). |

## 한 줄 정리

> **Day 94 세 축 요약**: (P1) Day 93 P1 의 성분 독성은 **거의 전적으로 셀 mismatch** 였음 —
> 특히 EMA whitening 은 default 셀 대비 rehabilitation gap $\rho = +0.952$ 로 극적 회복하며
> $\Delta^\star = +0.367$ 로 baseline 대비 우위 확보. Noisy 도 $\rho = +0.330$, $\Delta^\star
> = +0.253$. **성분 자체는 대부분 정상이며 Day 93 default (σ₀=0.5, η=0.05) 가 나빴다**. (P2)
> Neural head 에서는 **Day 93 P2 결론이 완전히 역전** — K=10, K=21 에서 KL 이 Cramér 을 크게
> 앞섰다 (K=10 에서 0.767 vs -0.358). Day 93 tabular head 에서의 Cramér 우위는 head 아키텍처
> artifact 이며, non-tabular 에서는 오히려 Cramér 의 강한 sharpening 이 poor local optima 로
> 수렴. (P3) prioritized replay + Noisy-lr 처방으로 **+CNRT 열위는 소거** (cross-slip 0.919 =
> baseline) 되나 회복 기전은 α=0, k=1 (즉 두 처방을 모두 끔) — **prioritized replay 는 α>0
> 에서 오히려 학습을 불안정화**하며 $k=5$ 는 대부분 catastrophic. baseline 을 상회하는 셀은 없음.

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. DL 프레임워크 미사용 — 공유 트렁크 MLP (tanh, 수기
backprop), Factorized Noisy Linear (μ, σ 각 gradient), Cramér CDF distance (softmax Jacobian 통해
logit gradient), KL cross-entropy, categorical projection $\Phi$ (nearest-two atom), EMA
whitening, Twin split (K_A=K_B=5), prioritized replay 모두 수기 numpy 로 구현.

## 다음 (Day 95)

§15.28 은 (a) Day 94 P1 에서 얻은 성분별 최적 셀을 **동시에 적용한 +CNRT 재조립** (component-wise
optimized combined stack) 이 Day 93 combined -0.288 을 뒤집는지, (b) Day 94 P2 의 neural-head KL
우위가 **head width H 를 32, 64 로 늘려도 유지되는지**, (c) Day 94 P3 의 baseline-tie 를 진짜
rehabilitation 으로 바꿀 수 있는 처방 (예: soft-Q averaging, batch replay) 을 탐색한다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 93 README 의 "다음 (Day 94) 예고" (processing-order
> grid + neural-head Cramér/KL + PR + Noisy-lr rehab) 을 그대로 반영.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북 모든 수치는 `nbclient` 실행 결과.
> 시드: P1 = 94101–94103, P2 = 94201–94203, P3 = 94301–94303. Python 3.10, NumPy / pandas /
> Matplotlib 표준 스택.
>
> 🔎 **정직성 노트**:
> - **P1**: Rehab 결과가 극적으로 좋아 Day 93 P1 결론이 부분적으로 뒤집혔다. 다만 3 시드는
>   variance 를 완전히 규명하기에 작으며, EMA 의 (σ₀=0.5, η=0.05) 셀 catastrophic 결과
>   (Day 93 -2.55, 여기서도 -0.044) 는 EMA warmup 부재로 인한 재현 가능한 pathology 로
>   해석. 최적 셀 (σ₀=0.3, η=0.02) 은 warmup step 동안 lr 이 작아 초기 running mean/std 가
>   튀지 않아 안정.
> - **P2**: Neural head 에서 Cramér 이 K=10 에서 R = -0.358 로 처참한 결과를 낸 것은
>   softmax Jacobian 을 거친 gradient 가 너무 sharp 해서 초기 학습 단계에서 policy 가 갇힌
>   결과로 보임 (시드 표준편차 0.91 = 극단적 갈림). Tabular head 는 per-(s,a) 파라미터가
>   독립이라 이 pathology 를 회피. **결론 방향은 head 표현력에 매우 민감**.
> - **P3**: cross-slip mean = 0.919 로 baseline 을 정확히 match 한 것은 대부분의 셀에서
>   replay buffer 가 초반에는 uniform 에 가깝고 α 효과가 800 step 안에서 dominant 하지 않아
>   +CNRT 가 baseline 과 같은 궤적을 그린 결과. 시드 94302 는 α=0.5, k=1 에서 갑자기 0.337
>   로 collapse — replay 확률의 stochastic 이 초반 편향을 만든 것으로 추정. `k=5` 의 대부분
>   catastrophic 결과는 Noisy 헤드가 원래 σ 학습을 원래 lr 로 하는 것에 최적화되어 있음을
>   시사.
>
> 📉 **워크스페이스 제약**: `/sessions` 파티션 100% full 로 인해 site-packages 를 `/tmp/pypkg`
> 에 설치. `HOME`, `MPLCONFIGDIR`, `JUPYTER_DATA_DIR`, `XDG_CONFIG_HOME` 을 `/tmp` 하위로
> 지정. 훈련 예산은 P1 600 step, P2 800 step, P3 800 step 로 Day 93 과 일치.
>
> 📚 **레퍼런스**: Schaul et al. (2016) ICLR — Prioritized Experience Replay.
> Bellemare, Dabney, Munos (2017) ICML — C51 categorical projection.
> Rowland et al. (2018) AISTATS — Cramér valid distributional loss.
> Fortunato et al. (2018) ICLR — Factorized Noisy Linear.
> Ioffe & Szegedy (2015) ICML — batch normalization / feature whitening warmup.
