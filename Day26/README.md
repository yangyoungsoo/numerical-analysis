# Day 26 — §6.4 Interpolation and Approximation by B-Splines (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 6. Spline Functions
> **절**: §6.4 Interpolation and Approximation by B-Splines — *Non-uniform & Clamped Knots, LSQ, Bernstein 극한*

## 오늘의 큰 그림
Day 25 (§6.3) 에서 *B-spline 의 국소 지지·비음·partition of unity* 라는 세 가지 핵심 성질을 Cox–de Boor
점화식으로 직접 확인했다. 그 도구가 *균등 단순 매듭* 의 한 풍경이었다면, §6.4 는 그 도구를 같은 형태로
*비균등 매듭* / *끝매듭 중복* / *과결정 LSQ* / *단일 구간 극한* 으로 일반화하면서 **같은 식 하나로**
보간·근사·기하학 세 영역을 한꺼번에 묶는다.

세 문제를 관통하는 한 식 — 임의의 매듭 벡터에서의 **B-spline 곡선**:

$$
\boxed{\;
P(x) \;=\; \sum_{i=0}^{n} c_i\, B_{i,k}(x),
\qquad
B_{i,k}(x) \text{ from Cox--de Boor on knot vector } T.
\;}
$$

매듭 벡터 $T$ 의 *구성* — 끝매듭 중복도, 내부 매듭의 균등성·중복도, 매듭 수 — 만으로
*경계 행동*, *국소 부드러움*, *모델 복잡도*, *기하학적 성질* 이 한꺼번에 결정된다.

| 관점 | 핵심 결과 | 매듭 벡터에서 결정되는 부분 |
|---|---|---|
| **경계 행동** | clamped end-knots → endpoint interpolation $P(a)=c_0, P(b)=c_n$ | 양 끝 매듭의 다중도 $k+1$ |
| **국소 부드러움** | 내부 매듭 중복도 $r$ 일 때 그 점에서 $C^{k-1-r}$ | 내부 매듭의 다중도 |
| **모델 복잡도** | LSQ 자유도 $n+1 = $ #knots $- k - 1$ | 내부 매듭 개수 |
| **수치 구조** | $A^\top A$ 가 $2k$-banded SPD | *국소 지지* (매듭 구성에서 자동) |
| **기하학적 성질** | variation-diminishing, convex hull, Bezier 극한 | 비음 + PoU |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_6_4_01.ipynb`](CE_6_4_01.ipynb) | Cox–de Boor 를 *비균등 매듭* 과 *clamped end-knots* 로 확장. 양 끝 매듭을 $k+1$ 중복으로 잠그면 $B_{0,k}(t_0)=1, B_{n,k}(t_{\max})=1$ 가 되어 곡선이 첫·마지막 제어점을 *정확히* 지난다 — 수치 검증에서 $|P(t_0) - c_0| = 0$. *Partition of unity* $\sum_i B_{i,k} = 1$ 가 세 매듭 벡터 (단순 균등 / 비균등 / clamped) 모두에서 $\sim \varepsilon_{\text{mach}}$ 수준으로 성립 — PoU 가 매듭 균등성에 의존하지 않는 *구조적* 성질임을 확인. 내부 매듭 다중도 $r = 1, 2, 3$ 의 cubic basis 를 그려, 그 점에서의 부드러움이 $C^2 \to C^1 \to C^0$ 로 *국소적* 으로 떨어지는 흐름을 시각화. |
| 2 | [`CE_6_4_02.ipynb`](CE_6_4_02.ipynb) | $f(x) = \sin(2\pi x) + 0.4\cos(5\pi x) + \mathcal N(0, \sigma^2)$ 의 $N = 200$ 잡음 표본에 cubic B-spline LSQ. 매듭 수 $\{5, 9, 17, 33, 65\}$ 로 *bias-variance trade-off* 곡선을 그리고, 같은 자유도의 *전역 polynomial LSQ* 와 정면 비교 — polynomial 의 test RMS 는 자유도가 늘면 *지수* 적으로 발산 (Runge 의 연속), B-spline 은 *부드럽게* degrade. 정규 행렬 $A^\top A$ 를 heatmap 으로 그려, 비영 entry 가 정확히 $\|i - j\| \le k = 3$ 안에 갇혀 있음을 *밴드 밖 max $\sim 0$* 으로 검증. polynomial 의 $V^\top V$ 는 dense + Hilbert-like, 조건수 $\kappa$ 가 자유도와 함께 $10^0 \to 10^{14+}$ 로 폭주. |
| 3 | [`CE_6_4_03.ipynb`](CE_6_4_03.ipynb) | (a) **Variation-diminishing** 통계 검증: $k = 3$, 6 개 무작위 제어점 1000 회 — *위반 0 회*, 곡선의 부호 변화가 *항상* 제어 다각형 이하. (b) **Bernstein = clamped 단일 구간 B-spline**: $T = (0^{k+1}, 1^{k+1})$ 에서 Cox–de Boor 가 $B_{i,k}(x) = \binom{k}{i} x^i (1-x)^{k-i}$ 로 정확히 환원됨을 수치적으로 ($\sim 10^{-15}$) 확인. (c) **Bezier vs Lagrange** 같은 제어값 $c = (0, 3, -2, 1)$ 에서 — Bezier 는 *끝점 통과 + convex hull 안 + 부호 변화 제어 다각형 이하*, Lagrange 는 모든 점 통과지만 *오버슛 + convex hull 밖 + 더 많은 부호 변화*. 고차 ($k = 8$) 에서는 Lagrange 의 *Runge 진동* 이 control hull 을 크게 벗어남. |

## 한 줄 정리
> **매듭 벡터 = B-spline 의 모든 다이얼.** Day 25 의 *국소 지지·비음·PoU* 세 성질이,
> 매듭 벡터를 *비균등 / 끝매듭 중복 / 내부 중복 / 단일 구간* 으로 바꾸는 것만으로 *endpoint interpolation*,
> *국소 부드러움 조절*, *bias-variance 다이얼*, *띠 정규 방정식*, *Bezier 모양 제어* 의 다섯 가지 도구로
> 확장된다. 같은 식, 다른 매듭, 다른 응용.

## 사용 라이브러리
- NumPy (`searchsorted`, `polyfit/polyval`, `linspace`, `vander`)
- SciPy (`BSpline`, `BSpline.design_matrix`, `make_lsq_spline`, `lagrange`)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (basis viewer, sparsity heatmap, semilog 조건수 비교, 영문 라벨 — 한글 폰트 의존 회피)
- 외부 데이터 없음 — 모든 함수 / 노드 / 평가 격자는 코드 내에서 생성

## 다음 (Day 27)
**§6.5 — Tensor-product B-spline surfaces (and NURBS).** 1D 의 *국소 지지* / *clamped end-knots* /
*variation-diminishing* 이 2D 의 *tensor-product basis* $B_{i,k}(u) B_{j,k}(v)$ 로 그대로 옮겨가고,
*control net* 의 *convex hull* 안에 곡면 전체가 들어간다는 *2D variation-diminishing* 으로 일반화된다.
이후 rational 가중치 (NURBS) 를 도입하면 정확한 원·구·이차곡면이 같은 표현 안에 들어온다.
그 다음은 챕터 7 ODE 로, *spline collocation* 으로 BVP 를 푸는 데까지 같은 도구가 재사용된다.

> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
> 사용자 커리큘럼 보강이 필요하면 `_meta/curriculum.md` 에 Day 14+ 항목을 추가하면 됨.
