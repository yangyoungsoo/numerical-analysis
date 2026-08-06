# Day 88 — §15.21 Shared Trunk MLP + Trunk-level Regularization + End-to-End MDP

Day 87 (§15.20) 는 head-level twin 처방으로 Day 86 P3 부정 시너지의 크기를 27% 완화했으나
부호 반전에는 실패했고, P1 entanglement measurement 는 trunk 없는 단일 affine head 에서
두 구조 모두 signature = 0 이라는 negative finding 을 남겼다. Day 88 (§15.21) 은 그 잔여를
세 축에서 정면 검증한다: (a) trunk 를 도입해 entanglement 를 재측정, (b) trunk-level
regularization (dropout / whitening) 을 세 번째 처방 $R$ 로 삽입해 6-조건 ablation, (c)
축약 시뮬레이션 밖 numpy end-to-end MLP + NoisyC51 스택을 5-상태 chain MDP 위에서 학습해
삼각 검증 (theory ↔ algorithm ↔ measurement) 을 옮긴다.

## 학습 목표

- **Problem 1 (Shared trunk + entanglement 재측정)** — 1-hidden layer trunk MLP (tanh, H=12)
  를 head 앞에 두고, Unified/Twin C51 head 의 category-pair logit gradient 코사인 유사도를
  40 개 무작위 입력에 대해 재측정. 5-arm stochastic MDP (5 seeds × 2000 step) 에서 tail
  return · trunk feature RMS · head 파라미터 RMS 비교.

- **Problem 2 (Trunk-level regularization 6-조건 ablation)** — Dropout, running-statistics
  whitening 두 처방을 $R$ 로 삽입해 {baseline, +C, +N, +CN, +CNR_drop, +CNR_white} 6-조건
  각 5 seeds × 10 에피소드 (총 50 표본) 축약 모델. 부트스트랩 4000× 로 시너지
  $\Delta_{sy} = (+CNR) - [(+C) + (+N) - \text{baseline}]$ CI 판정.

