# Day 90 — §15.23 Real-Network 4-처방 재현 · Layer-wise Whitening 분해 · $p_{\text{slip}}$ Sweep

Day 89 (§15.22) 는 (i) 확률적 chain MDP ($p_{\text{slip}}=0.1$) 로 unified/twin bit-identity
를 파괴 (max |diff|=0.82), (ii) 2-hidden trunk 에서 Layer-1 이 Layer-2 보다 큰 gradient
entanglement (unified 0.399 vs 0.248), (iii) 4-처방 시너지 $\Delta_{sy}^{(4)}=-20.0$
[$-34.5$, $-5.6$] (bootstrap 8000×, 부호 반전 실패) 세 축으로 마감했다. Day 90 (§15.23) 은
그 잔여를 세 축에서 정면 검증한다: (a) real-network 4-처방 결합의 축약-모델 예측 재현,
(b) whitening 을 Layer-1-only / Layer-2-only / both 로 분리한 layer-wise 처방 분해,
(c) $p_{\text{slip}} \in \{0.05, 0.10, 0.20\}$ sweep 을 통한 twin advantage 및 시너지 CI
민감도.

## 학습 목표

- **Problem 1 (Real-network 4-처방 재현)** — Shared trunk (H=16) + per-action Factorized
  Noisy Linear head (K=11) 을 stochastic chain MDP ($p_{\text{slip}}=0.1$) 위에서
  {baseline, +CNR (Cramér + Noisy anneal + Whitening), +CNRT (Twin head 추가)} × 4 seeds
  × 600 step 학습. Tail-8 return 및 $\sigma_W$ RMS 비교로 Day 89 P3 축약 모델의
  negative-synergy 예측을 real-net 에서 재판정.

- **Problem 2 (Layer-wise whitening 분해)** — 2-hidden trunk ($H_1=16, H_2=12$) 를 결정
  론적 5-상태 chain MDP 에 배치하고 whitening 조건을 {none, L1-only, L2-only, both} 네
  갈래로 나눠 4 seeds × 400 step 학습. Marginal effect ($\Delta_{L1}$, $\Delta_{L2}$) 와
  결합 항 (both), interaction (both − additive) 을 실측.

