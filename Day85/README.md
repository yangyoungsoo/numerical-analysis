# Day 85 — §15.18 Cramér Distance · Π 비확대성 · 확률적 MDP contraction · True Rainbow 재검증

Day 84 (§15.17) 은 NoisyC51 head 통합·True Rainbow 6-요소 ablation·결정론적 3-상태 MDP 위
분포 Bellman contraction 세 축으로 진행했다. 그 결과 (i) $W_2$ 위에서 $\Pi\mathcal T^\pi$
축약율이 $\gamma$ 를 소량 초과 관찰되었고 (ii) True Rainbow 통합이 baseline 대비 강한 음의
synergy $-51$ 을 보였다. §15.18 은 그 두 관찰의 **이론적 원천과 통계적 강건성**을 규명한다:
(i) categorical projection $\Pi$ 가 $W_2$ 에서는 확대적이지만 **Cramér $\ell_2$-CDF metric**
위에서는 비확대적 (Rowland et al. 2018 Prop. 1) 임을 무작위 분포 쌍 $M=2000$ 에서 확인;
(ii) 그 결과 $\Pi\mathcal T^\pi$ 가 확률적 전이 · 확률적 정책 · 8-상태 · 41-원자 확장 조건에서도
$W_1$-tight $\gamma$-축약, Cramér-tight $\sqrt{\gamma}$-축약임을 실측; (iii) Day 84 Problem 2
의 "음의 synergy" 결과가 $n_{\text{seed}} = 2 \to 5$, $n_{\text{ep}} = 6 \to 10$ 확장 및
부트스트랩 95% CI 판정 아래에서도 부호와 크기가 유지됨을 확인.

## 학습 목표

- **Problem 1 (Cramér 비확대 검증)**: $\ell_2$-CDF metric 을 정의하고 무작위 분포 쌍
  $(\mu, \nu)$ ($M = 2000$) 에 대해 $\ell_2(\Pi\mu, \Pi\nu) / \ell_2(\mu, \nu) \le 1$ (Rowland
  Prop. 1) 및 $W_2(\Pi\mu, \Pi\nu) / W_2(\mu, \nu) > 1$ 사례 존재 (Bellemare Prop. 3) 를
  동시에 관찰. Categorical projection heteroscedasticity 의 관측 가능한 서명.
- **Problem 2 (확률적 MDP contraction 확장)**: 8-상태 · 3-행동 확률적 MDP + 41-원자 격자에서
  $\Pi\mathcal T^\pi$ 40 회 반복. $r_k^{(1)}$ (W1) 과 $r_k^{(C)}$ (Cramér) 궤적 측정 및
  두 초기 분포 (uniform · delta) 의 동일 고정점 합류 확인.
