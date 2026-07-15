# Day 71 — §15.4 Markov Decision Processes & TD Learning (상태를 얹으면 밴딧이 강화학습이 된다)

Day 68(§15.1)~Day 70(§15.3) 은 **상태 없는** 순차결정 — 밴딧 — 을 다뤘다. §15.4 은 **상태** 를
도입해 순차결정을 일반화한다: 유한 MDP $\langle\mathcal S,\mathcal A,P,R,\gamma\rangle$ 위에서
(i) 모델을 알 때의 **정확 계산** (Bellman value iteration), (ii) 모델을 모른 채 표본으로 **가치를
학습** (TD(0) vs MC), (iii) 정책 자체를 **표본으로 학습** (SARSA vs Q-learning).

$$
\text{Bandit } (\text{no state}) \;\xrightarrow{+\text{state, transitions}}\;
\text{MDP} \;\xrightarrow{\text{known model}}\; \text{Value Iteration} \;\xrightarrow{\text{unknown model}}\;
\text{TD prediction / control}.
$$

> **공통 축**: 세 문제 모두 같은 근원 — Bellman 방정식 — 을 다르게 푼다. Value iteration 은
> 결정론적 fixed-point 반복. TD 는 확률적 근사 (stochastic approximation) 이며 그 목표가 관측
> return 이냐(MC), bootstrap 예측이냐(TD), $\max$-bootstrap 이냐(Q-learning) 로 갈린다.

## 학습 목표
- $\gamma$-압축 사상의 고정점 반복으로서 **value iteration** 이 지수 수렴함을 직접 관측하고,
  관측 축약비 $\approx \gamma$ 를 정량 확인한다.
- **TD(0) vs MC**: bootstrap 목표의 편향-분산 trade-off 를 random walk 에서 학습률 스캔으로
  실증한다.
- **on-policy vs off-policy 제어**: cliff-walking 에서 SARSA 는 안전 경로, Q-learning 은
  위험 최적 경로를 학습한다는 고전 결과를 재현한다.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_4_01.ipynb` | Value iteration on 4×4 gridworld | Bellman backup $V_{k+1} = \mathcal T V_k$, $\gamma$-압축 | $\gamma=0.9,\ \text{tol}=10^{-10}$ 에서 수렴, 관측 축약비 $\approx \gamma$; greedy 정책이 함정을 자연스럽게 우회 |
| 2 | `CE_15_4_02.ipynb` | TD(0) vs every-visit MC on 5-state random walk | 스텝별 TD error $\delta_t = r+\gamma V(s')-V(s)$ vs 종료 후 return $G_t$ | TD(0) $\alpha\in\{0.05,0.10,0.15\}$ 모두 MC 대비 초반 하강 빠름; 100 에피소드 후 참값 $i/6$ 에 소수 셋째 자리 근사 |
| 3 | `CE_15_4_03.ipynb` | SARSA vs Q-learning on 4×12 Cliff Walk | on-policy 표본 vs off-policy $\max$ 목표, $\varepsilon$-greedy 행동 | SARSA 학습 곡선이 위쪽/안정적, greedy return $\approx -15\sim -17$; Q-learning greedy return $\approx -13$ 최적 but 학습 중 절벽으로 자주 낙하 |

## 한 줄 정리
> **밴딧에서 MDP 로**: 상태 개념 도입은 리그렛 계산을 **동적계획법**(Bellman 방정식) 으로 바꾼다.
> 모델을 알면 압축 사상 반복으로 정확히 풀리고, 모르면 TD 계열의 확률적 근사로 학습되며,
> 정책 학습에서 on/off-policy 는 **탐색 리스크의 내재화 여부** 로 갈린다.

## 다음 Day 예고
Day 72 는 **함수 근사(function approximation)** 로 넘어간다. 상태공간이 커지면 tabular $Q$ 를
유지할 수 없으므로 선형/신경망으로 대체. 이때 나타나는 **deadly triad** — bootstrap + off-policy
+ function approximation — 의 발산 사례와 완화책 (target network, replay buffer) 을 다룰 예정.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는 스케줄러가
> 로드맵을 절 단위로 자동 진행해 왔다. Day 68 §15.1 (multi-armed), Day 69 §15.2 (contextual),
> Day 70 §15.3 (non-stationary) 에 이어 Day 71 은 예고된대로 **§15.4 MDP · TD 학습** 으로 넘어가
> 밴딧에서 상태 있는 강화학습으로의 이행을 정리한다.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (가치 함수, RMSE 학습 곡선, greedy return
> 평균/표준편차) 는 코드 실행 결과이며, 시드는 각 문제에서 명시된 범위로 고정되어 있어 재현 가능.
> 노트북 실행 환경: nbclient + IPython kernel, Python 3.10, NumPy/pandas/Matplotlib 표준 스택.
