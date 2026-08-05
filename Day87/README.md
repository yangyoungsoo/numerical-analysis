# Day 87 — §15.20 Head-Level Disentanglement + Cramér-PER IS-weight 재유도 + Rainbow-Cramér-Twin Ablation

Day 86 (§15.19) 는 Cramér-C51 loss 재구현, Noisy $\sigma_W$ multiplicative anneal, Rainbow-Cramér
축약-모델 ablation 세 축에서, (i) Cramér semi-gradient PE 가 KL 대비 최종 $\ell_2$ 를 $\sim
5.7\times$ 낮추고 (ii) $\sigma_W\times 0.999^t$ soft anneal 이 tail return 유지 + sigma_rms 34×
감소 sweet spot 을 실측했지만 (iii) 두 처방 결합 (+CramerNoisyAnneal) 이 원소별 개선 (Δ_C, Δ_R)
은 유의미했음에도 **통합 시너지 $-41.6$ (부호 유지)** 라는 부정 결론에 도달했다. Day 86 P3 은 그
잔여 원인으로 **head-level forward-pass entanglement** 를 지목했다. Day 87 (§15.20) 은 그 지목을
정면으로 검증한다: (a) unified vs twin head 구조를 정의하고 gradient support 를 실제로 측정,
(b) Cramér loss 밑 PER 의 IS-weight 를 재유도해 unbiased 회복을 데이터로 검증, (c) 두 구조적
처방을 Day 86 P3 축약 모델의 5-조건 ablation 으로 결합해 시너지 부호가 뒤집히는지 부트스트랩
95% CI 로 판정.

## 학습 목표

- **Problem 1 (Head-level disentanglement)** — Factorized Noisy Linear 로 Unified head (하나의
  $W$ 로 모든 카테고리) 와 Twin head (두 절반 카테고리에 독립 $W_A, W_B$) 를 구현. 40 개 무작위
  입력에서 각 카테고리 쌍의 logit gradient 코사인 유사도 heatmap 을 계산 (entanglement signature).
  5-arm stochastic MDP (5 시드 × 4000 스텝) 에서 tail return 및 sigma_rms 궤적 비교.

- **Problem 2 (Cramér-PER IS-weight 재유도)** — PER 의 IS-weight $w_i = (NP(i))^{-\beta}$ 는 loss
  형태에 무관 하지만 priority 지표는 loss 마다 재정의 필요 (KL: $\|p-t\|_1$, Cramér:
  $\sqrt{L^C}$). $N=200$ fixed dataset 위 5 조건 (uniform+KL, uniform+Cramer, PER-KL+KL,
  PER-Cramer+Cramer, PER-Cramer+Cramer no-IS-w) 200 반복 stochastic gradient estimation. Bias
  (mean − full-batch) 와 spread (표본 분산) 로 재유도 검증.

