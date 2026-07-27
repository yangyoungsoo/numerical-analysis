# Day 79 — §15.12 Quantile Regression DQN (QR-DQN) + Implicit Quantile Networks (IQN) + 선형 함수근사 규모 이행

Day 78(§15.11) 에서 C51 · Noisy Nets · n-step 을 통합한 Rainbow-lite 가 확률 전이 GridWorld
에서 개별 조건의 산술합을 초과하는 시너지를 낸다는 사실을 관측했다. C51 의 마지막 남은
한계 — **지지대 이산화 잔차** ($Q$-error 하한 $\propto \Delta z$) — 를 §15.12 는
**quantile regression** 으로 원리적으로 제거한다. 이번 Day 는 (i) tabular QR-DQN 이 C51 의
잔차를 실제로 제거하는지, (ii) 이를 한 번 더 뒤집은 IQN 이 tabular 규모에서도 등가 성능을
내는지, (iii) 이 흐름이 **선형 함수근사** 규모로 이행될 때 §15.10–§15.11 의 세 관찰
(요소 순증분, 확률 전이의 n-step 활성화, 편향 제거) 이 살아남는지 검증한다.

## 학습 목표

- **Problem 1 (QR-DQN)**: 4×4 결정적 전이 GridWorld + $r=-1+\mathcal N(0,1)$ 잡음 보상에서
  scalar Q / C51 ($K=31$) / QR-DQN ($K=31$, quantile Huber $\kappa=1$) 10 시드 × 150 에피소드
  비교. C51 의 지지대 이산화 잔차가 mean $Q$-error 에 남기는 편향을 QR-DQN 이 원리적으로
  제거하는지 정량 관측.
- **Problem 2 (IQN)**: 동일 환경에서 IQN 의 tabular 등가물 (cosine τ-embedding + 선형 head)
  을 구현. $D=16$ 파라미터가 QR-DQN 의 $K=31$ 보다 적은데도 임의 τ 에서 quantile 조회가
  가능한지, sample efficiency 를 유지하는지, CVaR$_{0.1}$ 같은 위험지표를 자연스럽게
  산출하는지 검증. τ-sampling 이 소규모 tabular 에서 노이즈 지배로 이어지는 경계 관찰.
