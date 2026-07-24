# Day 76 — §15.9 Off-Policy Value-Based Deep RL (Q-learning · DQN · Double DQN)

Day 75(§15.8) 에서 **on-policy** 정책경사 계열 (PPO + GAE) 을 완성했다. §15.9 는 이 흐름을
떠나 **off-policy value-based** 학습으로 진입한다. 학습의 주체가 정책 파라미터에서
**행동가치함수 $Q(s,a)$** 로 옮겨가며, 이에 따라 세 가지 실무적 골칫거리가 등장한다:

1. **부트스트랩 표적의 자기참조** — 함수근사 하에서 $\max_{a'} Q_\theta$ 를 표적으로 쓰면
   $\theta$ 갱신이 자기 참조로 발산 가능.
2. **표본의 시간 상관** — 온-라인 (s,a,r,s') 시퀀스는 강한 자기상관을 가져 semi-gradient
   업데이트를 편향시킴.
3. **max 연산자의 상향편향** — 잡음/근사 오차가 낀 $Q$ 에서 $\max$ 는 참값을 위로 편향
   (Jensen). 부트스트랩을 통해 이 편향이 누적된다.

세 노트북은 위 세 문제를 (i) tabular 베이스라인에서 확인하고, (ii) DQN 이 도입한 두 안정화
장치 (experience replay + target network) 의 개별 기여를 격리하며, (iii) Double Q-learning
이 최대치 편향을 어떻게 제거하는지를 각각 정량 관측한다.

공통 환경은 Day 75 와 동일한 **4×4 결정적 GridWorld** — start $(0,0)$, absorbing goal
$(3,3)$, 매 스텝 $-1$, 종단 후 $0$. 최적값 $V^\star(0,0) = -\sum_{k=0}^{5}\gamma^k \approx -5.85$
($\gamma=0.99$). Chapter 15 를 관통하는 벤치마크로 계속 재사용.

## 학습 목표

- **Problem 1**: tabular Q-learning 의 수렴을 $(\alpha,\varepsilon)$ 격자에서 관측. 최적
  $Q^\star$ 대비 오차 $\|Q-Q^\star\|_\infty$ 의 지수감쇠 + $O(\alpha)$ 상수 잡음 볼 —
  Robbins–Monro 이론의 재현. 학습된 greedy 정책을 화살표 필드로 시각화.
- **Problem 2**: 선형 근사기 $Q_\theta=\theta^\top\phi$ 에서 DQN 의 두 안정화 요소
  (replay, target-net) 를 4 조합 ablation. 리턴, $\|\theta\|_2$, TD 잔차 분산의 궤적을 비교.
  이 소규모 결정적 환경에서 어떤 조합이 실제로 이기는지 정직하게 관측.
- **Problem 3**: 확률적 보상 GridWorld ($r=-1+\mathcal N(0,\sigma^2)$) 위에서 표준
  Q-learning 의 최대치 편향을 세 $\sigma$ 레벨로 측정. **Double Q-learning** (Hasselt 2010)
  으로 편향이 사라지는지 검증.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_9_01.ipynb` | Tabular Q-learning 베이스라인 | ε-greedy + off-policy TD-control, $(\alpha,\varepsilon)$ 격자 스윕 20 시드×400 ep | $V^\star=-5.852$; 50 에피소드에서 리턴 $-6.55$, 400 에피소드 tail $-6.30$; $\|Q-Q^\star\|_\infty$: 초기 6.4 → 400 ep 1.5 (지수감쇠 후 $O(\alpha)$ 잡음 볼). 스윗스팟 $\alpha=0.3, \varepsilon=0.01 \to -6.04$. 학습된 greedy 는 정확한 최단경로. |
| 2 | `CE_15_9_02.ipynb` | Replay / target-net ablation (선형 근사) | 8 시드 × 150 ep × 4 조합, BATCH=16, TARGET_C=20 | tail return: **no-rep\|target = -11.22 (최우수, std 1.18)**, no-rep\|no-tgt = -13.05, rep\|no-tgt = -60.20, rep\|target = -73.50. **replay 는 이 소규모 결정적 환경에서 오히려 하락** (초기 wandering 궤적이 버퍼를 점유). target-net 만으로 TD 분산 1.14 → 0.84. |
| 3 | `CE_15_9_03.ipynb` | Double Q-learning: max 편향 정량화 | 20 시드 × 800 ep × 3 $\sigma$, tabular | $V^\star=-5.85$. Q-learn bias (tail100): $\sigma=0.5 \to 0.08$, $\sigma=1 \to 0.13$, $\sigma=2 \to 0.16$ (모두 **양의 상향편향**). Double Q bias: $-0.24, -0.27, -0.59$ (약간 하향으로 넘어가지만 상향편향은 **소거**). 리턴은 대동소이 (결정적 그리드는 $\arg\max$ 만 유지되면 정책 안정). |

## 한 줄 정리

> **Off-policy 가치기반 학습의 세 병리 — 부트스트랩 발산·시간 상관·max 편향 — 은 서로 다른
> 처방을 요구한다. target-net 은 (i) 을 저비용으로 다루며 거의 항상 이득이지만, replay 의
> 이득은 근사기 크기·환경 확률성·상태공간 다양성이 커져야 나타난다. max 편향의 근본 해결은
> Double Q-learning 의 selection/evaluation 분리이며, 이 아이디어가 신경망 위에서 Double DQN
> 으로 확장된다.**

## 사용 라이브러리
- NumPy 2.2, pandas 2.3, Matplotlib 3.10 (표준 스택). 신경망 프레임워크 미사용 —
  선형 근사기와 tabular 만으로 세 병리를 격리 관측.

## 다음 (Day 77)

§15.10 **Prioritized Experience Replay + Rainbow DQN 구성요소** — Problem 2 에서 uniform
replay 의 한계를 봤으니, 자연스러운 다음 절은 (i) TD-error 크기 기반 우선순위 샘플링,
(ii) dueling network 로 advantage 와 value 를 분리, (iii) n-step bootstrap 으로 편향-분산
다이얼. 여전히 같은 4×4 GridWorld 위에서 각 구성요소의 순증분 기여를 격리 측정한다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는
> 스케줄러가 로드맵을 절 단위로 자동 진행. Day 75 README 의 예고대로 **§15.9 Off-Policy
> Value-Based Deep RL (DQN & Double DQN)** 로 진행. Chapter 15 는 원 커리큘럼의 확장 사례연구
> 흐름이므로 다음 Day 세부 절 이름은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (Q-learning 학습곡선, 4-조합 ablation
> 요약표, Double Q 편향 표) 는 nbclient 로 실행된 결과. 시드 각각 명시 (Problem 1: 1000..1019
> 학습곡선용 + 2000..2007 격자, Problem 2: 3000..3007, Problem 3: 4000..4019). Python 3.10,
> NumPy 2.2, pandas 2.3, Matplotlib 3.10.
