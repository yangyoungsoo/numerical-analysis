# Day 25 — §6.3 B-Splines: Interpolation and Approximation (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 6. Spline Functions
> **절**: §6.3 B-Splines: Interpolation and Approximation — *국소 기저로 같은 부드러움을 다시 표현*

## 오늘의 큰 그림
Day 23 (§6.1) 에서 *조각마다 낮은 차수 + 노드 부드러움 조건* 의 spline 으로 가는 길을 열고, Day 24 (§6.2) 에서
자연 cubic spline 의 *tridiagonal 시스템* 과 *$\mathcal O(h^4)$ 수렴* 을 확인했다. 챕터 6 의 §6.3 은 그
같은 함수 공간을 *국소적 기저* — **B-spline** — 으로 다시 표현한다. 결과는 같은 곡선이지만, 표현이 *국소적*
이라는 사실 자체가 *시스템 폭의 좁음*, *데이터 노이즈에 대한 안정성*, *모양 보존* 의 세 가지 실용적 위력으로
이어진다.

세 문제를 관통하는 한 식 — **Cox-de Boor 점화식**:

$$
\boxed{\;
B_{i,k}(x)
\;=\;
\frac{x - t_i}{t_{i+k} - t_i}\, B_{i,k-1}(x)
\;+\;
\frac{t_{i+k+1} - x}{t_{i+k+1} - t_{i+1}}\, B_{i+1,k-1}(x).
\;}
$$

— 0차 step 에서 시작해 한 차원씩 *부드럽게 들어 올리는* 재귀. 그 결과인 $B_{i,k}$ 가 갖는 *partition of unity*,
*비음*, *국소 지지* 가 §6.3 의 모든 후속 결과의 출발점이다.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **기저의 성질** | $\sum_i B_{i,k} = 1$, $B_{i,k} \ge 0$, $\operatorname{supp} = k+1$ 구간 | Cox-de Boor 의 형태 |
| **데이터 안정성** | $\Lambda_j = \mathcal O(1)$ (B-spline) vs 지수 폭주 (Lagrange) | *지역 지지* 의 직접 효과 |
| **보간 시스템** | 띠 폭 3 (cubic), $\mathcal O(h^4)$, variation-diminishing | $B_{i,3}$ 의 매듭값 $(0, \tfrac16, \tfrac46, \tfrac16, 0)$ |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_6_3_01.ipynb`](CE_6_3_01.ipynb) | Cox-de Boor 점화식을 *0차 step* 에서부터 재귀로 직접 구현. 균등 단순 매듭 $T = \{0, 1, \dots, 10\}$ 위에서 차수 $k = 0, 1, 2, 3$ 의 모든 기저 함수를 한 그림에 그려, *지지 폭이 한 칸씩 늘고* 그 안에서 *한 번 더 미분 가능* 해지는 흐름을 시각화. **Partition of unity** 검증: 내부 범위 $[t_k, t_{m-k}]$ 에서 $\max\|\sum_i B_{i,k} - 1\| \le \varepsilon_{\text{mach}}$. **Local support** 검증: $B_{3,3}$ 이 $[3, 7]$ 외부에서 $\le \varepsilon_{\text{mach}}$, 내부에서 매끈한 종. 임의 점 $x = 4.7$ 에서 0 이 아닌 cubic B-spline 의 개수가 정확히 $k + 1 = 4$ 임을 직접 확인. |
| 2 | [`CE_6_3_02.ipynb`](CE_6_3_02.ipynb) | $n + 1 = 11$ 점 데이터에서 가운데 점을 $\delta = 1$ 만큼 흔들고, cubic B-spline 곡선과 전역 Lagrange 다항식 곡선의 *섭동 - 원본* 차이를 정면 비교. 차이 곡선 = *영향 함수* $\delta \cdot L_j(x)$ — B-spline 은 $x_j$ 주변 *좁은 종*, Lagrange 는 *전구간 진동*. 정량 지표: B-spline 의 $\Lambda_j \approx 1$, Lagrange 의 $\Lambda_j$ 는 *수 배 ~ 수십 배*. *$\|\Delta\| > 5\% \delta$ 영역의 길이* 로 본 국소성도 *$4h$ vs 전 구간* 으로 격차. $n + 1 = 6 \to 20$ 으로 데이터를 늘릴수록 Lagrange 의 $\Lambda_j$ 가 *지수적* 으로 증가하는 반면 B-spline 은 *1 근방* 에 머무는 것을 semilog 로 시각화. |
| 3 | [`CE_6_3_03.ipynb`](CE_6_3_03.ipynb) | (a) 균등 매듭의 매듭값 $(0, \tfrac16, \tfrac46, \tfrac16, 0)$ 으로 *3중대각* 보간 시스템을 직접 도출, SciPy `make_interp_spline(k=3)` (내부 banded LU) 로 풀어 $f(x) = \sin(\pi x / 2)$ 에서 인접 비율 $\to 16$, 경험 기울기 $+4$ 의 $\mathcal O(h^4)$ 를 측정. Day 24 자연 cubic spline 결과와 같은 그림에 겹쳐 두 표현의 *등가성* 확인. (b) 의도적으로 진동하는 step-like 데이터 ($y_i = (-1)^i$ 잡음 + 계단) 에서 cubic B-spline 의 *내부 극값 수* 와 같은 자유도 (deg $n$) Lagrange 다항식의 *내부 극값 수* 비교 — B-spline 은 *제어 다항형 이하*, 다항식은 *과진동* + *데이터 범위 밖 오버슛*. |

## 한 줄 정리
> **B-spline = 같은 $C^{k-1}$ 부드러움의 *국소 기저*** — 그 결과 *보간 시스템의 띠 폭이 좁고*, *데이터 노이즈에
> 안정적이며*, *데이터보다 더 많이 진동하지 않는다*. 자연 cubic spline (Day 24) 과 같은 함수를 *다른 기저*
> 로 표현한 것이며, 두 표현은 BC 가 같을 때 *정확히 같은 곡선*.

## 사용 라이브러리
- NumPy (`searchsorted` 로 구간 인덱싱, Cox-de Boor 재귀 직접 구현, `polyfit / polyval`)
- SciPy (`make_interp_spline(k=3)` 으로 banded LU 내부 사용, `CubicSpline` 으로 not-a-knot/natural BC 비교)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (subplot 4-panel basis viewer, semilog 민감도 그림, 영문 라벨 — 한글 폰트 의존 회피)
- 외부 데이터 없음 — 모든 함수 / 노드 / 평가 격자는 코드 내에서 생성

## 다음 (Day 26)
**§6.4 — Interpolation and Approximation by B-Splines (continued).** 균등 매듭에서 더 나아가
*non-uniform knot vector* 와 *clamped end-knot* 의 정확한 구성, 그리고 *최소제곱 B-spline 근사*
(보간이 아니라 *과결정* 시스템의 띠 정규 방정식) 로 자연스럽게 이어진다. *variation-diminishing* 은 거기서
*Bernstein 다항식* (특수 매듭 케이스) 으로의 다리가 된다.

> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
> 사용자 커리큘럼 보강이 필요하면 `_meta/curriculum.md` 에 Day 14+ 항목을 추가하면 됨.