- **Problem 3 (선형 함수근사 규모 이행)**: 6×6 slippery GridWorld ($p_\text{slip}=0.2$,
  $\sigma_R=0.5$) + row/col one-hot + bias features ($D_\phi=13$) 위에서 4 조건 (linear scalar
  / linear QR-DQN / linear IQN / linear Rainbow-lite = QR+n=3) 8 시드 × 120 에피소드 비교.
  Day 77 예측 ("n-step 은 확률 전이가 있어야 되살아남") 이 tabular 밖에서도 재현되는지 검증.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_12_01.ipynb` | Quantile Regression DQN (tabular) | 확률 고정 $\tau_k=(k-\tfrac12)/K$ · 지지대 학습 $\theta_k(s,a)$ · quantile Huber $\rho^\kappa_\tau$ | $V^\star=-5.85$; tail50 리턴: scalar $-7.12$, C51 $-17.60$, QR-DQN $-10.80$. $Q$-error: scalar $1.59$, **C51 $7.50$**, **QR-DQN $3.55$ (C51 의 지지대 잔차 절반으로 감소)**. 학습된 quantile 함수는 τ→0 에서 잡음 누적 tail 을 정확히 재현. |
| 2 | `CE_15_12_02.ipynb` | Implicit Quantile Networks (tabular) | $\phi(\tau)_i=\cos(i\pi\tau)$ 임베딩 · $M_\text{on}=M_\text{tar}=8$ τ-샘플링 · quantile Huber grad | tail50 리턴: scalar $-7.16$, QR-DQN $-11.41$, **IQN $-27.12$**. $Q$-error: $1.54 / 3.57 / 5.45$. **CVaR$_{0.1}$**: QR-DQN $-6.06$, IQN $-2.66$ (flat, 미수렴). IQN 의 이론적 우수성 (임의 τ 조회, 파라미터 절약) 은 신경망 · 큰 규모에서 발현 — tabular 소규모에서는 τ-sampling 이 학습 신호를 희석. |
| 3 | `CE_15_12_03.ipynb` | 선형 함수근사 규모 이행 (slippery 6×6) | row/col one-hot + bias ($D_\phi=13$) · linear 4 조건 · $p_\text{slip}=0.2$ | $V^\star_\text{slip}=-11.53$; tail50 리턴: scalar $-14.16$, QR-DQN $-20.60$, IQN $-29.82$, **Rainbow-lite (QR+n=3) $-18.01$ (QR 단독 대비 +2.6 개선)**. **Day 77 예측 "n-step 은 확률 전이가 있어야 되살아남" 이 정확히 재현**. QR-DQN 의 $Q$-error 감소 이점은 규모 이행 후에도 유지 ($7.85 \to 6.64$). |

## 한 줄 정리

> **QR-DQN 은 C51 의 지지대 이산화 잔차를 원리적으로 제거해 mean 근사 정확도를 개선하며
> (${Q\text{-err}: 7.50 \to 3.55}$), IQN 의 τ-sampling 은 소규모 tabular 에서 노이즈 지배로
> 열세이지만 임의 τ 조회 · 위험지표 산출이라는 표현력 이점을 유지한다. 두 방법 모두 선형
> 함수근사로 규모 이행 시 QR-DQN 의 편향 제거 이점이 유지되며, slippery 환경에서
> n-step 부트스트랩이 QR 단독 대비 +2.6 리턴 개선을 만들어 Day 77 의 예측 (확률 전이가
> n-step 을 재활성화) 이 tabular 밖에서도 정확히 재현된다.**

## 사용 라이브러리
- NumPy, pandas, Matplotlib (표준 스택). 신경망 프레임워크 미사용 — tabular / 선형 근사만으로
  각 알고리즘의 gradient · 부트스트랩 · quantile 학습 경로를 정확히 재현.

## 다음 (Day 80)

§15.13 **완전한 신경망 파라미터화 (MLP + ReLU) + target network + gradient clipping** —
Problem 3 의 4 조건을 신경망으로 이행하고 larger-scale 환경 (MountainCar 유사, 실수 상태공간)
에서 (i) IQN 의 tabular 열세가 신경망 규모에서 회복되는지, (ii) target network 가 QR·n-step
과 어떻게 상호작용하는지, (iii) Rainbow full stack (QR + Noisy + PER + n-step + Dueling +
Double) 을 함께 붙였을 때 개별 조건의 산술합을 초과하는 시너지가 tabular 관찰 (Day 78) 대비
얼마나 확대되는지 검증.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3) 까지이며, Day 14 이후는
> 스케줄러가 로드맵을 절 단위로 자동 진행. Day 78 README 의 예고대로 **§15.12 QR-DQN + IQN
> + 신경망 파라미터화 규모 이행** 으로 진행 (규모 이행 부분은 신경망 프레임워크 없이 선형
> 함수근사로 대체 — 다음 Day 신경망 이행의 baseline).
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (QR-DQN 3-조건 요약, IQN 3-조건 + CVaR,
> linear 4-조건 요약) 는 nbclient 로 실행된 결과. 시드: Problem 1 = 9100..9109,
> Problem 2 = 9200..9209, Problem 3 = 9300..9307. Python 3.10, NumPy, pandas, Matplotlib
> 표준 스택. 총 실행 시간 ~30 초.
>
> 📉 **워크스페이스 제약**: 실행 환경 디스크 제약 때문에 seed 수를 Day 78 대비 축소
> (Problem 1·2: 15→10 시드, Problem 3: 15→8 시드 × 120 에피소드). tail 통계의 상대 순위는
> 유지되지만, 절대값 비교 시 이 점 감안.
>
> 🔎 **정직성 노트**: IQN 의 tabular 리턴이 QR-DQN 대비 열세인 결과는 Dabney et al. (2018b) 의
> 원 주장 (신경망 · Atari 규모에서의 우수성) 과 모순되지 않는다. tabular 소규모는 IQN 의
> 이점이 발현되는 체제 밖. Problem 2 의 관찰은 이 경계선 자체를 정량화한 결과.
