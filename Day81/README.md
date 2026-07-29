# Day 81 — §15.14 Replay Buffer + Prioritized Experience Replay (PER) + Double DQN

Day 80 (§15.13) 은 MountainCar-lite MLP 규모에서 **target network + gradient clipping** 두
stabilizer 로 학습을 안정화하고 QR-DQN · IQN 으로 정책 성능을 개선했다. 그러나 학습 자체는
여전히 **online SGD** (매 스텝 최근 표본 1개 gradient) 였다. §15.14 는 이 마지막 남은
축을 정면 조준한다 — 표본을 **replay buffer** 에 저장해 (i) iid 근사로 gradient variance 를
줄이고, (ii) **priority ∝ |TD-error|** 로 정보량 큰 표본을 편향 표집하며, (iii) **Double
DQN** 으로 max 편향을 소거한 뒤 세 처방을 **Rainbow-lite-2** 로 통합해 각 요소의 순증분과
시너지를 정량 측정.

## 학습 목표

- **Problem 1 (uniform replay buffer)**: MountainCar-lite (force $0.005$, max 300 스텝) MLP
  ($2 \to 16 \to 3$) 에서 online SGD / replay-$B=16$ / replay-$B=32$ 3 조건을 4 시드 × 50
  에피소드 비교. 표본 재사용 · gradient variance 감소가 sample efficiency · tail-20 return ·
  시드편차에 어떻게 반영되는지 분해.
- **Problem 2 (Prioritized Experience Replay)**: uniform → PER-$\alpha$ ($\alpha=0.6$, β=0:
  IS 미적용) → PER-$\alpha\beta$ ($\beta_0=0.4 \to 1$ annealing) 3 조건 비교. 편향 표집이
  학습 신호에 미치는 영향과 IS 가중치의 편향 보정 필요성을 실측.
