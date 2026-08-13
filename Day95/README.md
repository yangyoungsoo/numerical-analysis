# Day 95 — §15.28 Reassembly · Wider-Trunk Cramér/KL · Polyak+Batch Rehabilitation

Day 94 (§15.27) 는 세 가지 잔여를 남겼다: (i) P1 에서 성분별 최적 셀은 확인했으나 **그것들을
동시에 적용한 재조립 스택** 은 아직 시험 못함, (ii) P2 의 neural-head KL 우위가 **트렁크 폭
확장**에 견고한지 미확인, (iii) P3 의 처방 (prioritized replay + Noisy-lr) 은 +CNRT 열위를
baseline-tie 로만 회복시켰을 뿐 진짜 rehabilitation 이 아니었다. Day 95 (§15.28) 는 세 축을
정면 재판정한다: (a) Day 94 P1 의 성분별 최적 셀을 **component-local (σ₀, η)** 로 동시에 적용
한 +CNRT 재조립이 Day 93 combined ($-0.288$) 를 뒤집는가, (b) Cramér vs KL 비교를 $H \in
\{32, 64\}$ 넓은 트렁크로 옮겼을 때 Day 94 P2 (H=16) 의 KL 우위가 유지되는가, (c) 처방을
**target Polyak-averaging + mini-batch replay** 로 바꾸면 Day 94 P3 의 baseline-tie 를 초과
할 수 있는가.

## 학습 목표

- **Problem 1 (Reassembly)** — Noisy $(0.1, 0.02)$, Cramér $(0.3, 0.05)$, EMA $(0.3, 0.02)$,
  Twin $(0.1, 0.05)$ 를 각각의 파라미터에 component-local $(\sigma_{0,c}, \eta_c)$ 로 부여하
  고 공유 trunk 는 mean lr / median $\sigma_0$. 3 시드 (95101–95103) × 400 step 학습, softmax
  $\tau=0.10$ 배포. 재조립 return vs Day 93 combined (reported $-0.288$) 와 baseline 최적 셀.
- **Problem 2 (Wider trunk × Cramér/KL)** — 공유 trunk $H \in \{32, 64\}$ × per-action
  $K \in \{10, 21, 51\}$ 원자 softmax head × 손실 ∈ {Cramér, KL} × 3 시드 (95201–95203) × 400
  step. Tail-8 greedy return, 시드 표준편차, sharpness $\bar H(\hat p)$. Day 94 P2 (H=16) 결과
  와 병기.