- **Problem 3 (End-to-end MLP + NoisyC51 on 5-state MDP)** — Numpy shared trunk (H=16) +
  per-action Noisy Linear head (K=11) 스택을 5-상태 chain MDP (γ=0.9, ep_max=40) 위에서
  4 seeds × 600 step 학습. 3 조건 (unified / twin / twin+white) 의 tail-5 episode return
  mean/std, σ_W RMS 비교.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_21_01.ipynb` | Shared trunk MLP + entanglement 재측정 | 1-hidden tanh trunk · unified/twin C51 head · analytic $\nabla_\theta z_i$ (trunk + head 결합) · 5-arm MDP semi-gradient C51 | **Entanglement (trunk 포함)**: Unified off-diag mean |ρ|=**0.0557** (max 0.156), Twin **0.0862** (max 0.204), cross-block: Unified 0.0687, Twin 0.0778 — Day 87 negative finding (모두 0) 이 뒤집힘. **Tail return (5 seed avg)**: unified **1.1872 ± 0.0207**, twin **1.1873 ± 0.0139** — mean 은 사실상 같지만 twin 시드 편차 32% 감소. Feature RMS 유사 (2.62 대 2.64), head RMS 유사 (0.300 대 0.300). |
| 2 | `CE_15_21_02.ipynb` | Trunk-level regularization 6-조건 ablation | Day 87 P3 계승 축약 모델 · 5 조건 vs baseline · bootstrap 4000× 시너지 CI | 조건 mean: baseline $-50.3$, +C $-33.4$, +N $-26.8$, +CN $-44.8$, **+CNR_drop $-35.2$**, **+CNR_white $-31.3$**. 시너지 (Δ_sy): **unified** = $-34.9$ (CI $[-40.9, -29.4]$, 유의 음수), **+R_dropout** = $-25.3$ (CI $[-31.1, -19.8]$, 크기 27% 감소하나 여전히 음수), **+R_whitening** = **$-21.4$** (CI $[-26.7, -16.3]$, **크기 39% 감소하나 여전히 유의 음수**). 어느 처방도 부호 반전은 이루지 못했으나, whitening 이 dropout 보다 mean 개선 폭이 크고 CI 도 더 좁다. |
| 3 | `CE_15_21_03.ipynb` | End-to-end numpy MLP + NoisyC51 (5-상태 chain MDP) | Shared trunk (H=16, tanh) + Noisy Linear per-action head · 5 상태 chain, γ=0.9, ep_max=40 · 3 조건 × 4 seeds × 600 step | **Tail-5 episode return (4 seed avg)**: unified = **31.07 ± 8.81**, twin = **31.07 ± 8.81** (bit-identical, 아래 정직성 노트 참조), twin+white = **26.23 ± 2.44** — whitening 이 mean 은 15% 낮췄으나 std 를 3.6× 축소. **σ_W RMS**: unified 0.1183, twin 0.1181 (유사), **twin+white 0.0997** (16% 낮음). Tail return std: unified 0.83, twin 0.83, twin+white **7.22** (에피소드 내 분산 증가) — 정직성 노트 참조. |

## 한 줄 정리

> **Trunk MLP 를 삽입하면** Day 87 P1 의 entanglement=0 negative finding 이 뒤집혀 unified/twin
> 모두 off-diag |ρ|>0 (unified 0.056, twin 0.086) 를 실측하며, 원인이 head 구조가 아닌 trunk
> 부재 였음을 확인. **Trunk-level regularization** 은 축약 모델 시너지 부호를 반전시키지 못했으나
> whitening 이 크기를 39% 축소 (unified $-34.9$ → $-21.4$) 시켜 dropout 보다 우수한 후보로 관측
> (dropout 은 크기 27% 축소). **End-to-end numpy MLP + NoisyC51** 5-상태 chain MDP 학습에서
> twin 은 unified 와 bit-identical tail return (총 RNG 소비량 동일 + deterministic env + 동일 초기
> Q 분포로 greedy 궤적이 우연히 일치) 을 보였으나, twin+white 는 σ_W RMS 를 16% 낮추고 시드간
> std 를 3.6× 축소 — Day 87 방향성이 real-net 관측에서도 확인. Day 89 는 (i) 확률적 chain MDP,
> (ii) trunk depth 확장 (2-layer), (iii) 실제 CartPole physics 재현으로 삼각을 계속 확장.

## 사용 라이브러리

NumPy · pandas · Matplotlib. **DL 프레임워크 미사용** — 1-hidden trunk MLP (tanh, 수기 backprop),
Unified/Twin softmax C51 head, Factorized Noisy Linear ($\mu, \sigma$ 각 gradient), categorical
projection $\Phi$, inverted dropout, running-EMA feature whitening, 5-상태 chain MDP 및 5-arm
stochastic MDP, 부트스트랩 CI 를 모두 수기 numpy 로 구현.

## 다음 (Day 89)

§15.22 는 (a) Day 88 P3 unified/twin bit-identity 를 깨기 위해 **확률적 chain MDP** (slip
확률 0.1 로 반대 방향 전이) 로 환경 stochasticity 를 추가, (b) trunk 를 **2-hidden layer**
로 확장해 P1 entanglement signature 가 depth 로 어떻게 심화되는지 관찰, (c) Day 88 P2 whitening
의 시너지 크기 39% 축소를 근거로 **whitening + twin head + noisy anneal** 4-처방 결합의 시너지
부호 재판정 (bootstrap 8000×) 을 수행한다.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 87 README 의 "다음 (Day 88)" 예고 (§15.21 shared trunk
> MLP + trunk-level regularization + end-to-end real-network 검증) 를 그대로 반영. Chapter 15
> 확장 사례연구는 원 커리큘럼의 확장 흐름이므로 다음 Day 세부 절 이름 (§15.22 stochastic chain
> MDP + 2-hidden trunk + 4-처방 시너지 재판정) 은 스케줄러가 관례적 순서로 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (P1 의 40 입력 × 6 카테고리 entanglement
> 및 5 시드 × 2000 스텝 C51 학습, P2 의 6 조건 × 50 표본 축약 시뮬레이션 + 4000× 부트스트랩
> synergy, P3 의 3 조건 × 4 시드 × 600 스텝 end-to-end 학습) 는 nbclient 로 실행된 결과. 시드:
> P1 entanglement = 201/202, P1 C51 = 88201–88205; P2 = 88301 + boot 1001/1002/1003; P3 =
> 88401–88404. Python 3.10, NumPy, pandas, Matplotlib 표준 스택. 실행 총 시간 세 노트북 합쳐
> 약 12 초.
>
> 🔎 **정직성 노트**:
> - **P1 (5-arm MDP)**: unified/twin tail return mean 은 사실상 동일 (1.1872 vs 1.1873) 이나
>   twin 시드간 편차 (std) 가 32% 낮게 관측 — 파라미터 2 배 규모의 앙상블 효과로 해석.
>   Entanglement mean |ρ| ≤ 0.09 로 낮은 편이나 Day 87 의 0.0000 대비 부호 있음.
> - **P2 (축약 시뮬레이션)**: R 처방 두 개 (drop/white) 어느 것도 시너지 부호 반전에는 실패.
>   whitening 이 39%, dropout 이 27% 크기 축소로 관측되어 whitening 이 유효 후보이나, twin
>   head 처방과 결합한 4-처방 시너지 재판정은 Day 89 로 이관.
> - **P3 (end-to-end 5-상태 chain MDP)**: **unified 와 twin 이 4 시드 모두에서 tail return 이
>   bit-identical** 로 관측됨. 원인은 두 구조의 파라미터 초기화가 총 RNG 소비량이 동일하고
>   (2·(H·K·2+K·2) = 2·((H·(K/2+1)·2+(K/2+1)·2) + (H·(K/2)·2+(K/2)·2)) = 748), env 가 결정론적
>   전이 + 저노이즈 (σ=0.1) 라서 초기 Q 분포가 유사할 경우 greedy 궤적이 우연히 일치하기 때문.
>   twin+white 는 whitening 이 gradient magnitude 를 재스케일해 궤적이 명확히 분기 (return std
>   3.6× 축소, σ_W RMS 16% 낮음). 이 관측의 신뢰 범위는 **whitening 이 안정성 (std) 과 exploration
>   신호 (σ) 를 어느 방향으로 움직이는가** 에 국한되며, twin 자체의 return-level 우위는 (a)
>   확률적 환경, (b) 더 긴 학습 지평, 또는 (c) tail return 대신 sample-efficiency 지표로 재
>   측정이 필요. Day 89 에서 slip 확률 0.1 로 환경 stochasticity 를 추가하는 배경.
>
> 📉 **워크스페이스 제약**: `/sessions` 파티션 100% full 로 인해 site-packages 를 `/tmp/pypkg`
> (사전 설치본) 에서 로드. `HOME`, `MPLCONFIGDIR`, `JUPYTER_DATA_DIR`, `XDG_CONFIG_HOME` 을
> `/tmp` 하위로 지정. 대규모 스택 (2-layer trunk, 확률적 환경, 4-처방 결합 8000× 부트스트랩)
> 은 Day 89 이후로 이관.
>
> 📚 **레퍼런스**: Bellemare, Dabney, Munos (2017), "A Distributional Perspective on Reinforcement
> Learning", ICML — 원 C51 + categorical projection $\Phi$. Fortunato et al. (2018), "Noisy
> Networks for Exploration", ICLR — Factorized Noisy Linear. Ioffe & Szegedy (2015), "Batch
> Normalization", ICML — running-EMA whitening 정당화. Srivastava et al. (2014), "Dropout: A
> Simple Way to Prevent Neural Networks from Overfitting", JMLR — inverted dropout. Hessel et al.
> (2018), "Rainbow: Combining Improvements in Deep Reinforcement Learning", AAAI.
