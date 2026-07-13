# Day 69 — §15.2 Contextual Bandits (문맥적 밴딧: 특징 조건부 순차 결정)

Day 68(§15.1)에서 정지(stationary) 밴딧의 세 정책 — $\varepsilon$-greedy, UCB1, Thompson Sampling — 을
탐색 강도의 자동 감쇠라는 관점에서 정면 비교했다. §15.2 는 그 위에 **문맥 벡터** $x_t$ 를 얹어,
"이번 라운드의 상태"가 팔의 기대 보상에 영향을 주는 **문맥적 밴딧(contextual bandit)** 로 확장한다.

$$
\text{정지 밴딧}\ \xrightarrow{+\text{context }x_t}\ \text{contextual bandit}\ \Rightarrow\
\text{LinUCB } (\tilde O(d\sqrt T)),\ \text{LinTS } (\tilde O(d^{3/2}\sqrt T)).
$$

> **공통 축**: 리지회귀 추정 $\hat\theta_k = A_k^{-1} b_k$ 를 유지하면서 **탐색을 어떻게 부여하느냐** —
> 결정론적 상한(LinUCB) vs 사후 표본(LinTS) — 이 정책의 이름을 결정한다.

## 학습 목표
- **LinUCB** 에서 신뢰반경 $\alpha\sqrt{x^\top A_k^{-1} x}$ 의 역할을 이해하고, $\alpha$ 상수가 리그렛
  기울기를 어떻게 조율하는지 Monte-Carlo 로 검증한다. $\alpha=0$ 고착 / 큰 $\alpha$ 상수 기울기 / 중간
  sweet spot 의 편향-분산 상충을 정량화한다.
- **LinTS** 의 켤레 사후 $\mathcal N(\hat\theta_k, v^2 A_k^{-1})$ 로부터 표본 sampling 만으로 자연스러운
  확률적 탐색을 만들어냄을 이해하고, $v$-감도를 LinUCB 대비 정량 비교한다.
- 문맥 차원 $d$ 를 로그 스캔하며 두 정책의 리그렛 지수 $R_T \propto d^{\beta}$ 를 log-log 회귀로
  경험적으로 추정하고, 이론값 $\beta_{\text{UCB}}=1$, $\beta_{\text{TS}}=1.5$ 와 대조한다.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_2_01.ipynb` | LinUCB 와 신뢰 상수 $\alpha$ | 리지 $\hat\theta_k = A_k^{-1}b_k$, $\alpha\sqrt{x^\top A_k^{-1}x}$ | $K{=}5,\ d{=}6,\ T{=}1500,\ M{=}60$. $\alpha=0$ 고착·큰 분산, $\alpha=3$ 상수 기울기, $\alpha\in[0.1,1]$ sweet spot |
| 2 | `CE_15_2_02.ipynb` | LinTS 와 사후 스케일 $v$ | $\tilde\theta_k \sim \mathcal N(\hat\theta_k, v^2 A_k^{-1})$, Cholesky sampling | $v\in\{0.1,\dots,2\}$ 스캔, LinUCB($\alpha=1$)와 동시 비교. $v\approx 0.5\sim 1$ 이 실용적 sweet spot |
| 3 | `CE_15_2_03.ipynb` | 차원 감도 (log-log 지수) | $d\in\{2,4,8,16,32\}$, $\log R_T = c + \beta\log d$ | LinUCB $\hat\beta \approx 1$, LinTS $\hat\beta \approx 1.5$ — 이론 $O(d)$ vs $O(d^{3/2})$ 격차의 실증 |

## 한 줄 정리
> **LinUCB = 결정론적 상한, LinTS = 사후 표본.** 저차원에서는 튜닝 편의로 LinTS 가 유리하나,
> **문맥 차원이 커질수록** LinUCB 의 결정론적 상한이 리그렛 스케일에서 확실히 이긴다.

## 다음 Day 예고
§15.2 는 정지 문맥 밴딧을 다뤘다. 다음 Day 는 §15.3 **비정지 밴딧(non-stationary bandit)** —
$\theta_k^\star$가 시간에 따라 드리프트할 때의 **discounting UCB**·**sliding-window UCB** — 로 이어간다.
Ch 14 의 드리프트/온라인 학습 이야기와 자연스럽게 결합된다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day 14 이후는 스케줄러가
> 로드맵을 절 단위로 자동 진행해 왔다. Day 68 이 §15.1 (multi-armed bandits) 을 마쳤으므로 Day 69 는
> `Day 69+` 로 기록되어 있던 **§15.2 Contextual Bandits (LinUCB, LinTS)** 를 채운다.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치(리그렛 평균/표준편차·$\hat\beta$ 회귀 추정치)는
> `nbclient` 로 실제 실행해 얻은 값이며, 본문 해석과 일치한다. Monte-Carlo 반복 $M \in \{40, 60\}$,
> 시드는 정책 간 공정 비교를 위해 각 라운드마다 재사용(seed = 5000+m / 7000+m).
