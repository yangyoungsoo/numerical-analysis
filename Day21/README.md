# Day 21 — §5.5 Gaussian Quadrature (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 5. Numerical Integration
> **절**: §5.5 Gaussian Quadrature — *노드 자체를 자유 변수로 두면, $n$ 점으로 $2n - 1$ 차 다항식이 정확*

## 오늘의 큰 그림
Day 18 사다리꼴은 $\mathcal O(h^2)$, Day 19 Simpson 은 $\mathcal O(h^4)$, Day 20 Romberg 는 *기하급수*. 셋 모두 노드를
*균등하게* 박았다 — 자유도는 가중치 $n$ 개. 오늘은 *노드 자체* 도 미지수로 풀어, 자유도 $2n$ 으로 *$2n - 1$ 차* 다항식을
정확히 적분한다. 부드러우면 *지수 수렴*, 깨지면 *Romberg 와 같은 진단 신호 (모든 비율이 같은 멱승)*.

세 문제를 관통하는 한 식은

$$
\boxed{\;\int_{-1}^{1} f(x)\, dx \;=\; \sum_{k=1}^{n} w_k\, f(x_k)\;\;\text{exact for}\;\; f \in \mathbb{P}_{2n-1}\;},
\qquad x_k = \text{root of } P_n(x),\quad w_k = \frac{2}{(1-x_k^2)[P_n'(x_k)]^2}.
$$

— 노드 = Legendre 영점, 가중치 = 폐형식. 임의 구간 $[a, b]$ 는 아핀 변환.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **자유도 / 차수** | $n$ 노드 + $n$ 가중치 = $2n$ → 차수 $2n - 1$ | 직교다항식 (Legendre) 의 영점 |
| **부드러운 함수** | $|E_n| \lesssim M / \rho^{2n}$, $\rho$ = Bernstein 타원 반경 | 복소 평면의 최근접 특이점까지 거리 |
| **끝점 특이점** | $|E_n| \sim n^{-(2\alpha + 2)}$ — *대수* | 가중함수가 미스매치 → Gauss-Jacobi 또는 변수 변환으로 복구 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_5_5_01.ipynb`](CE_5_5_01.ipynb) | Gauss-Legendre 노드/가중치를 *Newton-on-Chebyshev-seed* 로 직접 구현. NumPy 황금-표준과 머신 정밀도까지 일치. **차수 정확성**: $n = 2, \ldots, 6$ 마다 $x^k$ 적분 오차를 $k = 0, \ldots, 14$ 로 표 — $k \leq 2n - 1$ 까지는 $\sim 10^{-16}$, $k = 2n$ 에서 *유한 점프*. 세 부드러운 적분 ($e^x$, $\sin$, $1/(1 + x^2)$) 에 $n = 2, \ldots, 8$ 적용 — $n = 6$ 부터 머신 epsilon. 같은 $N_f$ 의 합성 Simpson 과 비교: Simpson 은 직선 ($-4$), Gauss 는 *위로 휘는 곡선*. |
| 2 | [`CE_5_5_02.ipynb`](CE_5_5_02.ipynb) | Trapezoid · Simpson · Romberg 대각 · Gauss 네 방법의 *비용 vs 정확도* 정면 비교 (loglog). 같은 정확도 ($|E| < 10^{-k}$) 를 위한 최소 $N_f$ 표 — Gauss 가 *한 자리수* 면 $10^{-12}$ 도달, Trapezoid 는 *수천 번* 필요. **break-even 표**: Romberg 가 Simpson 을 추월하는 $N_f \sim 5$–$9$, Gauss 가 Romberg 를 추월하는 $N_f \sim 5$–$10$. $4/(1 + x^2)$ 는 복소 극 ($\pm i$) 때문에 Gauss 의 지수 곡선 *기울기가 살짝 완만*. |
| 3 | [`CE_5_5_03.ipynb`](CE_5_5_03.ipynb) | Gauss-Legendre 의 *실패 모드* — $\sqrt x$ ($\alpha = 1/2$), $1/\sqrt x$ ($\alpha = -1/2$), $1/\sqrt{1 - x^2}$. 경험적 기울기 $-2.96, -0.98, -0.98$ vs 이론 $-3, -1, -1$ — *완벽 일치*. **두 복구 경로**: (a) **Gauss-Chebyshev** 의 가중함수 흡수 → $\int 1/\sqrt{1 - x^2} = \pi$ 가 *모든 $n$* 에서 정확; (b) **변수 변환** $x = u^2$ → $\sqrt x \to 2u^2$ (다항식), $1/\sqrt x \to 2$ (상수) → Gauss-Legendre 가 *$n = 2$* 부터 머신 정밀도. *알고리즘이 망가진* 게 아니라 *함수가 가정을 어긴* 것이라는 정확한 진단. |

## 한 줄 정리
> **Gauss = 노드를 자유 변수로 두는 Newton-Cotes 의 한 단계 도약.**
> 부드러우면 $n$ 함수 평가로 $2n - 1$ 차 정확 → *지수 수렴*. 깨지면 *대수* 로 떨어지지만,
> *가중함수 맞춤 (Gauss-Jacobi)* 또는 *변수 변환* 으로 정확히 복구. *Day 20 Romberg 의 실패-회복 구조와 평행*.

## 사용 라이브러리
- NumPy (Bonnet 점화로 $P_n$ 평가, Newton-on-Chebyshev-seed 로 영점 추출)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`semilogy`, `loglog`, 영문 라벨 — 한글 폰트 의존 회피)
- 외부 데이터 없음 — 모든 함수와 참값을 코드 안에서 정의

## 다음 (Day 22)
**§5.6 — Adaptive Quadrature**. 지금까지 본 모든 방법은 *전역적* 으로 노드 수를 정함.
적응형은 *어디가 더 어려운지* 데이터에서 학습해, 그곳에 *재귀적* 으로 노드를 추가한다 — 같은 $N_f$
로 *불규칙한* 함수에서 극적 개선. *변수 변환* 이 부드러움을 *직접* 만들어 준다면, 적응형은 *분할* 을
*간접* 으로 배운다 — 둘은 보완 관계.

> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
> 사용자 커리큘럼 보강이 필요하면 `_meta/curriculum.md` 에 Day 14+ 항목을 추가하면 됨.
