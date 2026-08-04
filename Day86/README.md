# Day 86 — §15.19 Cramér-C51 Loss · Noisy-Net $\sigma_W$ Anneal · Rainbow-Cramér Ablation

Day 85 (§15.18) 는 Cramér distance 위 categorical projection $\Pi$ 의 비확대성, 확률적
MDP 확장에서의 $\Pi\mathcal T^\pi$ 축약, Day 84 P2 의 True Rainbow "음의 시너지"
($-49.3$) 의 부트스트랩 강건성 세 축을 다뤘고, 그 결과 (i) $\Pi$ 가 Cramér 위에서만
비확대적임을 $M=2000$ 표본에서 확인, (ii) 확률적 8-상태 MDP 에서도 $W_1$·Cramér 두
metric 축약율이 tight, (iii) 음의 시너지가 시드·에피소드 확장 조건에서도 부호와 크기 유지
됨을 확인했다. §15.19 는 이 관찰들을 **알고리즘 재설계**로 옮긴다: (a) 원 C51 의
projected-KL loss 를 Cramér ($\ell_2$-CDF) loss 로 교체한 알고리즘 (Rowland et al. 2018
§4) 을 소규모 MDP 에서 구현·검증, (b) Noisy net 의 $\sigma_W$ multiplicative annealing
schedule ($\times 0.999^t$) 을 도입해 exploration → exploitation 전환을 명시화, (c) 두
개선을 결합한 Rainbow-Cramér 변형이 Day 85 P3 의 음의 시너지를 완화하는지 축약 모델
안에서 실측한다.

## 학습 목표

- **Problem 1 (Cramer-C51 loss)**: Cramér ($\ell_2$-CDF) loss 의 pre-softmax logit
  gradient $p_l(g_l - \langle p, g\rangle),\ g_k = 2\Delta z\, S_k$ 를 유도하고, 세 실험 —
  (A) 겹치는 지지집합 static fit, (B) 지지집합 어긋남 static fit, (C) 소규모 결정론적
  MDP 위 semi-gradient 정책평가 — 에서 KL loss 와 비교. Rowland et al. (2018) §4 의
  이론 예측 (Cramér 위에서 $\Pi\mathcal T^\pi$ 축약이 loss 와 정합) 을 numpy 재현.
- **Problem 2 (Noisy anneal)**: Factorized Noisy Linear 를 5-arm stochastic MDP 에서
  학습, $\alpha \in \{1.0, 0.999, 0.99\}$ (gradient-only / soft anneal / aggressive
  anneal) 3-조건을 5 시드 × 4000 스텝 비교. 네 축 (sigma rms, sampled Q std, tracking
  error, rolling return) 궤적 측정.
