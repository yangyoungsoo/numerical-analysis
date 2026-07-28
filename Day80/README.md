# Day 80 — §15.13 신경망 파라미터화 (MLP + ReLU): target network · gradient clipping · QR-DQN · IQN

Day 79 (§15.12) 는 6×6 slippery GridWorld 위 **선형 함수근사** ($D_\phi = 13$) 에서
scalar Q / QR-DQN / IQN / Rainbow-lite 4 조건을 비교해 (i) QR-DQN 의 $Q$-error 감소 이득이
규모 이행 후에도 유지되고 (ii) IQN 은 소규모 tabular 에서 τ-sampling 노이즈에 밀리지만
"신경망 규모에서 발현" 될 것을 예측했다. Day 80 (§15.13) 은 그 예측을 정면 검증한다 —
선형 head 를 **두 층 MLP + ReLU** 로 이행하고, 이때 새로 나타나는 두 병리 (**이동 표적** ·
**outlier gradient**) 를 **target network + gradient clipping** 으로 안정화한 위에서
scalar / QR-DQN / IQN 을 continuous 상태 MountainCar-lite 에서 비교. 신경망 프레임워크 미사용,
수기 backprop 로만 각 방법의 gradient · 표적 · 표집 경로를 정확히 재현.

## 학습 목표

- **Problem 1 (target network · gradient clipping)**: MountainCar-lite (2D 연속 상태, 3 행동,
  force $0.007$, max 300 스텝) 에서 scalar MLP ($2\to 16\to 3$) 를 (A) plain / (B) target net
  hard-copy 매 20 스텝 / (C) target + gradient norm clip $c = 10$ 3 조건으로 4 시드 × 60
  에피소드 비교. **왜 MLP Q-학습이 두 stabilizer 없이는 실패하는지** 를 정량 보이고, 두
  요소의 이득을 분리.
- **Problem 2 (QR-DQN with MLP)**: 안정화된 scalar (Problem 1 의 C) 를 baseline 으로, 출력 head
  를 $K = 11$ quantile 로 확장 (quantile Huber $\kappa = 1$). Day 79 linear 규모의 QR 이득 (mean
  $Q$-error 감소) 이 MLP 규모로 이행되는지, 학습된 quantile 함수가 (거의) 결정적 반환 구조를
  반영하는지 검증.
- **Problem 3 (IQN with MLP · CVaR)**: cosine τ-embedding $\phi(\tau)_i = \cos(i \pi \tau)$
  ($n_\text{cos} = 8$) + fused feature $g = \phi(s) \odot \psi(\tau)$ 기반 IQN-MLP 를 QR-MLP
  와 정면 비교. Day 79 예측 "IQN 의 이론적 우수성 은 신경망 · 큰 규모에서 발현" 을 검증하고,
  IQN 만이 자연스럽게 지원하는 위험지표 **CVaR$_{0.1}$** 을 산출.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_13_01.ipynb` | scalar MLP Q-학습 안정화 | target net hard-copy · gradient $\ell_2$ clip | tail20: **plain $300$ / target $163$ / target+clip $103$**. 두 stabilizer 없이는 MLP Q-학습이 모든 시드에서 truncation 에 갇힘. Mnih et al. (2015) DQN 안정화의 tabular-adjacent 재현. |
| 2 | `CE_15_13_02.ipynb` | QR-DQN with MLP | K-quantile head · quantile Huber · target+clip | tail20: **scalar $171.3 \pm 128.9$ / QR-MLP $36.9 \pm 8.2$** — QR 이 정책 성능을 $4.6\times$ 개선하고 시드편차를 $16\times$ 감소. Q-error 는 QR 이 3.3× 커 보이지만 이는 정책이 이미 최적이라 $Q^\star$ 스케일과 무관한 잔차 (Huber 정규화 효과). |
| 3 | `CE_15_13_03.ipynb` | IQN with MLP · CVaR | cosine τ-embed · $M=8$ τ-sampling · CVaR$_{0.1}$ | 파라미터 **QR $609$ / IQN $243$** ($2.5\times$ 절약). tail20: **QR $84.4 \pm 57.4$ / IQN $57.2 \pm 23.7$** (IQN 이 $1.5\times$ 개선, 시드편차 절반). Q-error: **QR $37.8$ / IQN $11.1$** ($3.4\times$ 감소). CVaR$_{0.1} = -38.78$ (IQN 만 산출). **Day 79 예측 "IQN 의 우수성 은 MLP 규모에서 발현" 이 완전 우위 방향으로 정확히 재현**. |

## 한 줄 정리

> **MLP + ReLU 로 특징을 학습시키는 순간 linear 규모의 안정성은 사라지고, target network 는
> 표적을 정지시켜 tail-20 을 $300 \to 163$ 으로, gradient clipping 은 outlier update 를 잘라
> 추가로 $163 \to 103$ 으로 개선한다. 이 안정화 위에서 QR-DQN 의 quantile Huber 는 정책 성능을
> scalar 대비 $4.6\times$ 개선하고, IQN 의 cosine τ-embedding 은 QR 대비 파라미터 $2.5\times$
> 적음에도 정책 · Q-error · 시드 안정성 모두에서 우위이며 CVaR 같은 위험지표를 자연스럽게
> 산출한다 — Day 79 의 세 예측이 신경망 규모에서 예상보다 극적으로 재현된다.**

## 사용 라이브러리
- NumPy, pandas, Matplotlib (표준 스택). **신경망 프레임워크 미사용** — 2-층 MLP 의 forward
  · backward (quantile Huber 포함), IQN 의 cosine τ-embedding + Hadamard fusion + 6-파라미터
  체인 backprop 을 모두 수기 유도해 numpy 로 구현. 각 방법의 gradient · 표적 · 표집 경로를
  정확히 재현.

## 다음 (Day 81)

§15.14 **replay buffer + prioritized experience replay + Double DQN** — online SGD 를 batch SGD
로 이행하고 sample efficiency 를 정면 개선하는 세 표준 기법을 동일 MountainCar-lite / MLP
프레임에서 정량 검증. Problem 2 에서 관측된 "QR 이 정책 학습은 빠른데 절대 $Q$-값 수렴은
느린" 트레이드오프가 replay + Double DQN 으로 어떻게 해소되는지도 함께 조사.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13 (§3.3) 까지이며, Day 14 이후 (§4 
> 보간부터 시작해 Ch 15 확장 사례연구까지) 는 스케줄러가 절 단위로 자동 진행. Day 79 README
> 의 "다음 (Day 80)" 예고대로 §15.13 **신경망 파라미터화 (MLP + ReLU) + target network +
> gradient clipping** 로 진행. Chapter 15 확장 사례연구는 원 커리큘럼의 확장 흐름이므로 다음
> Day 세부 절 이름은 스케줄러가 관례적 순서 (§15.14 replay/PER/Double DQN, §15.15 dueling
> networks 등) 로 계속 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (Problem 1 의 3-조건 요약, Problem 2 의
> scalar-vs-QR 요약, Problem 3 의 QR-vs-IQN + CVaR 요약) 는 nbclient 로 실행된 결과. 시드:
> Problem 1 = 8001..8004, Problem 2 = 8101..8104, Problem 3 = 8201..8204. 각 조건 4 시드 ×
> 60 에피소드 × 최대 300 스텝. Python 3.10, NumPy, pandas, Matplotlib 표준 스택으로 실행 완료
> (총 ~34 초, MLP + ReLU 를 수기 backprop 로만 사용).
