# Day 22 — §5.6 Adaptive Quadrature (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 5. Numerical Integration
> **절**: §5.6 Adaptive Quadrature — *데이터에서 배운 분할로 비매끈성을 정복하는 마지막 한 수*

## 오늘의 큰 그림
Day 18 사다리꼴 $\mathcal O(h^2)$, Day 19 Simpson $\mathcal O(h^4)$, Day 20 Romberg *기하급수*, Day 21 Gauss-Legendre
*$n$ 함수 평가로 $2n-1$ 차*. 셋은 *전역적* 으로 노드 수를 정한다 — 함수가 어디서 어려운지 *알고리즘이 모른다*.
오늘 적응형은 *지역 오차 추정* 한 줄로 그것을 *함수가 직접 가르치게* 한다 — 그리고 *재귀적* 으로 어려운 곳에만
분할을 추가한다.

세 문제를 관통하는 핵심 식은

$$
\boxed{\; |E_{\text{panel}}| \;\approx\; \tfrac{1}{15}\,\bigl|S^{(2)} - S^{(1)}\bigr|, \qquad
        \tau_{\text{child}} \;=\; \tfrac{1}{2}\,\tau_{\text{parent}} \;}
\qquad
\text{(Simpson Richardson + 비례 예산)}
$$

— *Day 20 Romberg* 의 한 칸을 *지역* 차원에 옮긴 것이 적응형의 골격. 오늘은 그 골격 위에서 *panel rule* 을
세 단계로 격상한다: Simpson → Gauss-Kronrod → 2D 텐서곱.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **지역 오차 추정** | $\widehat E = \|S^{(2)} - S^{(1)}\|/15$ (Simpson), $\widehat E = (200\|G - K\|)^{3/2}$ (GK) | Richardson 외삽, 차수 격차 |
| **함수 평가 재사용** | 부모-자식 노드 공유 → 1D 에서 *3 배 절약*, 2D 에서 *부분* 공유 | dyadic 트리 / 노드 위치의 적합성 |
| **MC 와의 break-even** | 부드러우면 cubature 압도, *비매끈 + 다차원* 에서 MC 의 $N^{-1/2}$ 가 살아남음 | 차수 vs 차원의 상대 우위 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_5_6_01.ipynb`](CE_5_6_01.ipynb) | *함수 평가 재사용*·*오차 예산 전파*·*재귀 깊이 추적* 의 세 디테일을 살린 적응형 Simpson. 세 적분 ($e^{-x^2}$, Runge 형 peak, $\sqrt x$) 에서 동일 $N_f$ 의 균등 Simpson 대비 *수십 ~ 수천 배* 가속을 측정. **leaf 분포 시각화** — Runge 는 peak 주변에 *기하급수* 누적, $\sqrt x$ 는 끝점에 *지수* 집중. 나이브 구현 대비 *3 배* 함수 평가 절약. |
| 2 | [`CE_5_6_02.ipynb`](CE_5_6_02.ipynb) | panel rule 을 Simpson 에서 **Gauss-Kronrod (7, 15)** 로 격상. QUADPACK 의 $\widehat E = (200\|G - K\|)^{3/2}$ 가 차수 13 vs 23 의 격차를 *경험적* 으로 이용. 동일 정확도 도달 $N_f$ 가 Simpson 대비 *2 – 10 배* 적음 (정밀도가 높을수록 격차가 커진다). panel 수는 *1/5 ~ 1/10* — `scipy.integrate.quad` 가 GK 기반인 이유. |
| 3 | [`CE_5_6_03.ipynb`](CE_5_6_03.ipynb) | 적응형 골격을 **2D 텐서곱 Simpson** 으로 확장 — 직사각형을 *사분면* 으로 재귀 분할. 세 2D 적분 ($e^{-(x^2+y^2)}$ 매끈, 가운데 peak, 코너 near-singular) 에서 *plain Monte Carlo* 와 정면 비교. 매끈에서는 cubature 가 *모든 $N_f$* 에서 압도, 코너 특이에서는 cubature 차수가 *대수* 로 무너져 MC 와 교차 — **break-even 점** 의 결정점. leaf 직사각형 시각화로 *2D 에서도 어려운 곳에 분할이 정확히 모이는* 구조 확인. |

## 한 줄 정리
> **적응형 = 지역 오차 추정 (한 줄) + 재귀 분할 + 함수 평가 재사용 (데이터 구조).**
> 같은 골격 위에서 panel rule 을 격상하면 정확도가 단계별로 도약하고, 차원이 늘면 *MC 의 차원 무관* $N^{-1/2}$
> 가 우위를 가지기 시작한다 — *cubature 와 MC 는 보완적*, 그 사이를 메우는 것이 quasi-MC.

## 사용 라이브러리
- NumPy (Patterson 의 G7/K15 노드와 가중치 직접 코딩, 텐서곱 가중 행렬, 적응형 재귀 트리)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`loglog`, leaf 사각형/막대 시각화 — 영문 라벨, 한글 폰트 의존 회피)
- 외부 데이터 없음 — 모든 함수와 참값을 코드 안에서 정의 (J2, J3 의 참값은 dense 4001×4001 Simpson 으로 계산)

## 다음 (Day 23)
**Ch 6 §6.1 — Splines and Piecewise Polynomial Approximation**. 적응형 cubature 의 *패널* 과 적응형 Simpson 의
*분할* 이 보여준 *지역성* 의 사고를, 이번엔 *함수 표현 자체* 로 끌어올린다. spline 은 *조각마다 낮은 차수*
의 다항식을 *부드러움 조건* 으로 이어 붙여, *전역 Lagrange* (Day 14 Runge 현상) 의 한계와 *균등 노드* 의 부담을
동시에 푼다. 오늘 본 *어디가 어려운지를 함수가 가르쳐주는 분할* 은 spline 의 *knot 배치 (adaptive knots)* 와
직접적으로 닮아 있다.

> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
> 사용자 커리큘럼 보강이 필요하면 `_meta/curriculum.md` 에 Day 14+ 항목을 추가하면 됨.
