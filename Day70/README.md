# Day 70 — §15.3 Non-Stationary Bandits (비정지 밴딧: 잊음이 이득이 되는 경계)

Day 68(§15.1) 에서 정지 밴딧의 세 정책 — $\varepsilon$-greedy, UCB1, TS — 을 다뤘고, Day 69(§15.2) 는
문맥 벡터를 얹어 LinUCB / LinTS 로 확장했다. 두 경우 모두 **정지 가정**이었다.
§15.3 은 이 가정을 깨고 팔의 평균 $\mu_k(t)$ 가 **시간에 따라 변하는** 비정지(non-stationary) 밴딧을
다룬다. 두 대표 전략: (i) 최근 $w$ 개 표본만 사용하는 **Sliding-Window UCB**, (ii) 과거 표본에 지수
감쇠 $\gamma^{t-s}$ 를 부여하는 **Discounted UCB**.

$$
\text{정지 UCB1 } (R_T = O(\log T))\ \xrightarrow{+\text{변화점 } \Upsilon_T}\ \text{선형 리그렛 위험}\
\xrightarrow[\text{soft weighting}]{\text{hard cutoff}}\ \text{SW-UCB},\ \text{D-UCB}.
$$

> **공통 축**: **잊음(forgetting) 을 통한 편향-분산 재조정**. 잊음이 크면 변화 추적력 ↑ / 잡음 ↑;
> 잊음이 작으면 정지 구간 효율 ↑ / 변화점 대응 ↓. 이 균형은 문제 규모($T$, $\sigma$, $\Delta$) 에
> 크게 좌우된다.

## 학습 목표
- **SW-UCB** 를 급격한 변화점 (abrupt change-point) 시나리오에서 실행하고, 이론이 예언하는 "UCB1 실패,
  SW-UCB 성공" 의 격차가 **어느 규모에서** 실제로 관찰되는지를 정량화한다.
- **Discounted UCB (D-UCB)** 를 매끄러운 sinusoidal 드리프트에서 실행하고, tracking 정확도 vs 리그렛
  이 항상 같은 방향이 아님을 정량 확인한다.
- $w$ 와 $\gamma$ 를 로그 스캔해, sweet spot 의 위치가 **드리프트 성격** 과 **horizon** 에 어떻게
  의존하는지 실증한다.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_3_01.ipynb` | Abrupt change-point 시나리오 | SW-UCB (Garivier-Moulines), $B\sqrt{\log \min(t,w)/N_k^{(w)}}$ | $K{=}5,\ T{=}4000,\ \tau{=}2000$. UCB1 $R_T=144\pm 17$ (arm 5 을 후반 94.5% 로 이동, 자기적응). SW-UCB($w{=}100$) $R_T=864\pm 10$ 오히려 6배 손해 — **작은 $w$ 는 정지 구간 손해가 변화점 이득보다 큼** |
| 2 | `CE_15_3_02.ipynb` | Smooth sinusoidal drift | D-UCB (Kocsis-Szepesvári), $2B\sqrt{\log n_t(\gamma)/N_k^{(\gamma)}}$, 지수 감쇠 업데이트 | $K{=}4,\ T{=}6000,\ P{=}1500$. UCB1 $R_T=281\pm 12$; D-UCB($\gamma{=}0.995$) $R_T=943$, $\gamma{=}0.99$ $R_T=1092$. **Tracking 은 개선되나 exploration 오버헤드로 리그렛은 악화** |
| 3 | `CE_15_3_03.ipynb` | 스캔: $w$-스캔 & $\gamma$-스캔, abrupt vs smooth | 각 조합 M 회 반복 후 sweet spot 탐색 | 두 정책 모두 sweet spot 이 $w{=}\infty$ / $\gamma{=}1$ (= UCB1) 에서. 이론 sweet spot ($w^\star{\sim}\sqrt{T/\Upsilon_T}$) 은 **훨씬 큰 $T$ 에서 만** 관찰됨 |

## 한 줄 정리
> **비정지 밴딧의 실전 교훈**: 이론이 예언하는 UCB1 선형 리그렛과 SW-UCB/D-UCB 부분선형 리그렛의
> 격차는 **문제 규모**에 강하게 의존한다. $T \le 6000$, $\sigma \gtrsim 0.1$ 규모에서는 UCB1 이 견고하며,
> 잊음 매커니즘 도입은 오히려 손해가 될 수 있다. sweet spot 은 (a) 매우 긴 horizon, (b) 낮은 잡음,
> (c) 여러 change point 인 경우에만 나타난다.

## 다음 Day 예고
§15.3 까지 밴딧 (상태 없음) 을 끝냈다. Day 71 는 Ch 15 후반부 — **MDP · TD 학습** 으로 넘어가,
상태 개념을 도입해 순차 결정을 일반화한다. 밴딧의 리그렛 이야기가 상태 공간을 갖는 강화학습으로
자연스럽게 확장된다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는 스케줄러가
> 로드맵을 절 단위로 자동 진행해 왔다. Day 68 §15.1 (multi-armed), Day 69 §15.2 (contextual) 에 이어
> Day 70 은 예고된대로 **§15.3 non-stationary bandits (SW-UCB, D-UCB)** 를 다룬다.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치 (누적 리그렛 평균/표준편차, sweep sweet-spot
> 위치) 는 `nbclient`/`nbconvert --execute` 로 실제 실행해 얻은 값이며, 본문 해석과 완전히 일치한다.
> Monte-Carlo 반복 $M\in\{12, 50, 60\}$, seed 는 정책 간 공정 비교를 위해 각 반복마다 재사용
> (seed = 10000+m / 20000+m / 30000+m).
>
> ⚠️ **honest reporting**: 본 Day 의 실측 결과는 이론적 예언 ("UCB1 은 비정지에서 선형 리그렛, SW-UCB/D-UCB 는
> 부분선형") 과 **부분적으로 어긋난다**. 그 이유 (horizon 규모, 신뢰항 크기) 는 각 노트북의 §4 에
> 자세히 논의했다. 실전에서 잊음 매커니즘을 **무비판적으로 도입하면 손해**라는 실용적 교훈을
> 정량적으로 보여주는 데이터로 남긴다.
