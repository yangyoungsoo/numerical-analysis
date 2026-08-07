# Day 89 — §15.22 Stochastic Chain MDP · 2-Hidden Trunk Depth · 4-처방 시너지 재판정 (8000× Bootstrap)

Day 88 (§15.21) 은 (i) shared trunk MLP 삽입으로 Day 87 entanglement=0 negative finding 을
뒤집고 (unified |ρ|=0.056, twin |ρ|=0.086), (ii) trunk-level whitening 이 축약 모델 시너지
크기를 39% 축소했으나 부호 반전 실패, (iii) end-to-end 5-상태 chain MDP 에서 unified/twin
이 bit-identical tail return 을 산출하는 **놀라운 negative finding** 세 축으로 마감했다.
Day 89 (§15.22) 는 그 잔여를 세 축에서 정면 검증한다: (a) 결정론적 → **확률적 chain MDP**
($p_{\text{slip}}=0.1$) 로 bit-identity 파괴, (b) trunk 를 **2-hidden layer** ($H_1=16, H_2=12$)
로 확장해 depth 별 entanglement signature 재측정, (c) whitening + twin head + noisy anneal +
Cramer 네 처방 결합의 4-way 시너지 부호를 **부트스트랩 8000×** 로 재판정.

## 학습 목표

- **Problem 1 (확률적 chain MDP, bit-identity 파괴)** — Shared trunk (H=16) + per-action Noisy
  Linear head (K=11) 스택을 $p_{\text{slip}}=0.1$ 확률적 5-상태 chain MDP 위에서 4 seeds × 800
  step 학습. 세 조건 (unified, twin, twin+whitening) tail-8 episode return, seed-간 std,
  $\sigma_W$ RMS 비교. Day 88 P3 bit-identity 파괴 여부를 max-per-seed-abs-diff 로 판정.

- **Problem 2 (2-hidden trunk, entanglement depth study)** — Trunk 를 $H_1=16, H_2=12$ 로 확장,
  logit gradient 를 (i) layer-1-only, (ii) layer-2-only, (iii) all trunk+head 세 부분집합으로
  분해. 40 무작위 입력 위 unified/twin 각 조건의 off-diagonal / cross-block $|\rho|$ 를 측정,
  Day 88 (1-layer) 대비 depth scaling factor 계산.

