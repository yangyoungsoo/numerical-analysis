# Day 19 — §5.3 Simpson's Rule and Adaptive Numerical Integration (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 5. Numerical Integration
> **절**: §5.3 Simpson's Rule and Adaptive Numerical Integration — *짝수 차수 사다리의 다음 칸, 그리고 함수가 자기 분할을 가르치는 알고리즘*

## 오늘의 큰 그림
Day 18 §5.2 에서 우리는 사다리꼴 + 중점법의 *Richardson* 한 단계가 $h^2 \to h^4$ 의 *짝수 차수 점프*
를 *공짜로* 만든다는 사실을 봤다. 오늘은 그 결합 — **Simpson 공식** — 을 *자기 자신의 규칙* 으로 다지고,
그것의 두 얼굴을 차례로 본다.

세 문제를 관통하는 한 식은

$$
S_n(f) - \int_a^b f\,dx \;=\; -\,\frac{(b-a)\,h^4}{180}\,f^{(4)}(\xi) \;+\; \mathcal O(h^6).
$$

— *전역* 차수 $4$ 는 부드러움이 보장하고, *국소* 상수 $f^{(4)}(\xi)$ 는 *적응형* 이 다스린다.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **기본 차수** | $S_n - I = -(b-a) h^4 f^{(4)}(\xi)/180$ — Day 18 의 $h^2$ 에서 *두 차수* 점프 | $f \in C^4$ 가정 |
| **균등 분할의 한계** | peak/특이점이 있으면 차수가 무너지거나, *상수* 가 폭발 | $\|f^{(4)}\|_\infty$ 또는 $f \in C^4$ 의 *국소 위반* |
| **적응형의 자기조절** | $\|S^{(2)} - S^{(1)}\|/15$ 한 줄로 지역 오차 추정 → 어려운 곳만 잘게 자른다 | $|S^{(2)} - S^{(1)}| < 15\tau$ 종료 조건 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_5_3_01.ipynb`](CE_5_3_01.ipynb) | 합성 Simpson 공식 $S_n$ 을 직접 구현, 세 부드러운 적분 ($e^x$, $\sin x$, $1/(1+x^2)$) 에 적용. $n = 2,4,\ldots,2^{14}$ 에서 측정한 log–log 기울기가 $-4.0$ — 이론값 $\mathcal O(h^4)$ 와 3–4 자리 일치. 연속 비율 $\|e_n\|/\|e_{2n}\| \to 16$ 와 점근 상수 $e\,h^4/180$ 까지 확인. Day 18 사다리꼴과 같은 평면 위에서 *두 차수의 점프* 를 시각화. |
| 2 | [`CE_5_3_02.ipynb`](CE_5_3_02.ipynb) | 세 *병리적* 적분 — Runge $1/(1+25 x^2)$, 끝점 특이점 $1/\sqrt x$, 좁은 Gaussian $e^{-1000(x-1/2)^2}$ — 에 균등 분할 Simpson 을 적용. Runge 는 차수는 살아남되 *상수가 매우 크다*. $1/\sqrt x$ 는 *차수 자체가 무너져* $\mathcal O(n^{-1/2})$ 로 떨어진다. 좁은 Gaussian 은 $n < \sqrt{1000}$ 에서 peak 을 *놓쳐* spurious 결과 — *균등 표집의 함정* 을 정량화. |
| 3 | [`CE_5_3_03.ipynb`](CE_5_3_03.ipynb) | 재귀 **적응형 Simpson** — 지역 오차 추정 $\|S^{(2)} - S^{(1)}\|/15$ 와 종료 조건 $\|S^{(2)} - S^{(1)}\| < 15\tau$ 로 *자동 세분화*. 같은 평가 횟수의 균등 Simpson 보다 *수십~수억* 배 작은 오차. partition 끝점 분포 시각화로 *적응형이 어디서 자르는지* 를 직접 보여준다 — Runge 의 peak 주변에 기하급수적 누적, $1/\sqrt x$ 끝점에 *지수적* 집중, 좁은 Gaussian peak 내부에만 분할. |

## 한 줄 정리
> **Simpson 의 $\mathcal O(h^4)$ 는 *부드러움 + 균등 표집* 가정 위에서만 성립한다.**
> 어느 한쪽이 깨지면 *적응형* 이 — 한 줄짜리 Richardson 오차 추정만으로 — 함수가 자기 자신의 분할을 가르치게 한다.

## 사용 라이브러리
- NumPy (벡터화 Simpson 가중치 $[1, 4, 2, 4, \ldots, 4, 1]$)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`loglog`, `semilogy`, 영문 라벨)
- `math.erf` (Problem 2 의 좁은 Gaussian 참값)
- 외부 데이터 없음 — 전부 코드 안에서 생성

## 다음 (Day 20)
**§5.4 — Romberg Integration**. 오늘 Problem 1 의 Simpson 이 *Richardson 한 단계* 였다면, Romberg 는
*Richardson 을 무한히 쌓는다* — $\mathcal O(h^2), \mathcal O(h^4), \mathcal O(h^6), \ldots$ 의 *자유로운*
차수 사다리. 적응형이 *지역* 에서 자원을 분배했다면, Romberg 는 *전역* 으로 차수를 끌어올린다.
두 방향은 *상보적* 이며, 둘 모두를 결합한 것이 현대의 `scipy.integrate.quad`.

> 본 노트북 시리즈는 `_meta/curriculum.md` 의 *향후 (Day 14+)* 예고에 따라 챕터 5 §5.3 으로 *같은 호흡*
> 으로 이어간다.
> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
