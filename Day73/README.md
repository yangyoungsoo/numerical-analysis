# Day 73 — §15.6 Policy Gradient Methods (REINFORCE · Baseline · Actor-Critic)

Day 72(§15.5)에서 value 함수 근사가 off-policy 부트스트랩과 결합될 때 **deadly triad** 발산이
일어남을 보였다. 해결책 중 하나는 아예 **정책을 직접 파라미터화**해 gradient 상승으로
최적화하는 것 — 정책경사(Policy Gradient) 계열이다. 이 계열의 핵심 결과 세 가지 (vanilla
REINFORCE → baseline 을 추가한 REINFORCE → 상태값 critic 을 함께 학습하는 one-step
actor-critic) 를 같은 태스크 (Sutton & Barto Example 13.1, **Short Corridor with Switched
Actions**) 위에서 구현·비교한다.

$$
\underbrace{\nabla J(\theta)}_{\text{policy gradient}}
\;=\; \mathbb E_{\tau\sim\pi_\theta}\Bigl[\,\sum_t A_t\,\nabla_\theta\log\pi_\theta(a_t|s_t)\,\Bigr],
\qquad
A_t \in \{\,G_t,\; G_t - b,\; \delta_t\,\}.
$$

> **공통 태스크 — Short Corridor with Switched Actions**: 4상태 · 2행동. 상태 1 은 행동이
> 뒤바뀐다 (right ↔ left). 세 개의 비종단 상태가 같은 특성을 공유하므로 **결정론적** 정책은
> 원리적으로 실패하고, 최적은 확률적 정책 $p^\star \approx 0.59$ (사전 격자탐색 확인).
> Q-learning/SARSA 는 greedy 정책만 뽑아 이 태스크에서 발산/무한 진동한다.

## 학습 목표
- **정책경사 정리** 를 실제 코드로 재현: $\theta\gets\theta + \alpha\,G_t\,(a_t-\sigma(\theta))$.
- **baseline 정리** 의 두 얼굴 — 기대값 불변 (bias 무) 과 분산 감소 — 을 정량화.
- **Actor-critic** 에서 critic 의 편향 대가와 분산 감소 이득을 같은 태스크·같은 시드로 비교.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_6_01.ipynb` | Vanilla REINFORCE | scalar softmax policy, $\alpha=2^{-13}$, 100 runs × 1000 eps | 격자탐색 $p^\star\!\approx\!0.55$·$J^\star\!\approx\!-10.8$(MC 근사, 참조 -11.6), 최종 $P(R)=0.546$, 마지막 100 ep 평균 리턴 $-11.80$ |
| 2 | `CE_15_6_02.ipynb` | REINFORCE + learned scalar baseline | $\alpha_\theta=2^{-9}$·$\alpha_w=2^{-6}$, 학습률 $16\times$ 확대 | 그래디언트 표본 분산 $685\!\to\!256$ (**×2.68 감소**), 최종 $P(R)=0.584$, 학습된 $b\!\approx\!-10.6$, 두 곡선 같은 극한으로 수렴 (bias 무) |
| 3 | `CE_15_6_03.ipynb` | One-step actor-critic (tabular critic) | TD(0) critic $\alpha_w\!=\!2^{-4}$, 매 스텝 online policy 갱신 | 표본 분산 **×11 감소** ($21.5\!\to\!1.95$), 학습된 $V\!\approx\!(-11.6, -9.6, -5.0)$ vs MC 참값 $(-11.9,-10.3,-5.8)$, actor-critic 곡선이 REINFORCE+baseline 대비 초반 기울기 우세 |

## 한 줄 정리
> **정책경사는 확률적 최적을 직접 찾을 수 있는 유일한 계열이며, `vanilla → baseline →
> actor-critic` 로 갈수록 표본 분산이 자릿수로 감소해 학습률을 그만큼 크게 잡을 수 있다.**
> Short-corridor 는 결정론 greedy 정책이 원리적으로 실패하는 예제로, value-based 방법에서
> policy-based 방법으로 넘어가는 이유를 가장 짧게 보여주는 표준 벤치마크다.

## 다음 Day 예고
Day 74 는 **§15.7 Trust-Region / Clipped Surrogate (TRPO · PPO 아이디어)** 로 진행 예정.
Day 73 에서 본 것처럼 학습률을 크게 하면 분산이 다시 커져 학습이 불안정해질 수 있다.
정책 갱신 자체에 **KL 신뢰영역** 또는 **clipping** 을 걸어 큰 유효 학습률에서도 안정을
유지하는 실무적 표준 (PPO) 의 동기와 재현.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는
> 스케줄러가 로드맵을 절 단위로 자동 진행해 왔다. Day 68 §15.1 → Day 72 §15.5 (함수근사·deadly
> triad) 에 이어 Day 73 은 Day 72 README 의 예고대로 **§15.6 정책경사** 로 진행.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (격자탐색 $J(p)$, 100-run 평균 학습 곡선,
> 그래디언트 표본 분산, actor-critic critic 값) 는 nbclient 로 실행된 결과이며 시드는 각 문제에서
> 명시된 값 (1000+r, 2000+r, 3000+r) 으로 고정되어 재현 가능. 노트북 실행 환경:
> nbclient + IPython kernel, Python 3.10, NumPy/pandas/Matplotlib 표준 스택.
