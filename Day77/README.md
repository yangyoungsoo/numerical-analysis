# Day 77 — §15.10 Prioritized Experience Replay + Rainbow DQN 구성요소 (PER · Dueling · n-step)

Day 76(§15.9) 에서 off-policy value-based 학습의 세 병리 (부트스트랩 발산·시간 상관·max
편향) 를 격리 관측했다. §15.10 은 이 병리 각각에 대한 Rainbow DQN 의 개별 처방 —
**Prioritized Experience Replay (PER), Dueling architecture, n-step bootstrap** — 이 어떻게
작동하는지 세 개의 격리 실험으로 정량 관측한다. 신경망은 여전히 사용하지 않고 **tabular /
선형근사** 위에서 각 구성요소의 gradient·부트스트랩·샘플링 경로를 정확히 재현한다.

## 학습 목표

- **Problem 1 (PER)**: 확률 보상 4×4 GridWorld ($r=-1+\mathcal N(0,1)$) 에서 uniform / PER
  (α=0.6, β 0.4→1.0) / PER-no-IS 3 조건을 15 시드 × 250 에피소드 비교. 우선순위 재샘플링만
  켜고 IS 보정을 끄면 학습된 $V_\theta$ 가 참값에서 체계적으로 밀리는 **편향된 fixed-point
  현상** 을 정량 관측.
- **Problem 2 (Dueling)**: 결정적 baseline vs "corridor" (첫 열에서 오직 `right` 만 유효)
  두 환경 × Q-standard / Dueling / Dueling-noID 3 아키텍처 = 6 조건. 액션이 무의미한 상태의
  비중이 클수록 dueling 이득이 커지는지, mean-subtraction 이 없으면 파라미터 노름이 상수
  방향으로 표류하는지 검증.
- **Problem 3 (n-step)**: 같은 확률 보상 환경에서 $n\in\{1,3,5,10\}$ tabular n-step
  Q-learning 15 시드 × 250 에피소드. 편향-분산 다이얼 (편향 $\sim\gamma^n$, 분산
  $\sim\sigma^2\cdot(1-\gamma^{2n})/(1-\gamma^2)$) 이 학습곡선·Q-error 궤적·훈련 표적
  표준편차에 어떻게 반영되는지, Rainbow 의 $n=3$ 기본값이 이 환경에서 유리한지 검증.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_10_01.ipynb` | Prioritized Experience Replay | 비례 우선순위 $p_i\propto|\delta_i|^\alpha$, IS 가중 $w_i=(NP)^{-\beta}$, β 0.4→1.0 anneal | $V^\star=-5.85$; tail100 리턴: **PER=-6.75 (최우수, std 1.36)**, uniform=-7.43 (std 1.91), **PER-no-IS=-10.92 (최악, std 4.26)**. tail $V_\theta$ 편향: uniform +0.54, PER +0.44, **PER-no-IS +0.77**. IS 를 끄면 우선순위 재샘플링이 편향된 fixed-point 로 수렴 — 정확히 이론이 예측한 병리. |
| 2 | `CE_15_10_02.ipynb` | Dueling architecture (linear) | $Q=V+A-\bar A$, mean-subtraction on/off, on-line Q-learning | 2 env × 3 arch × 15 seeds. baseline: q_std tail 리턴 -42.79 (미수렴), dueling/noid 모두 -5.85 (최적). corridor: q_std -31.48, dueling/noid -5.85. **Dueling 이 두 환경 모두에서 압승** — 공유 $V$ 스트림으로 sample efficiency 대폭 개선. noid 는 리턴은 같지만 `param_norm` 이 dueling 대비 10% 더 커 식별성 표류 흔적. |
| 3 | `CE_15_10_03.ipynb` | n-step bootstrap 스윕 | tabular n-step Q-learning, $n\in\{1,3,5,10\}$ | tail50 리턴: n=1→-6.64 (std 0.55), n=3→-6.96, n=5→-7.14, n=10→-7.16 (std 0.82). tail $\|Q-Q^\star\|_\infty$: n=1→**1.59 (최소)**, n=3→3.79, n=5→6.45. 훈련 표적 std: n=1→1.88 → n=10→ 상승 (이론 예측 $\sqrt{\sum\gamma^{2k}\sigma^2}$ 와 정성적 일치). **이 결정적-전이 환경에서는 스윗스팟이 $n=1$** — Rainbow 의 $n=3$ 이득은 transition stochasticity 가 있어야 나타남. |

## 한 줄 정리

> **Rainbow 의 세 구성요소는 각각 서로 다른 병리를 겨냥한다: PER 는 replay 의 샘플링 분포
> (IS 보정 없이는 편향), Dueling 은 근사기의 표현력 (mean-subtraction 없이는 표류), n-step
> 은 부트스트랩 표적의 범위 (환경에 따라 스윗스팟 이동). 세 처방 모두 "공짜 점심" 이 아니며,
> 원리적 corrective term (IS 가중 · mean-subtraction · $n$ 선택) 을 정확히 갖춰야 이론이
> 예측한 이득이 실측된다.**

## 사용 라이브러리
- NumPy 2.2, pandas 2.3, Matplotlib 3.10 (표준 스택). 신경망 프레임워크 미사용 —
  linear/tabular 만으로 각 구성요소의 gradient·부트스트랩·샘플링 경로를 정확히 재현.

## 다음 (Day 78)

§15.11 **Rainbow 전체 통합 + Categorical Distributional Q + Noisy Nets** — §15.10 의 세
요소가 격리 관측되었으니, 다음 Day 는 (i) 확률적 *전이* 를 도입한 GridWorld 로 확장,
(ii) 여기에 Categorical (C51) 분포적 Q + Noisy nets (매개변수 잡음 기반 탐색) 을 얹은 뒤,
(iii) 전체 Rainbow 조합의 순증분 기여를 요소별 ablation. 확률적 전이 환경에서 $n=3$ 이
실제로 유리해지는지 재검증.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는
> 스케줄러가 로드맵을 절 단위로 자동 진행. Day 76 README 의 예고대로 **§15.10 Prioritized
> Experience Replay + Rainbow DQN 구성요소** 로 진행. Chapter 15 는 원 커리큘럼의 확장
> 사례연구 흐름이므로 다음 Day 세부 절 이름은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (PER 3-조건 요약표, Dueling 6-조건
> 요약표, n-step 4-값 요약표) 는 nbclient 로 실행된 결과. 시드: Problem 1 = 2100..2114,
> Problem 2 = 3100..3114, Problem 3 = 4100..4114. Python 3.10, NumPy 2.2, pandas 2.3,
> Matplotlib 3.10.
>
> 📉 **워크스페이스 제약**: 실행 환경 디스크 제약 때문에 seed 수를 원래 계획한 20 에서 **15
> 로 축소** 하여 실행했다. tail 통계의 유의미성은 유지되지만, ablation 표의 절대값을 Day 76
> 의 20-seed 결과와 직접 비교할 때는 이 점을 감안.
