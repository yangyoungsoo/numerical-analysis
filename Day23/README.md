# Day 23 — §6.1 First-Degree and Second-Degree Splines (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 6. Spline Functions
> **절**: §6.1 First-Degree and Second-Degree Splines — *조각마다 낮은 차수, 노드에서 부드러움 조건*

## 오늘의 큰 그림
Day 14 에서 전역 Lagrange 의 *유일성* 과 *Runge 발산* 을 동시에 봤고, Day 22 에서는 적응형 적분이
*함수가 어려운 곳에 분할이 자동으로 모이는* 그림을 보여줬다. 챕터 6 은 그 두 갈래를 *함수 표현 자체* 로
끌어올린다 — *조각마다 낮은 차수의 다항식*, *노드에서 부드러움 조건* 의 spline.

세 문제를 관통하는 한 식:

$$
\boxed{\;\bigl\|f - S_k\bigr\|_\infty \;\le\; C_k\,h^{k+1}\,\max_x \bigl|f^{(k+1)}(x)\bigr| \quad (k = 1, 2)\;}
\qquad
\text{지역적 · 안정적 · 발산하지 않음}
$$

— 전역 다항식의 차수 $n$ 이 아니라 *분할 크기 $h$* 가 정확도를 결정한다. 그래서 *어떤 함수에서도* 발산하지
않고, *어디에 노드를 두느냐* 가 다음 자유도가 된다.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **1차 spline $S_1$** | $\|f - S_1\|_\infty \le h^2 M_2 / 8$, $C^0$ 부드러움 | $f''$ 와 분할 $h$ |
| **2차 spline $S_2$** | $z_{i+1} = -z_i + 2(y_{i+1} - y_i)/h_i$, $C^1$ 부드러움 | 한 자유도 $z_0$ 의 *부호-교대* 전파 |
| **knot 배치** | $\rho(x) \propto \sqrt{|f''(x)|}$ 의 *등배분* | 지역 오차 균형 (적응형 적분과 동일 원리) |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_6_1_01.ipynb`](CE_6_1_01.ipynb) | $f(x) = \cos(\pi x/2)$ 에서 1차 spline 의 *$\mathcal O(h^2)$ 수렴* 직접 측정 ($n$ 을 두 배로 → 오차 $\sim 1/4$, 인접 비율 → 4). 이론 상한 $h^2 M_2 / 8$ 가 *느슨한 상한* 으로 잘 작동. 전역 Lagrange 와 같은 노드에서 정면 비교 — 매끈한 함수에서는 둘 다 수렴하지만 spline 은 *지역적 · 상수-시간 평가*. $n = 8$ 케이스에서 spline 오차가 각 구간에서 *포물선 $\tfrac12 (x - t_{i-1})(x - t_i) f''(\xi)$* 의 자취 그대로 나타나는 것을 시각화. |
| 2 | [`CE_6_1_02.ipynb`](CE_6_1_02.ipynb) | 2차 spline 의 점화식 $z_{i+1} = -z_i + 2(y_{i+1} - y_i)/h$ 를 직접 구현. *세 가지 $z_0$* ($\cos 0 = 1$ 정직, $0$, $3$ 왜곡) 를 비교 — 오차가 정확히 $z_i = z_i^\star + (-1)^i \delta$ 의 *부호 교대* 로 전파됨을 노드값으로 확인 (코드 출력 $+2, -2, +2, -2, \ldots$). 진폭은 증폭되지 않지만 사라지지도 않아 *전구간 톱니 진동* 으로 드러남. 정직한 $z_0$ 에서 경험 수렴 기울기 $\approx +3$ — 1차의 $+2$ 보다 한 차수 이득. |
| 3 | [`CE_6_1_03.ipynb`](CE_6_1_03.ipynb) | *Runge 함수* $1/(1 + 25 x^2)$ 에서 spline 의 *수렴* 과 Lagrange 의 *발산* 을 *같은 균등 노드* 위에서 정면 대비 — loglog 기울기가 spline $+2$, Lagrange *음수* (발산). 그 다음 *$\rho(x) \propto \sqrt{|f''(x)|}$* 의 등배분으로 *적응 노드* 를 만들어, 같은 $n$ 에서 균등 대비 *수배 ~ 십수배* 의 오차 감소를 측정. 적응 노드가 원점 (peak) 주변에 *몰리는* 분포 시각화 — Day 22 적응형 적분의 *어려운 곳에 분할이 모이는* 그림과 정확히 평행. |

## 한 줄 정리
> **Spline = 조각마다 낮은 차수 + 노드에서 부드러움 조건 = *어떤 함수에서도 발산하지 않는* 표현.**
> 1차에서 *$h^2$ 수렴 + $C^0$*, 2차에서 *$h^3$ + $C^1$* — 차수를 올릴 때마다 *경계/자유도* 의 잠금이
> 한 차원씩 추가되며, 이 흐름은 §6.2 cubic spline 의 *tridiagonal 시스템* 으로 자연스럽게 이어진다.

## 사용 라이브러리
- NumPy (`searchsorted` 로 구간 인덱싱, Vandermonde 다항식 `polyfit/polyval`, $z_i$ 점화 직접 구현)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`loglog` 수렴 그림, 노드 분포 vline, 영문 라벨 — 한글 폰트 의존 회피)
- 외부 데이터 없음 — 모든 함수 / 노드 / 평가 격자는 코드 내에서 생성

## 다음 (Day 24)
**§6.2 — Cubic Splines.** 2차 spline 의 *부호-교대 진폭 잔존* 을 *tridiagonal 연립방정식* 으로 잠그면
3차 cubic spline 이 된다. $C^2$ 부드러움 + 자유도 두 개의 *경계 조건* (natural, clamped, not-a-knot) 의
선택이 어떻게 정확도와 끝점 거동을 바꾸는지, $\mathcal O(h^4)$ 의 수렴을 직접 측정한다.
오늘 본 *등배분 knot 배치* 의 사고는 cubic spline 에서 한층 더 강력해진다 ($f^{(4)}$ 가 큰 곳에 모음).

> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
> 사용자 커리큘럼 보강이 필요하면 `_meta/curriculum.md` 에 Day 14+ 항목을 추가하면 됨.