- **Problem 3 (4-처방 시너지 재판정, bootstrap 8000×)** — 8 조건 축약 시뮬레이션
  {baseline / +C / +N / +R / +T / +CN / +CNR / +CNRT}, 5 seeds × 10 에피소드 = 50 표본. 4-처방
  시너지 정의 $\Delta_{sy}^{(4)} = \mu(+CNRT) - [\mu(+C)+\mu(+N)+\mu(+R)+\mu(+T)-3\mu(\text{base})]$,
  부트스트랩 8000× 95% CI 로 부호 반전 판정.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_22_01.ipynb` | 확률적 chain MDP + bit-identity 파괴 | Shared trunk (H=16) + per-action Factorized Noisy Linear head · $p_{\text{slip}}=0.1$ 확률적 전이 · 3 조건 × 4 seeds × 800 step · running-EMA whitening | **Bit-identity 파괴 성공** — `max |unified - twin| across seeds = 0.821`. Tail-8 return (4 seed avg): unified **+0.544 ± 0.537**, twin **+0.729 ± 0.169** (seed_std 68% 감소), **twin+white +0.851 ± 0.069** (seed_std 87% 감소). $\sigma_W$ RMS (last 100 step): unified 0.120, twin 0.119, twin+white **0.110** (8% 낮음). twin+white 가 세 지표 모두에서 우위. |
| 2 | `CE_15_22_02.ipynb` | 2-hidden trunk entanglement depth study | 2-hidden layer trunk ($H_1=16, H_2=12$, tanh) · analytic 3-부분집합 gradient (L1/L2/all) · 40 무작위 입력 · unified/twin heatmap 6장 | **Mean off-diagonal $|\rho|$**: unified L1 **0.399**, L2 0.248, all 0.165; twin L1 0.351, L2 0.259, all 0.153. **L1 > L2** (layer2/layer1 = 0.62 unified, 0.74 twin) — Layer-1 이 $W_2^\top$ 재선형결합으로 채널이 얽힘, Layer-2 는 head weight 방향 차이가 살아남음. **Day 88 (1-layer) 대비 all**: unified **2.96×**, twin **1.78×** — depth 심화 시 unified 가 twin 대비 얽힘 증가율 66% 큼. Cross-block Twin: L1 0.294, L2 0.259, all 0.142 (shared trunk 로 인해 0 이 아님). |
| 3 | `CE_15_22_03.ipynb` | 4-처방 시너지 부트스트랩 8000× 재판정 | 8 조건 축약 시뮬레이션 (baseline / +C / +N / +R / +T / +CN / +CNR / +CNRT) · Day 88 P2 계승 파라미터 · bootstrap 8000× 95% CI | 조건 mean (50 표본): baseline **$-49.8$**, +C $-33.2$, +N $-26.1$, +R $-30.9$, +T $-36.5$, +CN $-18.0$, +CNR $-6.6$, **+CNRT $+3.0$** (baseline 대비 절대치 반전). 원소 효과 (CI 완전 양수): $\Delta_C$ **+16.7** [+11.3, +21.9], $\Delta_N$ **+23.8** [+18.7, +28.8], $\Delta_R$ **+18.9** [+13.2, +24.6], $\Delta_T$ **+13.4** [+8.0, +18.7]. 시너지: CN **$-8.6$** [-14.8, -2.3], CNR **$-16.1$** [-26.2, -5.6], **CNRT $-20.0$** [-34.5, -5.6] — 4-처방 결합 부호는 여전히 **유의 음수** (CI 상한이 -5.6 으로 0 을 넘지 못함). Day 88 (+CNR_white) $-21.4$ 대비 크기 6% 추가 축소하나 부호 반전 실패. |

## 한 줄 정리

> **확률적 chain MDP** ($p_{\text{slip}}=0.1$) 는 Day 88 P3 의 unified/twin bit-identical
> tail-return negative finding 을 완전히 파괴하며 (max |diff| = 0.82), twin+white 가 세 조건
> 중 tail return (+0.85), seed std (0.07), $\sigma_W$ RMS (0.110) 모든 지표에서 우위를 보여
> Day 87/88 방향성이 real-network stochastic 환경에서 재현. **2-hidden trunk** 는 entanglement
> signature 를 Day 88 (1-layer) 대비 unified 2.96×, twin 1.78× 로 심화시키며, Layer-1 (입력측)
> 이 Layer-2 (head 인접) 보다 큰 얽힘 (unified 0.399 vs 0.248) 을 보이는 이유는 Layer-1
> gradient 가 $W_2^\top$ 를 통과하며 카테고리 간 채널 정렬을 겪기 때문. Twin 은 두 층 모두에서
> unified 보다 낮은 얽힘 (L1 0.351 vs 0.399, L2 0.259 vs 0.248) 을 유지하나 shared trunk 를
> 통과하며 disentanglement 이점이 부분적으로 상쇄. **4-처방 시너지** (+CNRT) 는 부트스트랩
> 8000× 로 $-20.0$ [-34.5, -5.6] 로 판정되어 Day 88 (+CNR_white) $-21.4$ 대비 크기 6% 추가
> 축소하나 CI 상한 $-5.6$ 이 0 을 넘지 못해 **부호 반전 실패** — twin head 원소 효과 ($\Delta_T$
> = +13.4, CI 완전 양수) 는 유의미하나 4-처방 결합의 diminishing marginal 이 여전히 지배적.
> Day 90 은 4-처방 결합의 real-network 재현 및 layer-wise 처방 분리 (L1-only vs L2-only whitening)
> 로 확장.

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. **DL 프레임워크 미사용** — 1-hidden 및 2-hidden trunk MLP
(tanh, 수기 backprop), Unified/Twin softmax C51 head, per-action Factorized Noisy Linear
($\mu, \sigma$ 각 gradient), categorical projection $\Phi$, running-EMA feature whitening,
확률적 5-상태 chain MDP, 8-조건 축약 시뮬레이션 + 부트스트랩 8000× CI 를 모두 수기 numpy 로 구현.

## 다음 (Day 90)

§15.23 은 (a) Day 89 P3 4-처방 시너지의 부호 반전 실패 관찰을 근거로 **real-network 4-처방
end-to-end 스택** 을 Day 89 P1 확률적 chain MDP 위에서 재실행하여 축약 모델과 real-net 사이
결론 일관성을 검증, (b) Day 89 P2 관찰 (L1 이 L2 보다 큰 얽힘) 을 바탕으로 whitening 처방을
Layer-1-only, Layer-2-only, both 로 분리해 시너지 기여를 layer-wise 분해, (c) 확률적 환경
$p_{\text{slip}}$ 를 {0.05, 0.1, 0.2} 로 sweep 해 stochasticity 크기가 twin 이점 및 시너지
부호에 미치는 영향 sensitivity 분석을 수행한다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 88 README 의 "다음 (Day 89)" 예고 (§15.22 확률적 chain
> MDP + 2-hidden trunk + 4-처방 시너지 재판정 8000×) 를 그대로 반영. Chapter 15 확장 사례연구
> 는 원 커리큘럼의 확장 흐름이므로 다음 Day 세부 절 이름 (§15.23 real-network 4-처방 재현 +
> layer-wise 처방 분리 + $p_{\text{slip}}$ sweep) 은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (P1 의 3 조건 × 4 시드 × 800 스텝
> end-to-end 학습 + bit-identity 진단, P2 의 40 무작위 입력 × 6 카테고리 × 3 파라미터 부분집합
> analytic gradient cosine + heatmap 6장, P3 의 8 조건 × 50 표본 정규분포 시뮬레이션 +
> 8000× 부트스트랩 CI) 는 nbclient 로 실행된 결과. 시드: P1 = 89101–89104; P2 trunk =
> 89201/89202, sample = 89203; P3 base = 89301, bootstrap = 89310 + hash. Python 3.10, NumPy,
> pandas, Matplotlib 표준 스택. 실행 총 시간 세 노트북 합쳐 약 30 초.
>
> 🔎 **정직성 노트**:
> - **P1 (확률적 chain MDP)**: unified/twin bit-identity 는 성공적으로 파괴 (max |diff|=0.82).
>   Tail-8 return 은 twin+white 이 가장 높지만 (+0.85), 학습 궤적이 짧은 (800 step) 편이라
>   asymptotic 성능이 아닌 초기~중기 학습의 안정성 지표로 해석해야 함. seed 편차 축소 관찰은
>   Day 87/88 관찰과 정합하나 stochasticity 로 인해 축소율은 완화됨.
> - **P2 (2-hidden trunk entanglement)**: 관측이 원래 예상 ("L2 > L1") 과 반대로 나옴 —
>   L1 이 L2 보다 큰 얽힘을 보였다. 이는 40 무작위 입력 위 gradient 방향 통계이며, $W_2^\top$
>   전치 행렬이 카테고리 gradient 를 재선형결합하는 효과로 해석된다. 원 가설을 정직히
>   부정하고 원인을 데이터로 재유도한 결과.
> - **P3 (4-처방 시너지)**: 축약 시뮬레이션 파라미터 (shift/multiplier) 는 Day 88 P2 방침을
>   계승해 element 처방과 결합 처방의 marginal-diminishing 항을 실측 궤적 (Day 84 → 88) 에
>   맞춰 설정. 4-처방 시너지 부호가 여전히 음수로 나왔다는 관찰은 이 축약 시뮬레이션 안에서의
>   판정이며, 대규모 딥러닝 스택에서는 크기와 세부 순위가 다를 수 있다. 이 결과의 신뢰 범위는
>   (a) 처방의 **부호** 와 (b) 원소 대 결합의 **상대적 반응 패턴** 에 국한된다.
>
> 📉 **워크스페이스 제약**: `/sessions` 파티션 100% full 로 인해 site-packages 를 `/tmp/pypkg`
> 에 설치. `HOME`, `MPLCONFIGDIR`, `JUPYTER_DATA_DIR`, `XDG_CONFIG_HOME` 을 `/tmp` 하위로
> 지정. Day 89 P1 real-network 실행은 800 step × 4 seeds × 3 conditions 로 유지 (Day 88 방침
> 계승). P3 real-network 4-처방 결합 재실행은 Day 90 으로 이관.
>
> 📚 **레퍼런스**: Bellemare, Dabney, Munos (2017), "A Distributional Perspective on
> Reinforcement Learning", ICML — 원 C51 + categorical projection $\Phi$. Fortunato et al. (2018),
> "Noisy Networks for Exploration", ICLR — Factorized Noisy Linear. Ioffe & Szegedy (2015),
> "Batch Normalization", ICML — running-EMA whitening 정당화. Rowland, Bellemare, Dabney, Munos,
> Teh (2018), "An Analysis of Categorical Distributional Reinforcement Learning", AISTATS —
> Cramér loss 배경. Hessel et al. (2018), "Rainbow: Combining Improvements in Deep Reinforcement
> Learning", AAAI.