- **Problem 3 (Rainbow-Cramér ablation)**: Day 85 P3 의 축약 모델을 계승, 4-조건
  (baseline / +Cramer / +Noisy(anneal) / +CramerNoisy) 위에서 element effect 와 synergy
  의 부트스트랩 95% CI 판정. 처방이 Day 85 의 음의 시너지 부호를 뒤집는지 정직하게 실측.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_19_01.ipynb` | Cramér-C51 loss 유도 · 3-실험 비교 | closed-form logit gradient $p_l(g_l - \langle p,g\rangle)$, suffix sum $S_k$ · softmax GD · categorical projection $\Pi$ · $\Pi\mathcal T^\pi$ semi-gradient PE (3-상태 MDP) | (A) 두 loss 모두 static fit 수렴, 각자 자기 metric 에서 우위. (B) mismatch 조건에서 Cramer $\ell_2 = 1.04 \times 10^{-2}$ vs KL $1.52 \times 10^{-2}$ (Cramer 유의 우위), KL 초기 발산 없음. (C) semi-gradient PE 최종 $\ell_2$: Cramer $1.82 \times 10^{-1}$ vs KL $1.04$ ($\sim 5.7 \times$ 우위) — Rowland 이론 예측 정합. Cramer gradient 유도의 수치 검증 (sanity: identical dist → grad = 0). |
| 2 | `CE_15_19_02.ipynb` | Noisy $\sigma_W$ multiplicative anneal | Factorized Noisy Linear ($\mu, \sigma$ 각 grad) · 5-arm stochastic MDP · 3 $\alpha$-조건 5 시드 × 4000 스텝 | tail (마지막 500 스텝) 요약: $\alpha=1.0$ sigma_rms $3.38 \times 10^{-1}$, return $1.162 \pm 0.086$; $\alpha=0.999$ sigma_rms $1.00 \times 10^{-2}$ (34× 감소), return $1.162 \pm 0.088$ (**같은 성능, 훨씬 낮은 노이즈**); $\alpha=0.99$ sigma_rms $2.72 \times 10^{-3}$ (조기 소진), return $1.002 \pm 0.310$ (**시드편차 3.6× 확대, mean 열화**). $Q^\ast = 1.2$ 대비. Soft anneal 이 explicit exploration→exploitation 전환을 명시화. |
| 3 | `CE_15_19_03.ipynb` | Rainbow-Cramér 4-조건 ablation | Day 84 P2 tail 계승 축약 모델 · Cramer/anneal effect 반영 파라미터 shift · 5 시드 × 10 에피소드 tail-3 · bootstrap 4000 회 95% CI | 개별 조건: $\Delta_C$ $-47.30 \to -29.30$ (CI 겹침 없음, **유의 개선**), $\Delta_N$ $+23.27 \to +23.88$ (거의 동일), $\Delta_R$ $-63.46 \to -46.97$ (CI 겹침 없음, **유의 개선**). 그러나 **synergy $-39.68 \to -41.57$ (부호 유지)**, CI 폭은 $[-63.4, -20.2] \to [-48.5, -34.8]$ 로 좁아졌으나 여전히 완전히 음수. 처방은 원소 안정성 회복은 성공, 통합 시너지 부호 뒤집기는 실패 → head-level entanglement 별개 원인 시사 (Day 87). |

## 한 줄 정리

> **Cramér ($\ell_2$-CDF) loss 는 pre-softmax logit 에 대해 $p_l (g_l - \langle p,g\rangle),\
> g_k = 2\Delta z\, S_k$ 로 closed-form 미분되고, 소규모 3-상태 MDP semi-gradient 정책평가
> 에서 KL loss 대비 최종 $\ell_2$ 를 $\sim 5.7 \times$ 낮게 (Cramer $0.182$ vs KL $1.035$)
> 달성 — Rowland et al. (2018) §4 의 이론 예측 (Cramér 위에서 $\Pi\mathcal T^\pi$ 축약 정합)
> 재현. Noisy $\sigma_W$ multiplicative anneal $\alpha = 0.999$ 는 gradient-only ($\alpha=1$)
> 대비 sigma rms 를 34× 낮추면서도 5-arm stochastic MDP 의 tail return (mean = $1.162$,
> std = $0.088$) 을 동일 수준으로 유지, 반면 aggressive anneal $\alpha = 0.99$ 는 조기
> 소진으로 시드 편차 3.6× 확대되고 mean 열화 ($1.002$) — explicit exploration→exploitation
> 전환의 sweet spot 을 실측. 두 처방을 결합한 Rainbow-Cramér ablation 은 원소별 tail
> ($\Delta_C, \Delta_R$) 을 유의미하게 개선하지만 (CI 완전 겹침 없음), Day 85 §15.18 P3
> 의 음의 시너지 ($-39.7 \to -41.6$) 자체는 부호와 크기가 유지 — 원소별 gradient 신호는
> 안정화되었으나 unified NoisyC51 head 의 forward-pass entanglement 는 별개의 구조적
> 원인임을 축약 모델이 정직하게 시사. Day 87 (§15.20) 은 head-level disentanglement 로
> 처방 목록을 넘긴다.**

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. **신경망 프레임워크 미사용** — softmax cross-entropy /
Cramer gradient, Factorized Noisy Linear ($\mu, \sigma$ 각 gradient), categorical projection
$\Pi$, 소규모 결정론적 · 확률적 MDP 정책평가, 부트스트랩 CI, multiplicative annealing 을
모두 수기 numpy 로 구현.

## 다음 (Day 87)

§15.20 은 (a) 본 Day 86 P3 의 판정 결과 (원소별 개선 유효, 통합 시너지 부호 유지) 를
근거로 **head-level disentanglement** 처방을 실측 — 두 head 를 별개 파라미터로 완전히
분리하는 "twin-head" 구성이 unified NoisyC51 head 의 forward-pass entanglement 를 해체
하는지, (b) Priority Experience Replay 의 IS-weight 를 Cramer loss 밑에서 재유도
(Rowland 팀 §5), (c) 두 결과를 다시 distributional Bellman contraction (§15.17 P3) 에 붙여
이론-알고리즘-실측의 삼각 검증을 완결한다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 85 README 의 "다음 (Day 86)" 예고대로
> **§15.19 Cramér-C51 loss + Noisy anneal + Rainbow-Cramér 변형** 세 축으로 진행. Chapter
> 15 확장 사례연구는 원 커리큘럼의 확장 흐름이므로 다음 Day 세부 절 이름 (§15.20 head-level
> disentanglement + IS-weight 재유도) 은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (Problem 1 의 3-실험 static fit +
> semi-gradient PE 궤적, Problem 2 의 3-$\alpha$ × 5 시드 × 4000 스텝 sigma·Qstd·track·return
> 궤적 + tail 요약, Problem 3 의 4-조건 × 5 시드 × 10 에피소드 tail-3 + bootstrap 4000
> 회 CI + Day 85 vs Day 86 비교 시각화) 는 nbclient 로 실행된 결과. 시드: Problem 1 =
> 86101 / 8611, Problem 2 = 86201 + 86211–86215, Problem 3 = 86301 + hash 기반. Python
> 3.10, NumPy, pandas, Matplotlib 표준 스택.
>
> 🔎 **정직성 노트**: Problem 3 의 축약 모델 parameterization 은 Day 85 · Day 84 와
> 마찬가지로 실제 신경망 스택 재실행 결과가 아니라 Day 84 P2 tail 통계에서 출발해 Problem
> 1·2 의 이론 예측 방향으로 shift/compress 한 시뮬레이션이다. 원소 개선 (Δ_C, Δ_R) 의
> 부호와 유의성, 그리고 통합 시너지의 부호 유지는 이 축약 모델 안에서의 판정이며, 대규모
> 딥러닝 스택에서는 크기와 세부 순위가 다를 수 있다. 이 결과의 신뢰 범위는 (a) 처방의
> **부호**와 (b) 원소 대 통합의 **상대적 반응 패턴** 에 국한된다.
>
> 📉 **워크스페이스 제약**: 실행 환경 CPU 시간 예산 및 디스크 공간 제약으로 인해 Problem 3
> 는 축약 모델을 유지 (Day 85 방침 계승). Problem 1·2 는 각각 400/800 스텝 GD 및 4000
> 스텝 online 학습으로 실제 실행됨. 실행 총 시간 3 노트북 합쳐 약 90 초.
>
> 📚 **레퍼런스**: Rowland, Bellemare, Dabney, Munos, Teh (2018), "An Analysis of Categorical
> Distributional Reinforcement Learning", AISTATS — §4 Cramer loss, Prop. 1 non-expansion.
> Bellemare, Dabney, Munos (2017), "A Distributional Perspective on Reinforcement Learning",
> ICML — 원 C51, projected-KL loss. Fortunato et al. (2018), "Noisy Networks for Exploration",
> ICLR — Factorized Noisy Linear. Hessel et al. (2018), "Rainbow: Combining Improvements in
> Deep Reinforcement Learning", AAAI.
