# Day 75 — §15.8 Neural Function Approximation for Policy Gradient

Day 74(§15.7) 에서 clipped surrogate 로 vanilla PG 의 학습률 붕괴를 막았지만, 그 실험은
스칼라 파라미터 정책 (short corridor) 이었다. 실제 문제로 확장하려면 **정책과 가치가
모두 함수근사** 여야 한다. §15.8 은 그 확장의 세 축을 통합한다:

1. **가치함수 회귀** — tabular vs linear vs MLP 근사의 통계적 성질,
2. **GAE ($\lambda$-return)** — 학습 중 노이지 baseline 을 사용하는 편향-분산 다이얼,
3. **완전한 PPO 파이프라인** — 8-유닛 tanh MLP 정책 + 선형 V + GAE(0.9) + clip(0.2) + entropy(5e-3).

공통 환경은 **4×4 결정적 GridWorld** — start $(0,0)$, absorbing goal $(3,3)$, 매 스텝 $-1$,
종단 보상 $0$. 균일 정책의 참값 $V^{\pi_u}(0,0)=-58.43$, 최적 정책 리턴 $=-5$ (최단 6칸,
마지막 스텝은 종단 $0$). 특징 $\varphi(s) \in \mathbb R^7$ (다항 + Manhattan) 는 세 노트북 공용.

## 학습 목표
- **Problem 1**: 균일 정책 하 $V^{\pi_u}$ 를 세 근사기로 회귀. 표본이 적은 상태에서 tabular 의
  큰 오차가 발생하고, 선형근사가 근접 상태로부터 정보를 공유해 sample-efficient 라는 것을 관찰.
- **Problem 2**: $\hat V = 0.7\,V^{\pi_u}$ (systematic bias) 위에서 GAE $\lambda$-스윕 —
  $\lambda \uparrow$ 시 편향 감소·분산 증가의 로그 그래프, 완벽한 $V$ 와의 비교.
- **Problem 3**: 통합 PPO vs vanilla PG (10 시드 × 80 배치) — 학습곡선, 시드 편차,
  성공률, 정책 엔트로피 궤적.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_8_01.ipynb` | 가치함수 회귀 | tabular MC / linear LS / MLP(H=8) | 2000 에피소드에서 tabular RMSE 0.94 (밀집), linear 1.45, MLP 2.83. **표본 50-300 구간에선 linear ≤ tabular** (예: N=50 → tab 10.5 vs lin 9.99). 방문 밀도 그라디언트가 tabular 오차의 근원. |
| 2 | `CE_15_8_02.ipynb` | GAE λ 편향-분산 스윕 | $\lambda \in \{0, 0.3, 0.6, 0.8, 0.9, 0.95, 0.99, 1\}$, 2000 rollouts × 8 λ | 완벽한 V: $\lambda=0$ MSE=0, $\lambda=1$ MSE≈2078. 편향된 $\hat V=0.7V^{\pi_u}$: $\lambda=0$ MSE=0.18, $\lambda=1$ MSE=2456. **MSE 최소 $\lambda^\star \approx 0$** (이 uniform scaling bias 는 $\delta$ 에서 상쇄되기 때문). PPO 실무값 0.9-0.97 은 훨씬 노이지한 신경망 V 를 전제. |
| 3 | `CE_15_8_03.ipynb` | PPO 완전체 vs vanilla PG | 10 seeds × 80 batches × 8 eps, GAE(0.9), clip(0.2), ent(5e-3), MLP(H=8), 선형 V | **PPO**: last-10 mean return $-7.56$ (opt=$-5$), seed std $7.51$, 성공률 **90%**, 최종 $H(\pi\|s_0)=0.065$. **Vanilla PG**: mean $-11.28$, seed std $16.38$ (2.2× 더 큼), 성공률 80%, 최종 엔트로피 $0.105$. **PPO 우위는 평균 리턴보다 시드 재현성에서 더 두드러진다.** |

## 한 줄 정리
> **가치함수 근사 (선형이든 신경망이든) 는 방문 밀도의 불균형을 흡수하는 통계적 장치이며,
> GAE 는 학습 중 이 노이지 baseline 을 얼마나 신뢰할지의 다이얼이고,
> PPO clip 은 큰 스텝의 파괴적 갱신을 잘라내는 안전핀 — 세 도구가 상보적으로 작동해
> 시드 간 재현성을 vanilla PG 대비 절반의 표준편차로 압축한다.**

## 사용 라이브러리
- NumPy 2.2, pandas 2.3, Matplotlib 3.10 표준 스택.

## 다음 (Day 76)
§15.9 **Off-Policy Value-Based Deep RL (DQN & Double DQN)** — on-policy PG 계열을 떠나
experience replay + target network 기반의 Q-learning 을 다룬다. 특히 max operator 의
overestimation bias 를 실측하고 Double DQN 이 어떻게 이를 완화하는지 이 GridWorld 위에서
재현한다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는
> 스케줄러가 로드맵을 절 단위로 자동 진행해 왔다. Day 74 README 의 예고대로
> **§15.8 Neural Function Approximation for PPO** 로 진행. Chapter 15 는 원 커리큘럼의
> 확장 사례연구 흐름이므로, 다음 Day 로 이어지는 절 이름은 스케줄러가 관례적 순서
> (§15.9 Off-Policy Value-Based Deep RL) 로 계속 이어갈 예정.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (RMSE 표, GAE 편향-분산 표, PPO vs PG
> 최종 성능표) 는 nbclient 로 실행된 결과이며, 시드는 각 문제에서 명시된 값 (`2026`, `2027`,
> `2028`, `2029`, `0..9`) 으로 고정되어 재현 가능. 노트북 실행 환경: nbclient + IPython kernel,
> Python 3.10, NumPy 2.2 / pandas 2.3 / Matplotlib 3.10.