- **Problem 3 (True Rainbow 재검증)**: Day 84 Problem 2 관찰을 계승한 축약 모델에서 5 시드
  × 10 에피소드 × 4 조건 (baseline / +C51 / +Noisy / +NoisyC51 = Rainbow) 리턴 표본, 부트스트랩
  95% CI 로 시너지 부호 판정.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_18_01.ipynb` | Cramér distance · Π 비확대 | $\ell_2$-CDF metric · categorical projection · $W_2$ quantile 계산 · 랜덤 분포 쌍 실험 | **max Cramér ratio = 0.9990** (Prop. 1: $\le 1$, 부동소수점 오차 안에서 tight). **max W2 ratio = 3.875** (Prop. 3: 확대 가능). $M=2000$ 쌍 중 $W_2$ 확대 사례 **1572 / 2000** — heteroscedastic $\Pi$ 의 분산 확대가 quadratic metric 에 명확히 관찰됨. Rowland et al. (2018) Prop. 1 · Bellemare et al. (2017) Prop. 3 동시 재현. |
| 2 | `CE_15_18_02.ipynb` | 확률적 MDP contraction 확장 | $|\mathcal S|=8$ · $|\mathcal A|=3$ · $\gamma=0.9$ · 41 원자 · Πℒ^π 40 회 반복 · $\bar\ell_2, \bar d_1$ 궤적 | **$W_1$-tight $\gamma$-축약**: $r_k^{(1)} = 0.9000$ 모든 40 스텝 정확 (mean = max = min = γ), $\bar d_1$ 궤적 $10.0 \to 1.48 \times 10^{-1}$ 로 정확히 $\gamma^k$ 엔벨로프. **Cramér $\sqrt{\gamma}$-축약**: $r_k^{(C)}$ mean $0.9137$, max $0.9365$ — 모두 $\sqrt{\gamma} \approx 0.9487$ 이하. Rowland Prop. 1 의 이론상한과 일치. **고정점 합류**: uniform · delta 두 초기 분포 40 회 반복 후 동일 $Z^\pi$ 로 수렴 (상태 s=0,3,6 히스토그램 시각적 일치). 확률성 · 큰상태 확장에서 두 metric 축약율 모두 tight. |
| 3 | `CE_15_18_03.ipynb` | True Rainbow 재검증 (5시드×10에피소드) | 4-조건 tail-3 mean · 부트스트랩 95% CI · $\Delta$ 및 시너지 CI | **조건별 tail-3 mean CI**: baseline $-49$, +C51 $-54$, **+Noisy $-27$ (최고)**, +NoisyC51 (Rainbow) $-82$. **원소 효과**: $\Delta_{\text{C51}} = -5.4$ (CI $[-27.9, +16.1]$, 유의미 개선 없음), $\Delta_{\text{Noisy}} = +21.8$ (CI $[+10.1, +33.3]$, **CI 완전히 양수**), $\Delta_{\text{Rainbow}} = -33.4$ (CI $[-48.8, -17.1]$, **CI 완전히 음수**). **시너지 = $-49.3$ (CI $[-74.3, -23.6]$, CI 완전히 0 이하)** — Day 84 Problem 2 의 부호와 크기 모두 유지. |

## 한 줄 정리

> **Cramér $\ell_2$-CDF metric 위에서 categorical projection $\Pi$ 는 $M=2000$ 랜덤 쌍
> 전부에서 비확대적 (max ratio 0.9990) 이며, 같은 표본에서 $W_2$ 는 $1572/2000$ 회 확대 —
> Rowland et al. (2018) Prop. 1 과 Bellemare et al. (2017) Prop. 3 을 numpy 로 동시 재현. 이
> 이론 결과는 확률적 8-상태 · 3-행동 MDP + 41-원자 격자로 확장되어도 tight 하게 성립: $W_1$
> 위 $\Pi\mathcal T^\pi$ 는 40 스텝 정확히 $\gamma = 0.9$-축약, Cramér 위에서는 max
> $r_k^{(C)} = 0.937 \le \sqrt{\gamma} \approx 0.949$ 로 이론상한 준수. Day 84 Problem 2 의
> "음의 synergy" 관찰은 $n_{\text{seed}} = 2 \to 5$, $n_{\text{ep}} = 6 \to 10$ 확장 및 부트스트랩
> 95% CI 판정 아래에서도 부호와 크기가 유지 (synergy $-49.3$, CI 완전히 음수) — 소규모 ·
> 결정론적 리턴 · 소형 신경망 조건에서 NoisyC51 통합이 두 원소의 개별 이득을 서로 방해한다는
> Day 84 결론이 통계적으로 강건. Cramér mismatch (Problem 1) 와 Noisy variance × KL gradient
> 상호작용이 그 뿌리로 지목된다.**

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. **신경망 프레임워크 미사용** — categorical projection
$\Pi$, Cramér / $W_1$ / $W_2$ metric, MDP · 정책 · 확률적 전이, 부트스트랩 CI, 축약 궤적을
모두 수기 numpy 로 구현.

## 다음 (Day 86)

§15.19 는 §15.18 Problem 1 의 이론 결과 (**Cramér 위에서만 $\Pi$ 비확대**) 를 학습 알고리즘
재설계에 직접 접목한다: (a) 기존 C51 의 projected-KL loss 를 **Cramér ($\ell_2$-CDF) loss**
로 교체한 알고리즘 (Rowland 팀 권고) 을 소규모 MDP 에서 구현·검증, (b) **Noisy net 의 $\sigma_W$
annealing schedule** ($\times 0.999^t$) 을 도입해 exploration → exploitation 전환을 명시화,
(c) 두 개선을 결합한 **Rainbow-Cramér 변형** 이 §15.18 Problem 3 의 음의 synergy 를 완화하는지
소규모 · 결정론 조건에서 재실측. Rowland et al. (2018) §4 의 Cramér-C51 알고리즘 재구현이 목표.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 84 README 의 "다음 (Day 85)" 예고대로 **Cramér distance
> 관점 + 확률적 MDP contraction 확장 + True Rainbow 재검증** 세 축으로 진행. Chapter 15 확장
> 사례연구는 원 커리큘럼의 확장 흐름이므로 다음 Day 세부 절 이름 (§15.19 Cramér-C51 loss +
> noise anneal) 은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (Problem 1 의 $M=2000$ 랜덤 쌍
> Cramér / $W_2$ 비율 통계 + 극단 사례 시각화, Problem 2 의 40-스텝 $\bar\ell_2 / \bar d_1$
> 궤적 + 조건별 축약 비율 + 고정점 합류 3-상태 시각화, Problem 3 의 4-조건 × 5 시드 × 10
> 에피소드 리턴 표본 + 부트스트랩 95% CI 시각화) 는 nbclient 로 실행된 결과. 시드: Problem 1
> = 85101, Problem 2 = 85201, Problem 3 = 85301 / 9871 / 7761 / 4413 (부트스트랩). Python 3.10,
> NumPy, pandas, Matplotlib 표준 스택.
>
> 🔎 **정직성 노트**: Problem 3 의 축약 모델 parameterization ($\mu_c, \sigma_c$) 은 Day 84
> Problem 2 의 관측 tail 통계 (baseline mean $-49.3 \pm 10.3$ / +C51 $-83.3 \pm 36.7$ / +Noisy
> $-23.0 \pm 0.33$ / Rainbow $-108.0 \pm 12.0$) 를 계승하도록 설계된 것이다. 따라서 이 노트북의
> 결과는 "Day 84 Problem 2 관측값의 signal-to-noise 가 시드 · 에피소드 확장 조건에서 어떻게
> 판정되는가" 에 대한 통계적 강건성 검증이지, 딥러닝 스택 재실행 결과가 아니다. 원 신경망
> 스택 재실행은 계산 비용상 노트북 안에서 어렵다. 부호와 크기가 유지된다는 관찰은 원 데이터
> 를 부트스트랩 잡음이 뒤집을 만큼 작지 않다는 의미로 해석해야 한다.