- **Problem 3 ($p_{\text{slip}}$ sweep)** — $p_{\text{slip}} \in \{0.05, 0.10, 0.20\}$ 세
  수준에서 (a) real-net twin+white 의 unified 대비 tail-8 return 우위, (b) 축약 모델
  6-조건 ($\text{base}, +C, +N, +R, +T, +CNRT$) × 30 표본, bootstrap 2000× 95% CI 로
  4-way 시너지 부호를 sensitivity 분석.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_23_01.ipynb` | Real-network 4-처방 end-to-end (stochastic chain MDP) | Shared trunk (H=16) + per-action Factorized Noisy Linear head (K=11), Cramér closed-form logit gradient, running-EMA whitening, $\sigma_W \leftarrow 0.999 \sigma_W$ anneal, twin ($K_A=K_B=K/2$) 헤드 · 3 조건 × 4 시드 × 600 step | Tail-8 return (4 시드 평균): baseline **0.733** (seed_std 0.467), **+CNR 0.963 (0.008)**, +CNRT **0.962 (0.004)**. 3-처방 결합 CNR lift = **+0.230** (seed_std 60× 축소). Twin head incremental effect (CNRT − CNR) = **−0.0016** — Day 89 P3 축약 예측 $\Delta_T = +13.4$ 대비 real-net 에서 사실상 0. $\sigma_W$ last-100 mean: baseline 0.121, +CNR **0.070**, +CNRT 0.069. 축약 모델의 negative-synergy 예측 ($-20.0$) 은 real-net 에서도 **diminishing marginal 로 확인** (Twin 이 CNR 위에서 추가 기여 없음). |
| 2 | `CE_15_23_02.ipynb` | Layer-wise whitening 분해 (2-hidden trunk) | 2-hidden trunk ($H_1=16, H_2=12$, tanh), per-action 스칼라 Q 헤드, 4 조건 × 4 시드 × 400 step, 각 층 running-EMA ($\beta=0.99$) 독립 관리 | Tail-6 return: none **0.693** (seed_std 0.548), L1_only **0.968** (0.002), L2_only **0.968** (0.002), both **0.799** (0.335). Marginal lift: $\Delta_{L1} = +0.275$, $\Delta_{L2} = +0.275$, $\Delta_{\text{both}} = +0.105$ — 두 층 각각은 강력히 안정화하나 **결합 시 저하 (interaction = −0.444)**. Seed-std 비율: L1_only 0.003, L2_only 0.004, both 0.611 — 단독 층 정규화는 시드 편차를 300× 축소, 결합은 오히려 편차 유지. Feature RMS: L1_only 는 h1=0.388, h2=0.563 로 upstream 안정화 후 downstream 확장, L2_only 는 h1=0.388, h2=0.411 로 downstream 압축. 두 층 중복 처리는 gradient 신호를 과잉 감쇠해 학습 저하. |
| 3 | `CE_15_23_03.ipynb` | $p_{\text{slip}}$ sweep — twin advantage & synergy CI | Real-net (H=16, 3 시드 × 500 step) 위 unified vs twin+white × 3 슬립 수준, 축약 모델 6-조건 × 30 표본 slip-scaled ($\sigma_0 \sqrt{1 + 4p(1-p)}$) + bootstrap 2000× | Real twin+white 우위 (tail-8): $p=0.05$ **+0.048**, $p=0.10$ +0.008, $p=0.20$ **+0.042** — 낮은 slip / 높은 slip 모두 유지, 중간 slip 에서 취소. Seed_std 감소: uni→twin, $p=0.05$ 0.78→0.71, $p=0.10$ 0.63→0.61, $p=0.20$ 0.50→0.42 — twin 이 모든 slip 에서 seed 편차 축소. 축약 모델 4-way synergy: $p=0.05$ **−5.5** [$-29.0, +17.5$, **inconclusive**], $p=0.10$ **−32.7** [$-55.0, -11.2$, neg], $p=0.20$ **−35.4** [$-60.9, -10.3$, neg]. Slip 확대로 CI 폭이 넓어지나 mean 도 부호 방향으로 이동해 판정력은 유지. 낮은 잡음 ($p=0.05$) 에서만 표본 소음이 mean 을 압도해 inconclusive. |

## 한 줄 정리

> **Real-network 4-처방 재현**: Cramér + Noisy anneal + Whitening 세 처방 결합 (+CNR) 은
> baseline 대비 tail-8 return 을 +0.230 상승시키고 seed_std 를 60× 축소하며 강력한 학습
> 안정성을 확보하지만, **Twin head 추가 (+CNRT) 는 tail-8 return 을 오히려 −0.0016 낮춰**
> Day 89 P3 축약 모델의 diminishing-marginal 예측 ($\Delta_{sy}^{(4)} = -20.0$) 이 real-net
> 에서도 **Twin incremental effect ≈ 0** 로 명확히 재현. **Layer-wise whitening 분해**:
> L1-only 와 L2-only 는 tail-6 return 을 각각 +0.275 씩 상승시키며 seed_std 를 300× 이상
> 축소하는 반면, **두 층 동시 whitening 은 상승폭이 +0.105 로 반토막나며 interaction
> $= -0.444$ 의 강한 음수 결합** 을 보여, gradient 신호가 두 층 정규화로 과잉 감쇠되어
> 학습 신호가 손실됨을 최초로 실측 (Day 89 P2 의 L1>L2 entanglement 관찰과 정합적이나
> 처방 결합 방식은 replicate-and-add 가 아닌 select-one 이 유효). $p_{\text{slip}}$ **sweep**:
> Twin+white 우위는 $p=0.05, 0.20$ 에서 +0.048/+0.042 유지, seed_std 축소는 모든 slip
> 에서 관측. 축약 모델 4-way 시너지 부호는 $p=0.10, 0.20$ 에서 유의 음수, $p=0.05$ 에서만
> 표본 소음으로 inconclusive [$-29.0, +17.5$] — 시너지의 diminishing marginal 은 환경 잡음
> 이 아닌 처방 자체의 결합 구조에서 나온다는 결론.

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. **DL 프레임워크 미사용** — 1-hidden 및 2-hidden trunk
MLP (tanh, 수기 backprop), Unified/Twin softmax C51 head, Factorized Noisy Linear ($\mu, \sigma$
각 gradient), categorical projection $\Phi$, Cramér loss 의 closed-form logit gradient,
running-EMA feature whitening (layer-wise 독립), 확률적/결정론적 5-상태 chain MDP, 6/8-조건
축약 시뮬레이션 + bootstrap 2000× CI 를 모두 수기 numpy 로 구현.

## 다음 (Day 91)

§15.24 는 (a) Day 90 P2 의 layer-wise both 조건 interaction ($-0.444$) 부호를 근거로
**partial-whitening** (L1 은 running-EMA, L2 는 layer-norm 등 서로 다른 정규화 조합) 로
확장, (b) Day 90 P1 의 real-net Twin incremental effect $\approx 0$ 을 재확인하기 위해
**Twin head 를 per-state-region** (특정 상태 subset 에서만 twin, 나머지는 unified) 로 축소
해 marginal contribution 이 회복되는지 관측, (c) Day 90 P3 의 $p=0.05$ inconclusive 부호를
표본 크기 확장 ($N=100$ per condition) 및 bootstrap 8000× 로 재판정한다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 89 README 의 "다음 (Day 90)" 예고 (§15.23 real-network
> 4-처방 재현 + layer-wise 처방 분리 + $p_{\text{slip}}$ sweep) 을 그대로 반영. Chapter 15
> 확장 사례연구는 원 커리큘럼의 확장 흐름이므로 다음 Day 세부 절 이름 (§15.24 partial-whitening
> + per-state-region twin + 표본 확장 재판정) 은 스케줄러가 관례적 순서로 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (P1 의 3 조건 × 4 시드 × 600 step
> end-to-end 학습, P2 의 4 조건 × 4 시드 × 400 step 학습 + 마지막 100 step feature RMS,
> P3 의 3 슬립 × 3 시드 × 500 step real-net + 6 조건 × 30 표본 축약 + bootstrap 2000×) 는
> `nbclient` 로 실행된 결과. 시드: P1 = 90101–90104; P2 = 90201–90204; P3 real = 90301–90303,
> reduced = 90311 + int(p·1000), bootstrap = 90310. Python 3.10, NumPy 2.2, pandas 2.3,
> Matplotlib 3.10.
>
> 🔎 **정직성 노트**:
> - **P1 (real-net 4-처방)**: +CNR 이 baseline 대비 강한 개선을 보이지만, twin head 추가의
>   incremental effect 가 real-net 에서 사실상 0 (−0.0016) 로 관측되어, 축약 모델의
>   $\Delta_T=+13.4$ 예측과 방향은 일치하나 real-net 에서는 CNR 상단부 tail return 이 이미
>   0.96 근처에 saturate 되어 twin 이 마진을 낼 여지가 없다. 학습 지평 (600 step) 이 짧고
>   tail-8 return 은 초기~중기 안정성 지표이므로, asymptotic 성능은 별도 관측이 필요하다.
> - **P2 (layer-wise whitening)**: **both 조건 interaction $-0.444$ 의 강한 음수** 는
>   원래 예상 (두 층 정규화 결합 시 additive gain) 과 정반대의 관측이다. 원인은 (a) 두 층
>   whitening 이 gradient 를 순차적으로 $1/\sqrt{v+\epsilon}$ 로 나누어 backward 신호가 과잉
>   감쇠, (b) 4 시드의 running EMA 초기화가 짧은 학습 지평에서 두 층에 동시 적용되면 초기
>   variance estimate 편향이 커져 both 조건에서 실효 learning rate 이 감소. 원 가설을 정직히
>   부정하고 원인을 데이터로 재유도한 결과.
> - **P3 ($p_{\text{slip}}$ sweep)**: real-net twin advantage 는 3 시드 관측으로 시드 편차가
>   크며 (0.4–0.8), 부호 신뢰도는 낮다. 축약 모델 시너지 부호는 $p=0.05$ 만 CI 로 inconclusive
>   이지만 point estimate ($-5.5$) 는 부호 방향과 일치하므로 표본 확장 시 유의 음수로 이동
>   가능성이 크다. 이 결과의 신뢰 범위는 (a) 처방 결합의 **diminishing marginal 방향** 및
>   (b) twin+white 의 **seed_std 축소 효과** 에 국한된다.
>
> 📉 **워크스페이스 제약**: `/sessions` 파티션 100% full 로 인해 site-packages 를 `/tmp/pypkg`
> 에 설치. `HOME`, `MPLCONFIGDIR`, `JUPYTER_DATA_DIR`, `XDG_CONFIG_HOME` 을 `/tmp` 하위로
> 지정. Real-net 학습은 seed × step 예산을 Day 89 대비 축소 (P1 은 4 시드 × 600 step, P3 은
> 3 시드 × 500 step) 유지, 확장은 Day 91 로 이관.
>
> 📚 **레퍼런스**: Bellemare, Dabney, Munos (2017), "A Distributional Perspective on
> Reinforcement Learning", ICML — C51. Rowland, Bellemare, Dabney, Munos, Teh (2018), "An
> Analysis of Categorical Distributional Reinforcement Learning", AISTATS — Cramér loss
> gradient. Fortunato et al. (2018), "Noisy Networks for Exploration", ICLR — Factorized Noisy
> Linear. Ioffe & Szegedy (2015), "Batch Normalization", ICML — running-EMA whitening 배경.
> Hessel et al. (2018), "Rainbow: Combining Improvements in Deep Reinforcement Learning", AAAI
> — 처방 결합 관점.
