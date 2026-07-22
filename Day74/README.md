# Day 74 — §15.7 Trust-Region and Clipped Surrogate Methods (TRPO · PPO)

Day 73(§15.6)에서 정책경사 계열이 `vanilla REINFORCE → +baseline → actor-critic` 로 갈수록
표본 분산을 자릿수로 줄일 수 있음을 보였다. 그러나 분산이 작아졌다고 곧바로 학습률을
크게 잡을 수 있는 것은 아니다. 큰 $\alpha$ 는 로짓을 포화시키거나 정책을 반대편 안장점으로
날려버려 학습을 붕괴시킨다. **§15.7** 은 그래서 정책 갱신 자체에 *trust region* 을 부여하는
두 표준을 다룬다.

$$
L^{\text{CLIP}}(\theta) = \mathbb E_t\bigl[\min\bigl(r_t(\theta) A_t,\;
\text{clip}(r_t(\theta), 1-\varepsilon, 1+\varepsilon)\, A_t\bigr)\bigr],
\qquad r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_\text{old}}(a_t|s_t)}.
$$

$$
L^{\text{KLPEN}}(\theta) = \mathbb E_t[r_t(\theta) A_t]
- \beta\; \overline{\mathrm{KL}}\bigl(\pi_{\theta_\text{old}} \| \pi_\theta\bigr),
\qquad
\beta \leftarrow \begin{cases} 2\beta & \overline{\mathrm{KL}} > 1.5\,\delta \\
\beta/2 & \overline{\mathrm{KL}} < \delta/1.5. \end{cases}
$$

> **공통 태스크 — Short Corridor with Switched Actions** (Sutton & Barto Ex. 13.1, Day 73 과 동일).
> 스칼라 softmax 정책 $\pi_\theta(\text{right}) = \sigma(\theta)$. 최적 확률 정책 $p^\star \approx 0.59$,
> 참 리턴 $J^\star \approx -11.6$. 스칼라이므로 $\mathrm{KL}(\pi_\text{old}\|\pi_\theta)$ 를 닫힌형으로
> 정확히 잴 수 있어 trust-region 계열을 이론에 가장 가깝게 재현 가능.

## 학습 목표
- **동기 (Problem 1)**: vanilla REINFORCE 의 학습률 상한을 $\alpha$-스윕으로 관측 — 큰 $\alpha$ 에서
  분산 증폭 → 로짓 포화 → 학습 정지 라는 두-단계 실패를 확인.
- **PPO (Problem 2)**: clipped surrogate 를 구현해 붕괴하지 않는 학습을 시각적·정량적으로 확인,
  batch 정규화만으로도 vanilla PG 가 부분 안정화됨을 함께 관찰.
- **TRPO/PPO 하이퍼파라미터 (Problem 3)**: clip $\varepsilon$ 민감도를 훑고, TRPO 원형 아이디어인
  **adaptive KL-penalty** 를 clip 과 정면 비교 (PPO 논문의 원 결과 재현).

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_7_01.ipynb` | Vanilla REINFORCE $\alpha$-스윕 | per-episode PG, $\alpha \in \{2^{-13},2^{-10},2^{-8},2^{-6},2^{-4}\}$, 40 seeds × 1000 ep | 격자탐색 $p^\star\!\approx\!0.55$, $J^\star\!\approx\!-11.2$. 수렴 성공률 $\alpha=2^{-8}\!\to\!87.5\%$, $2^{-6}\!\to\!30\%$, $\mathbf{2^{-4}\!\to\!0\%}$ (모든 시드가 극단으로 튄 뒤 max-step 500 에 걸림) |
| 2 | `CE_15_7_02.ipynb` | PPO clipped surrogate | $\alpha\!=\!2^{-4}, \varepsilon\!=\!0.2, K\!=\!4$, 30 seeds × 30 batches × 40 eps | PPO tail 리턴 $-12.00$ · seed std $0.75$ · 96.7% 수렴; batch 정규화된 REINFORCE 도 $-12.22$ 로 수렴. PPO 의 배치 KL median $\sim\!5\!\times\!10^{-3}$ (TRPO 목표 $\delta\!=\!10^{-2}$ 근처) |
| 3 | `CE_15_7_03.ipynb` | Clip $\varepsilon$ 스윕 + adaptive KL-penalty | $\varepsilon\!\in\!\{0.05,0.10,0.20,0.40\}$ / KLPEN $\delta\!=\!10^{-2}$, 20 seeds | clip=$0.2$ 100% 수렴 tail $-12.0$ 로 최우수; $\varepsilon\!=\!0.05$ 는 45% 만 수렴 (너무 느림); **KLPEN adaptive 는 50% 수렴 tail $-256$ 로 clip 대비 열세** (β 궤적이 $[10^{-3}, 10]$ 로 폭발), Schulman et al. 2017 결론 재현 |

## 한 줄 정리
> **PPO 의 clipped surrogate 는 vanilla PG 가 붕괴하는 학습률 ($\alpha = 2^{-4}$) 에서도 안정적으로
> 큰 스텝을 낼 수 있게 하며, 이 스칼라 태스크에서는 clip 이 adaptive KL-penalty 보다 실무적으로
> 훨씬 강건하다 — TRPO 논문에서 시작해 PPO 논문에서 clip 이 표준으로 자리잡은 이유가 가장 단순한
> 세팅에서 정확히 재현된다.**

## 다음 Day 예고
Day 75 는 **§15.8 신경망 함수근사 하의 PPO** — 스칼라 정책이 아닌 다층 퍼셉트론 정책에서
PPO 를 CartPole/LunarLander 같은 표준 벤치마크에 적용, GAE ($\lambda$-return) 와 entropy bonus,
value function head 를 함께 다루는 실무 파이프라인으로 넘어간다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는
> 스케줄러가 로드맵을 절 단위로 자동 진행해 왔다. Day 73 README 의 예고대로 **§15.7 Trust-Region /
> Clipped Surrogate** 로 진행. Chapter 15 는 원 커리큘럼의 확장 사례연구 흐름이므로 다음 Day 로
> 이어지는 세부 절 이름은 스케줄러가 관례적 순서 (§15.8 함수근사 하의 PPO) 로 계속 이어갈 예정.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (α 스윕 결과 표, PPO vs REINFORCE 비교,
> clip ε 스윕, KLPEN β 궤적) 는 nbclient 로 실행된 결과이며 시드는 각 문제에서 명시된
> 값 (10000+..., 20000+r, 30000+r, 40000+..., 50000+r) 으로 고정되어 재현 가능. 노트북 실행 환경:
> nbclient + IPython kernel, Python 3.10, NumPy 2.2 / pandas 2.3 / Matplotlib 3.10 표준 스택.