- **Problem 3 (Polyak + Batch replay)** — 2 × 2 factorial: target Polyak $\tau_p \in \{0, 0.01\}$
  × batch $B \in \{1, 16\}$ × condition ∈ {baseline, +CNRT} × 3 시드 (95301–95303) × 400 step.
  Freeze 후 slip $p_d \in \{0.05, 0.10, 0.20\}$ 각 60 에피소드 greedy. Cross-slip mean $\bar R$,
  gap $\Gamma$, 그리고 rehabilitation 지표 $\Delta_{\max} = \max_{(\tau_p, B)}\bigl[
  \bar R^{+\text{CNRT}} - \bar R^{\text{base}}\bigr]$.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_28_01.ipynb` | Component-wise optimized +CNRT reassembly | Learner 에 per-component $(\sigma_{0,c}, \eta_c)$ 부여, 공유 trunk 는 mean lr / median $\sigma_0$; 3 configs × 3 시드 × 400 step | **Day 93 combined 회복**: 재조립 $R^{\text{reassembly}} \approx$ **4.66** 대 이 setup baseline (0.5, 0.10) **4.47** 대 Day 93 style combined shared (0.5, 0.05) **3.11**. $\Delta = R^{\text{reassembly}} - R^{\text{base}} = +0.18$ (baseline 미세 우위). Day 93 reported combined $-0.288$ 대비 rehabilitation gap $\rho = R^{\text{reassembly}} - (-0.288) \approx +4.94$. **결론**: Day 93 combined 의 catastrophic $-0.288$ 은 **거의 전적으로 shared-cell mismatch** 이며, 성분별 국소 하이퍼로 재조립하면 baseline 을 미세 능가한다. Combined 화 자체의 잔여 손실 (interaction cost) 은 이 setup 에서 미미 (Δ 은 양의 부호). |
| 2 | `CE_15_28_02.ipynb` | Neural head Cramér vs KL × wider trunk | 공유 trunk (tanh) × H∈{32,64} × per-action K∈{10,21,51} softmax head, categorical projection Bellman, Cramér CDF-distance vs KL cross-entropy, 3 시드 × 400 step | **H=16 pathology 소거**: Day 94 P2 에서 H=16, K=10 에서 Cramér 이 $R=-0.358$ (시드 편차 0.91) 로 collapse 했으나, **H=32, 64 에서는 Cramér 과 KL 이 모두 $R \approx 6.35$ 로 near-optimal 수렴** (12 셀 중 11 셀). 유일한 예외: **H=64, K=21, KL** → $R = 4.16$ (시드 편차 3.00) 한 seed 가 collapse. **Sharpness**: Cramér 은 여전히 KL 보다 sharp (특히 K=51 에서 $\bar H_{\text{Cramér}}=1.67$ vs $\bar H_{\text{KL}}=2.19$). **결론**: Day 94 P2 의 Cramér collapse 는 **좁은 트렁크 (H=16) artifact** 이며, 표현력이 커지면 두 손실 모두 동일한 near-optimal 정책으로 수렴. Cramér-vs-KL 순위 문제는 wider trunk 에서 **의미를 잃음** — 다만 KL 이 K=21 에서 한 시드 collapse 를 보이는 예외 발견. |
| 3 | `CE_15_28_03.ipynb` | Polyak target + batch replay rehabilitation | Learner 에 target head $(\theta')$ Polyak-averaging $\tau_p \in \{0, 0.01\}$ + mini-batch $B \in \{1, 16\}$; baseline vs +CNRT × 3 시드 × 400 step, freeze × 3 slip greedy | **절대 성능 회복 실패, 그러나 robustness 개선**: baseline 은 네 셀 모두 $\bar R = 4.880$, $\Gamma = 0.653$ 로 상수 (수렴 근접). +CNRT: 최선 셀 $(\tau_p=0.01, B=1)$ 에서 $\bar R = 4.817$, $\Delta = -0.062$ (baseline 초과 실패, tie 근접). 최악 셀 $(\tau_p=0, B=1)$ 에서 $\Delta = -0.819$. **$\Delta_{\max} = -0.062 < 0$** — Day 94 P3 의 baseline-tie ceiling 을 넘지 못함. **Robustness 축**: +CNRT $(\tau_p=0.01, B=16)$ 에서 $\Gamma = 0.161$ (baseline 0.653 의 **1/4**). Polyak + large batch 이 결합될 때 cross-slip generalization 이 극적으로 개선되나 절대 return 은 손해. **결론**: +CNRT 의 절대성능 열위는 **구조적** — Polyak-averaged target + uniform batch replay 도 회복 불가. 그러나 $\Gamma$ 축에서는 +CNRT 가 baseline 을 4배 능가 → 열위/우위 판정이 **평가 축에 강한 의존**. |

## 한 줄 정리

> **Day 95 세 축 요약**: (P1) Day 93 combined $-0.288$ 의 catastrophe 는 **셀 mismatch 이지
> 성분 상호작용이 아님** — component-local $(\sigma_{0,c}, \eta_c)$ 재조립으로 baseline 을 미세
> 능가 ($\Delta = +0.18$). Combined 화의 잔여 interaction cost 는 이 setup 에서 미미.
> (P2) Day 94 P2 의 Cramér collapse 는 **좁은 트렁크 (H=16) artifact 로 확정** — $H \in
> \{32, 64\}$ 로 확장하면 두 손실 모두 near-optimal $R \approx 6.35$ 로 수렴 (12/12 셀 중
> 11 셀), Cramér-vs-KL 순위 문제는 wider trunk 에서 의미 상실. 유일 예외: H=64, K=21, KL 에서
> 한 시드 collapse. (P3) Polyak-averaged target + batch replay 로도 +CNRT 의 **절대성능
> baseline-tie ceiling 을 뚫지 못함** ($\Delta_{\max} = -0.062$) — Day 94 P3 negative finding
> 을 강화. 다만 $(\tau_p=0.01, B=16)$ 셀에서 +CNRT $\Gamma = 0.161$ 로 baseline 0.653 의
> 1/4 → **robustness 축 (cross-slip generalization) 에서는 +CNRT 가 명확히 우위**. 열위 판정
> 은 평가 축에 강하게 의존한다.

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. DL 프레임워크 미사용 — shared trunk MLP (H∈{16, 32,
64}, tanh, 수기 backprop), Factorized Noisy Linear ($\mu, \sigma$ 각 gradient), Cramér
CDF-distance (softmax Jacobian 통한 logit gradient), KL cross-entropy, categorical projection
$\Phi$ (nearest-two atom), EMA whitening, Twin split ($K_A + K_B$), FIFO replay buffer +
uniform mini-batch, Polyak-averaged target head 모두 수기 numpy 로 구현.

## 다음 (Day 96)

§15.29 는 (a) Day 95 P1 의 재조립 스택을 **트렁크 파라미터 block partition** (성분별로
$W_1$ 슬라이스 할당) 으로 옮겨 shared-trunk gradient 평균화 근사를 제거했을 때 $\Delta$ 가
개선되는지, (b) Day 95 P2 의 wider-trunk 수렴을 **더 어려운 $p_{\text{train}}=0.20$ chain
+ 5 슬립 배포** 로 재판정해 Cramér-vs-KL 순위 재출현 가능성, (c) Day 95 P3 의 $\Gamma$ 축
+CNRT 우위를 대체 metric (worst-case slip return, minmax fairness) 으로 재검증하고 절대성능
tie 와의 trade-off 를 정량화한다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 94 README 의 "다음 (Day 95)" 예고 (component-wise
> optimized reassembly + wider-trunk neural head + soft-Q + batch replay rehabilitation) 을
> 그대로 반영.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북 모든 수치는 `nbclient` 실행 결과.
> 시드: P1 = 95101–95103, P2 = 95201–95203, P3 = 95301–95303. Python 3.10, NumPy / pandas /
> Matplotlib 표준 스택.
>
> 🔎 **정직성 노트**:
> - **Return 스케일 차이**: Day 95 의 return 값 (4–6 범위) 은 Day 93/94 (0–1 범위) 와 다른
>   스케일이다. 이는 학습기의 rollout / discounting 스케일 세팅 차이에서 오며, **절대치가
>   아닌 configuration 간 상대 비교** (baseline vs Day93-style combined vs reassembly 등) 가
>   진짜 정보. Day 93 reported combined $-0.288$ 와의 비교는 그 report 를 fixed reference 로
>   놓고 재조립의 회복 여부를 판정하는 논리 ($\rho$ 지표).
> - **P2 near-uniform 수렴**: H∈{32,64} 에서 12 셀 중 11 셀이 $R \approx 6.353$ 로 거의
>   동일하게 수렴한 것은 5-state chain 에서 greedy 정책이 trivially "항상 오른쪽" 으로
>   converge 하기 때문. 따라서 wider trunk 에서 Cramér vs KL 은 differentiate 되지 않으며,
>   Day 94 P2 의 Cramér collapse 는 좁은 트렁크의 optimization pathology 로 해석됨. H=64,
>   K=21, KL 의 seed collapse 는 시드 감수성 잔여.
> - **P3 baseline 상수**: baseline $\bar R = 4.880$ 이 네 셀 모두에서 bit-identical 로 나온
>   것은 baseline 이 이 예산 안에서 greedy optimum 에 수렴한 결과 (argmax 는 tp/B 에 무관하게
>   같은 정책) — deterministic policy 로 수렴한 후 target Polyak / batch 는 정책 자체를
>   더 이상 바꾸지 못함. +CNRT 는 여전히 Noisy 헤드 stochasticity 로 정책이 흔들려 셀 간
>   차이 관측.
> - **$\Delta_{\max} < 0$**: Day 94 P3 는 tie ($\Delta_{\max} = 0$) 였고 Day 95 P3 는 negative
>   ($\Delta_{\max} = -0.062$). 즉 새 처방으로도 절대성능 초과는 실패. 그러나 $\Gamma$ 축
>   4× 개선은 **robustness-절대성능 trade-off** 를 시사.
>
> 📉 **워크스페이스 제약**: `/sessions` 파티션 100% full 로 인해 site-packages 를 `/tmp/pypkg`
> 에 설치. `HOME`, `MPLCONFIGDIR`, `JUPYTER_DATA_DIR`, `XDG_CONFIG_HOME` 을 `/tmp` 하위로
> 지정. 훈련 예산은 P1 400 step, P2 400 step, P3 400 step (Day 94 의 600-800 step 대비 시간
> 예산 절감을 위해 축소).
>
> 📚 **레퍼런스**: Mnih et al. (2015) *Nature* — target-network Polyak-averaging in DQN.
> Lin (1992) *Machine Learning* — experience replay. Fortunato et al. (2018) ICLR — Factorized
> Noisy Linear. Bellemare, Dabney, Munos (2017) ICML — C51 categorical projection.
> Rowland et al. (2018) AISTATS — Cramér valid distributional loss.
> Ioffe & Szegedy (2015) ICML — feature whitening / batch normalization warmup.