- **Problem 3 (Double DQN + Rainbow-lite-2 ablation)**: baseline (replay-32) / +Double /
  +PER / **Rainbow-lite-2 = Double + PER + n=3** 4 조건을 3 시드 × 40 에피소드로 학습해
  요소별 순증분 · 시드편차 감소를 정량화. Day 78/79 의 "요소 통합의 시너지" 관찰이
  MLP + PER 규모에서 어떻게 재현되는지 (또는 다른 축으로 재분배되는지) 검증.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_14_01.ipynb` | uniform replay buffer | 순환 버퍼 $N=5{,}000$ · 미니배치 표집 · target net + clip 유지 | tail20: **online $-28.6 \pm 4.2$ / replay-16 $-42.1 \pm 22.9$ / replay-32 $-24.3 \pm 0.88$**. replay-32 는 online 대비 mean 을 살짝 개선 (+4.3) 하고 **시드편차를 $4.2 \to 0.88$ 로 $4.8\times$ 감소**. gradient norm 분포도 좁아지고 p95 가 $17.7 \to 7.3$ 으로 축소. 반면 replay-16 은 배치 부족으로 일부 시드에서 학습 실패. |
| 2 | `CE_15_14_02.ipynb` | Prioritized Experience Replay | $P(i) \propto p_i^{0.6}$ · IS weight $w_i = (NP(i))^{-\beta}$ · β annealing | tail20: **uniform $-24.9 \pm 0.7$ / PER-α $-160.5 \pm 86.2$ (학습 실패) / PER-α+β $-35.0 \pm 14.8$**. **예측 반전 관측**: 이 소규모 MLP 규모에서는 순수 PER 이 편향된 표적으로 학습을 붕괴시키며, IS 보정을 켜야 uniform 수준으로 회복은 하되 추가 이득 없음. mean IS weight 는 $\beta$ annealing 을 따라 $1 \to 0.45$ 로 감소 (편향 소거가 실제 작동). PER 의 이점은 더 큰 신경망 · sparse 보상 · 긴 학습에서 발현 (Day 79 IQN 관찰과 동종 체제 경계 현상). |
| 3 | `CE_15_14_03.ipynb` | Double DQN + Rainbow-lite-2 | Double target ($a^\ast = \arg\max Q_\theta$, 평가 $Q_{\theta^-}$) · PER-α+β · n=3 부트스트랩 | tail20: **baseline $-38.8 \pm 14.8$ / +Double $-31.2 \pm 6.7$ / +PER $-26.6 \pm 1.9$ / Rainbow-lite-2 $-24.2 \pm 0.16$**. 조건별 순증분: +Double $+7.6$, +PER $+12.2$, Rainbow $+14.6$. **mean 관점에서는 산술합 이하 (19.7 vs 14.6)** 이나, **시드편차 관점에서 극단적 시너지**: $14.8 \to 0.16$ ($92\times$ 감소). Rainbow-lite-2 는 모든 시드에서 사실상 동일한 tail-20 $\approx -24$ 도달. |

## 한 줄 정리

> **Replay buffer 는 iid 근사로 gradient variance 를 줄이고 시드편차를 $4.8\times$ 감소시켜
> "정책 성능의 재현성" 이라는 축에서 online SGD 대비 결정적 이득을 낸다. PER 은 소규모
> MLP 에서 IS 보정 없이는 학습을 붕괴시키며 (β=0 → tail20 $-24.9 \to -160.5$), IS
> annealing 을 켜면 안전한 baseline 을 회복. Double DQN + PER + n-step 을 통합한
> Rainbow-lite-2 는 tail-20 mean 을 $-38.8 \to -24.2$ 로 끌어올리는 동시에 시드편차를
> $14.8 \to 0.16$ ($92\times$) 로 압축한다 — 요소 통합의 시너지는 mean 이 아닌 "실무
> reproducibility" 라는 축에서 결정적으로 나타난다.**

## 사용 라이브러리
- NumPy, pandas, Matplotlib (표준 스택). **신경망 프레임워크 미사용** — 2-층 MLP forward
  · backward, PER buffer (우선도 배열 + 배치 표집 + IS weight), Double DQN target,
  n-step 부트스트랩 큐를 모두 수기로 numpy 구현. 각 요소의 gradient · 표적 · 표집 경로를
  정확히 재현.

## 다음 (Day 82)

§15.15 **Dueling network 구조** ($Q(s,a) = V(s) + A(s,a) - \bar A(s)$ 분해) — 상태값 $V$ 와
행동이점 $A$ 를 별도 head 로 분리해 표현력을 넓히고, 위 4 조건 위에 dueling 을 얹은
**Rainbow-lite-3** 를 검증. Problem 3 에서 관측한 "mean vs variance 시너지의 축 분리" 가
dueling 추가 후에도 유지되는지, MLP 규모의 max 편향 · 분포 표적 · 우선표집 · 이점분해
4 축이 결합될 때 gradient · Q-error · 정책 안정성 트레이드오프가 어떻게 재분배되는지 정량화.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3) 까지이며, Day 14 이후
> (§4 보간부터 시작해 Ch 15 확장 사례연구까지) 는 스케줄러가 절 단위로 자동 진행. Day 80
> README 의 "다음 (Day 81)" 예고대로 **§15.14 replay + PER + Double DQN** 로 진행. Chapter
> 15 확장 사례연구는 원 커리큘럼의 확장 흐름이므로 다음 Day 세부 절 이름 (§15.15 dueling)
> 은 스케줄러가 관례적 순서로 계속 이어갈 예정.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북의 모든 수치 (Problem 1 의 3-조건 요약, Problem 2 의
> PER 3-조건 + IS annealing, Problem 3 의 4-조건 ablation + waterfall 분해) 는 in-process
> exec 로 실행된 결과. 시드: Problem 1 = 8301..8304, Problem 2 = 8401..8403, Problem 3 =
> 8501..8503. Python 3.10, NumPy, pandas, Matplotlib 표준 스택. MLP + PER + Double + n-step
> 을 수기 numpy 로 구현 (신경망 프레임워크 미사용).
>
> 🔎 **정직성 노트**: Problem 2 에서 순수 PER (β=0) 이 IS 보정 없는 uniform 대비 학습을
> 붕괴시킨 결과는 Schaul et al. (2016) 의 원 주장 (신경망 · Atari 규모에서의 우수성) 과
> 모순되지 않는다. 소규모 MLP + short-horizon 은 PER 의 이점이 발현되는 체제 밖이며,
> Problem 2 의 관찰은 이 경계선 자체를 정량화한 결과 (Day 79 IQN 관찰과 동종).
>
> 📉 **워크스페이스 제약**: 실행 환경 디스크 제약 때문에 시드/에피소드 수를 Day 80 대비
> 축소 (Problem 1: 4 시드 × 50, Problem 2: 3 시드 × 50, Problem 3: 3 시드 × 40). tail
> 통계의 상대 순위는 유지되지만, 절대값 비교 시 이 점 감안.