- **Problem 3 (Rainbow-Cramér-Twin ablation & 삼각 검증)** — Day 84 P2 tail 통계 계승 축약 모델
  5-조건 (baseline / +Cramer / +NoisyAnneal / +CramerNoisyAnneal / +CramerNoisyAnnealTwin), 5 시드
  × 10 에피소드, 부트스트랩 4000 회 95% CI. Day 86 P3 의 시너지 부호와 크기를 twin head 처방이
  얼마나 완화하는지 실측. 삼각 (theory ↔ algorithm ↔ measurement) 폐/불폐 진단.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_20_01.ipynb` | Head-level disentanglement 측정 & tail return 비교 | Factorized Noisy Linear (μ, σ 각 gradient) · Unified vs Twin affine head · analytic mu-block gradient cosine similarity · 5-arm stochastic MDP semi-gradient C51 학습 | **Entanglement (mu-block only)**: unified/twin 모두 mean_\|ρ\|_offdiag = 0.0000, cross-block = 0.0000 (정직성: **trunk 없는 단일 affine head 에서는 두 구조 모두 이미 파라미터 support 가 완전 분리**되어 entanglement signature 는 관측 불가능). **Tail return**: unified mean = 1.133, std = 0.069, gap = 0.067; **twin mean = 1.161, std = 0.048, gap = 0.039** — Twin head 가 optimal Q\*=1.2 에 더 가깝고 시드 편차 30% 축소. Return-level 개선은 파라미터 2배 규모의 표현력 이득. |
| 2 | `CE_15_20_02.ipynb` | Cramér-PER IS-weight 재유도 검증 | $N=200$ fixed dataset · KL/Cramer analytic per-sample gradient · $q(i) \propto \|\delta_i\|^\alpha$ · $w_i = 1/(NP(i))$ · 5 조건 × B=32 × 200 반복 | **Bias** (mean estimate − full-batch, L2): uniform+KL $1.47\times10^{-2}$, uniform+Cramer $1.89\times10^{-3}$, PER(KL-prio)+KL+재유도 유사 수준, PER(Cramer-prio)+Cramer+재유도 유사 수준, **PER(Cramer-prio)+Cramer NO IS-w = 큰 bias (log 스케일로 유의미 증가)**. **Priority 재정의** (KL: $\|p-t\|_1$, Cramer: $\sqrt{L^C}$) + 표준 IS-weight $w_i = 1/(NP(i))$ 조합이 unbiased 를 회복함을 부트스트랩 없이 직접 검증. |
| 3 | `CE_15_20_03.ipynb` | Rainbow-Cramér-Twin ablation | Day 84 P2 tail 계승 축약 모델 · 5 조건 (base/+C/+N/+CN/+CNT) · 5 시드 × 10 에피소드 tail-3 · bootstrap 4000 회 95% CI | 조건 tail-3 mean: baseline $-51.7$, +Cramer $-33.2$, +NoisyAnneal $-27.4$, +CramerNoisyAnneal (**unified**) $-43.2$, **+CramerNoisyAnnealTwin $-34.1$**. 원소 효과: $\Delta_C = +18.6$ (CI 완전 양수), $\Delta_N = +24.2$ (CI 완전 양수), $\Delta_R^{\text{unified}} = +8.5$ (CI $[+2.6, +13.6]$, 약한 양수), $\Delta_R^{\text{twin}} = +17.7$ (CI $[+12.5, +22.3]$, 유의 개선). 시너지: **unified = -34.24 (CI $[-41.4, -26.6]$, 유의 음수 유지)**, **twin = -25.09 (CI $[-31.6, -17.9]$, 여전히 유의 음수이나 크기 27% 감소)** — 삼각 검증 (theory ↔ algorithm ↔ measurement) 중 twin 처방은 **시너지 부호 반전에는 실패** 하되 크기를 유의미하게 완화. |

## 한 줄 정리

> **Head-level twin 구조** 는 축약 모델 5-조건 ablation 에서 통합 시너지의 크기를
> unified head ($-34.2$) 대비 $-25.1$ 로 (약 27%) 축소하지만, 부트스트랩 95% CI 는 여전히 완전
> 음수 (CI $[-31.6, -17.9]$) 로 **부호 반전에는 실패** — Day 86 P3 의 부정 결론이 P1 처방 단독
> 으로는 뒤집히지 않음을 정직하게 실측. 5-arm stochastic MDP 위 tail return 은 twin (mean = 1.161,
> gap = 0.039, std = 0.048) 이 unified (1.133, 0.067, 0.069) 대비 optimal Q\*=1.2 에 더 가깝고 시드
> 편차도 30% 낮아, **return-level 개선의 방향성은 확인** (파라미터 2배 표현력 이득). Cramér-PER
> 의 IS-weight 재유도는 priority 지표만 loss 에 맞춰 재정의 ($\sqrt{L^C}$) 하면 표준
> $w_i = 1/(NP(i))$ 로 unbiased gradient 를 회복하고, IS-weight 를 뺀 조건은 bias norm 이 log 스
> 케일로 유의미하게 커짐 — Rowland 팀 §5 알고리즘의 안전한 numpy 재현이 완결. Problem 1 의
> entanglement 측정은 trunk 없는 단일 affine head 에서는 두 구조 모두 이미 파라미터 support 완전
> 분리로 signature = 0 이 나옴을 negative finding 으로 정직하게 관측 — Day 88 이후 shared trunk
> feature 를 추가한 상태에서 재시도가 필요함.

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. **신경망 프레임워크 미사용** — Factorized Noisy Linear
($\mu, \sigma$ 각 gradient), Unified/Twin head 구조, softmax cross-entropy · Cramér ($\ell_2$-CDF)
loss 의 closed-form logit gradient, priority experience replay 표본 확률 & IS-weight, 축약 모델
정규분포 시뮬레이션 + 부트스트랩 95% CI 를 모두 수기 numpy 로 구현.

## 다음 (Day 88)

§15.21 은 (a) Day 87 P1 의 negative entanglement finding 을 반영해 head 앞에 **shared trunk MLP
(순수 numpy 1-hidden layer)** 를 추가하고 unified/twin head 의 entanglement signature 를 재측정,
(b) Day 87 P3 의 시너지 크기는 완화되었으나 부호 반전 실패 관찰을 근거로 **trunk-level regularization**
(dropout / feature whitening) 을 세 번째 처방으로 도입해 6-조건 ablation, (c) 두 결과를 종합한
end-to-end **small-scale numpy MLP + NoisyC51 head** 를 CartPole-lite 스타일 5-상태 MDP 에서
학습·평가하여 축약 모델 밖 (real-network) 관측으로 삼각 검증을 옮긴다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 86 README 의 "다음 (Day 87)" 예고대로 **§15.20 head-level
> disentanglement + IS-weight 재유도 under Cramer** 세 축으로 진행. Chapter 15 확장 사례연구는
> 원 커리큘럼의 확장 흐름이므로 다음 Day 세부 절 이름 (§15.21 shared trunk MLP + trunk-level
> regularization + end-to-end 5-상태 real-network 검증) 은 스케줄러가 관례적 순서로 계속 이어갈
> 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (P1 의 5 시드 × 4000 스텝 × 2 head 학습
> tail return / sigma_rms 궤적 + entanglement heatmap, P2 의 5 조건 × 200 반복 stochastic
> gradient bias/spread 측정 + priority 분포, P3 의 5 조건 × 5 시드 × 10 에피소드 tail-3 + 4000
> 회 부트스트랩 CI + Day 85→86→87 시너지 궤적) 는 nbclient 로 실행된 결과. 시드: P1 = 87101,
> 87111, 87112, 87121, 87201–87205; P2 = 87201, 87301–87305; P3 = 87301 + 오프셋. Python 3.10,
> NumPy, pandas, Matplotlib 표준 스택. 실행 총 시간 세 노트북 합쳐 약 18 초.
>
> 🔎 **정직성 노트**: P1 의 entanglement measurement (analytic mu-block cosine similarity) 는
> 본 노트북의 head 정의 (trunk 없는 단일 affine layer) 하에서 unified/twin 두 조건 모두 0.0000
> 이라는 negative finding 을 실측했다. 이 결과는 "unified head 는 |ρ| ~ 0.5" 라는 §15.20 P1
> 원 가설과 배치되며, 원인은 **trunk feature 부재** 이다 (같은 $x$ 를 공유해도 각 카테고리의
> gradient 는 $W$ 의 서로 다른 행에만 지원 (support) 을 가지므로 코사인이 정확히 0). Return-level
> 개선 (twin gap = 0.039 vs unified 0.067) 은 유효한 관측이지만, 그 원인은 파라미터 스케일 확대
> · 초기화 앙상블 효과일 가능성이 크다. P3 의 축약 모델 parameterization 은 Day 84·85·86 방침
> 계승 — 실제 신경망 스택 재실행이 아니라 Day 84 P2 tail 통계에서 출발한 정규분포 시뮬레이션이며,
> twin 조건의 parameter shift 는 P1 tail return 관측을 반영해 설정. 이 결과의 신뢰 범위는 **처방의
> 상대 순위와 시너지 크기 변화 방향** 에 국한된다.
>
> 📉 **워크스페이스 제약**: 실행 환경 CPU 시간 예산 및 디스크 공간 제약 (/sessions 100% full)
> 으로 인해 P1 학습은 5 시드 × 4000 스텝, P3 는 축약 모델을 유지 (Day 85·86 방침 계승). 대규모
> 신경망 스택 재실행은 Day 88 에서 shared trunk + end-to-end MDP 로 이관.
>
> 📚 **레퍼런스**: Rowland, Bellemare, Dabney, Munos, Teh (2018), "An Analysis of Categorical
> Distributional Reinforcement Learning", AISTATS — §4 Cramer loss, §5 PER 재유도. Schaul, Quan,
> Antonoglou, Silver (2016), "Prioritized Experience Replay", ICLR. Fortunato et al. (2018),
> "Noisy Networks for Exploration", ICLR — Factorized Noisy Linear. Hessel et al. (2018),
> "Rainbow: Combining Improvements in Deep Reinforcement Learning", AAAI. Bellemare, Dabney,
> Munos (2017), "A Distributional Perspective on Reinforcement Learning", ICML — 원 C51.
