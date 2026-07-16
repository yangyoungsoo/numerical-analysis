# Day 72 — §15.5 Value Function Approximation & the Deadly Triad (부트스트랩 · off-policy · 함수근사)

Day 71(§15.4) 은 tabular MDP 위에서 value iteration, TD(0), SARSA/Q-learning 이
같은 Bellman 방정식을 다르게 푸는 세 얼굴임을 봤다. 상태공간이 커지면 tabular 표현이
불가능해지고 **함수근사** $\hat V(s;\mathbf w) = \mathbf x(s)^\top \mathbf w$ 를 도입해야
한다. 그런데 여기서 세 요소 — **bootstrap + off-policy + function approximation** — 가 동시에
성립하면 **가중치가 발산**할 수 있다. Sutton & Barto 는 이를 *deadly triad* 라 부른다.

$$
\underbrace{\text{TD}}_{\text{bootstrap}} \;+\; \underbrace{b \ne \pi}_{\text{off-policy}} \;+\; \underbrace{\hat V = \mathbf x^\top \mathbf w}_{\text{function approx.}}
\;\;\Longrightarrow\;\; \|\mathbf w_t\| \to \infty \text{ possible.}
$$

> **공통 축**: 세 문제 모두 같은 반례(Baird) 또는 그 정신적 확장을 다룬다. 문제 1은
> **발산의 재현**, 문제 2는 **한 요소(off-policy) 제거로 안정화** (Mountain Car + tile-coding
> + on-policy SARSA), 문제 3은 **DQN 계열 완화책** (replay buffer, target network) 의 정량 비교.

## 학습 목표
- Baird 7-state 반례로 deadly triad 의 **발산을 코드 위에서 직접 관측**하고, 참 가치 $V=0$ 이
  표현공간 안에 있음에도 안정 고정점이 아님을 확인한다.
- On-policy 를 유지한 채 **tile-coding + semi-gradient SARSA** 로 Mountain Car 를 학습해
  선형 함수근사가 실용적으로 잘 동작함을 보인다.
- Off-policy 부트스트랩을 유지하되 **replay buffer** 와 **target network** 각각/조합의
  효과를 노름 궤적과 RMS Bellman error 로 정량화한다.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_5_01.ipynb` | Baird's counterexample | off-policy semi-gradient TD, importance-sampling ratio $\rho=7$ | $\alpha=0.01,\ 2000$ 스텝 후 $\|\mathbf w\|\approx 4.6\times 10^3$ 발산 vs on-policy $\approx 9.05$, off-policy $n$-step MC $\approx 10.3$ 유계 |
| 2 | `CE_15_5_02.ipynb` | Semi-gradient SARSA on Mountain Car | 8 tilings × 8×8 tile-coding, $\alpha=0.5/M$, $\varepsilon$-decay | 500 에피소드 학습으로 초기 $\bar{\text{steps}}\approx 367$ → 후반 $\approx 131$, greedy 평가 100 에피소드 평균 $122\pm 20$ 스텝, cost-to-go 히트맵이 계곡→언덕 구조를 형성 |
| 3 | `CE_15_5_03.ipynb` | Deadly-triad 완화 (Baird) | replay buffer $(C{=}5000, B{=}32)$, target network $(K{=}100)$ | A baseline $\|\mathbf w\|\!\approx\!4.6\text{e}3$, B replay $\!\approx\!4.9\text{e}3$, **C target-net $\!\approx\!64$**, **D replay+target $\!\approx\!63$**; RMS Bellman error 는 $10^3 \to 1.1 \to 0.77$ 로 3자리 감소 |

## 한 줄 정리
> **함수근사로 넘어가는 순간 안정성은 공짜가 아니다.** On-policy 를 유지하면 (문제 2) 선형
> 함수근사는 실용적으로 잘 수렴하지만, off-policy 부트스트랩과 결합되면 (문제 1) Baird 발산이
> 벌어진다. **target network 지연**과 **replay buffer 표본** 조합이 (문제 3) 발산을 3자리
> 완화하며, 이는 DQN 계열이 성공한 실무적 이유다.

## 다음 Day 예고
Day 73 는 **§15.6 Policy Gradient (REINFORCE, actor-critic)** 로 넘어간다. Value 기반이
겪는 발산 문제를 정책을 직접 파라미터화해 우회하고, baseline 도입과 actor-critic 로
분산을 감소시키는 표준 결과를 재현할 예정. 이후 §15.7 에서 자연스러운 확장으로
**trust-region / clipped surrogate (PPO 아이디어)** 로 이어진다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는
> 스케줄러가 로드맵을 절 단위로 자동 진행해 왔다. Day 68 §15.1 → Day 71 §15.4 (MDP · TD) 에
> 이어 Day 72 는 예고된대로 **§15.5 함수근사 · deadly triad** 로 진행, tabular 에서 함수근사로
> 넘어가는 이행 구간을 정리한다.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (Baird 노름 궤적, Mountain Car greedy
> 평가, deadly-triad 완화 비교) 는 nbclient 로 실행된 결과이며 시드는 각 문제에서 명시된
> 값으로 고정되어 재현 가능. 노트북 실행 환경: nbclient + IPython kernel, Python 3.10,
> NumPy/pandas/Matplotlib 표준 스택.
