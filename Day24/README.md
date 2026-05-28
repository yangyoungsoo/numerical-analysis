# Day 24 — §6.2 Natural Cubic Splines (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 6. Spline Functions
> **절**: §6.2 Natural Cubic Splines — *$C^2$ 부드러움 + tridiagonal 시스템*

## 오늘의 큰 그림
Day 23 (§6.1) 에서 *조각마다 낮은 차수 + 노드에서 부드러움 조건* 의 spline 으로 가는 길을 열었다.
1차 spline 은 $C^0 + \mathcal O(h^2)$, 2차 spline 은 $C^1 + \mathcal O(h^3)$ 까지 갔지만 2차에서는
*$z_i$ 의 부호-교대 잔존* 이라는 한계가 남았다. 챕터 6 의 §6.2 는 그 잔존을 *tridiagonal 시스템* 으로
*잠가서* — $S \in C^2$ 와 $\mathcal O(h^4)$ 수렴을 동시에 얻는다.

세 문제를 관통하는 한 식:

$$
\boxed{\;h_{i-1}\, z_{i-1} + 2(h_{i-1} + h_i)\, z_i + h_i\, z_{i+1}
       \;=\; 6\!\left(\tfrac{y_{i+1} - y_i}{h_i} - \tfrac{y_i - y_{i-1}}{h_{i-1}}\right)\;}
\qquad i = 1, \dots, n-1
$$

— 대각 우세한 SPD tridiagonal. Thomas (Day 10) 로 $\mathcal O(n)$ 에 푼다. *경계조건* 의 두 자유도
$z_0, z_n$ (혹은 $S'(a), S'(b)$, 혹은 *not-a-knot*) 의 선택이 *끝점에서의 차수* 를 결정한다.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **자연 BC 일치** | $\|f - S\|_\infty \le 5 h^4 M_4 / 384$, $C^2$ 부드러움 | $f^{(4)}$ 와 분할 $h$ |
| **노드 분포** | Runge 에서 spline 은 수렴, Lagrange 는 발산 | spline 의 *지역성* + 노드 *분포 상수* |
| **경계조건** | 끝 5% 영역: natural 위험, clamped / not-a-knot 안전 | 두 자유도의 *잠금 방식* |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_6_2_01.ipynb`](CE_6_2_01.ipynb) | $f(x) = \sin(\pi x / 4)$ 에서 자연 3차 spline 의 *$\mathcal O(h^4)$ 수렴* 직접 측정. $f''(0) = f''(4) = 0$ 이라 자연 BC 가 *정확히* 맞아떨어져, 인접 비율 $E_n / E_{2n} \to 16$ 가 표에서 깔끔하게 ($16.89 \to 16.23 \to 16.06 \to 16.01 \to 16.02$). 경험 기울기 $+4.02$, 이론 $+4$. Day 23 의 1차 spline 을 *같은 노드/같은 함수* 에 다시 적용해 정면 비교 — loglog 평면에서 *기울기 두 배 차이* 가 시각적으로 분명. $n = 8$ 케이스에서 cubic spline 은 그림에서 $f$ 와 거의 구분되지 않고, linear 는 노드 사이에서 포물선 모양 오차 자취를 그대로 남긴다. |
| 2 | [`CE_6_2_02.ipynb`](CE_6_2_02.ipynb) | Runge 함수 $1/(1 + 25 x^2)$ 에서 (a) 균등 노드 vs (b) Chebyshev 노드 × {자연 cubic spline, 전역 Lagrange} 의 *네 가지* 조합 정면 비교. *Lagrange 균등* 은 $n$ 이 커질수록 *지수 발산* (경험 기울기 $-2.13$). *Spline 균등* 은 $\mathcal O(h^{3.4})$ 로 안전하게 수렴 — Runge peak 에서 $f^{(4)}$ 가 폭주해 이론 $+4$ 보다 조금 무뎌짐. Chebyshev 노드로 바꾸면 Lagrange 가 $+2.25$ 로 수렴하고, spline 도 *상수* 가 줄어 정확도 추가 이득. $n = 16$ 균등 Lagrange 의 *대표적 Runge 진동* ($|y| \gg 1$) 과 같은 노드 spline 의 *부드러움* 을 한 그림에 나란히. |
| 3 | [`CE_6_2_03.ipynb`](CE_6_2_03.ipynb) | $f(x) = \sin x$ on $[0, \pi]$ 에서 *natural · clamped · not-a-knot* 세 경계조건을 모두 직접 구현. 전역 max 오차는 셋 다 $\mathcal O(h^4)$ ($+4.02, +4.03, +4.61$) — sin 은 운 좋게 $f''(0) = f''(\pi) = 0$ 이라 natural 도 살아남는다. 만약 $f''$ 가 끝에서 0 이 아니라면 (앞 문제 1 의 초기 시도에서 봤듯이) natural 의 끝점 차수가 $\mathcal O(h^2)$ 로 떨어진다. SciPy `CubicSpline` 의 기본 BC 가 *not-a-knot* 인 이유 — $f'$ 정보 없이도 전역 $h^4$ 을 안전히 유지한다. |

## 한 줄 정리
> **자연 3차 spline = $C^2$ + tridiagonal + $\mathcal O(h^4)$ — *경계조건* 이 그 차수를 끝까지 지켜준다.**
> Day 23 의 *2차 spline 의 부호 교대* 가 *3차의 SPD 시스템* 으로 잠기면서, *어떤 노드 분포에서도*
> 발산하지 않는 표현이 된다. 다음 단계는 §6.3 — 같은 $C^2$ 부드러움을 *기저의 국소성* 으로 다시 표현하는 B-spline.

## 사용 라이브러리
- NumPy (`searchsorted`, `polyfit/polyval`, `linspace`)
- SciPy (`scipy.linalg.solve_banded` 로 tridiagonal $\mathcal O(n)$ 해결, `scipy.linalg.solve` 로 clamped / not-a-knot 의 변형 시스템)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`loglog` 수렴 그림, 영문 라벨 — 한글 폰트 의존 회피)
- 외부 데이터 없음 — 모든 함수 / 노드 / 평가 격자는 코드 내에서 생성

## 다음 (Day 25)
**§6.3 — B-Splines: Interpolation and Approximation.** 같은 $C^2$ 부드러움을 *기저의 국소성*
으로 다시 표현 — Cox-de Boor 점화식, *국소 지지 (local support)* 의 한 데이터 변경이 *그 점 주변* 몇
구간만 영향을 미치는 성질, 그리고 *convex hull* / *variation-diminishing* 같은 기하학적 성질을 직접
측정한다. 오늘의 *tridiagonal* 에서 B-spline 의 *띠 행렬 보간* 으로 자연스럽게 이어진다.

> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
> 사용자 커리큘럼 보강이 필요하면 `_meta/curriculum.md` 에 Day 14+ 항목을 추가하면 됨.
