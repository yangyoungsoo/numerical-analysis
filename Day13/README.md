# Day 13 — §3.3 Secant Method (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 3. Nonlinear Equations
> **절**: §3.3 Secant Method — *도함수 없이도* 황금비 차수의 초선형 수렴

## 오늘의 큰 그림
Day 11 이분법은 *부호변화* 만으로 안전했지만 *선형* 으로 느렸고,
Day 12 Newton 은 *도함수* 의 도움으로 *이차* 였지만 변곡점이나 fractal 경계에서
*깨졌다*. 오늘은 그 둘 사이의 *황금 지점* 에 있는 **Secant 방법** 을 본다.

도함수를 *유한차분* 으로 근사하기에 *한 스텝당 함수 평가가 1 회* 뿐이고,
그 대신 수렴 차수는 $\phi = (1+\sqrt 5)/2 \approx 1.618$ — 정확히 황금비.
*함수 평가당 효율* 로 보면 Newton 보다 빠른 경우가 흔하다.

세 문제를 관통하는 메시지:

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **차수 (스텝당)** | $p = \phi \approx 1.618$ — $p^2 = p + 1$ 의 양의 해 | $f''(r)/f'(r)$ 와 *두 점* 점화 구조 |
| **효율 (평가당)** | $E_S = \phi^1 = 1.618 > E_N = 2^{1/2} = 1.414$ | evals/step (Secant 1, Newton 2) |
| **강건성** | 순수 secant 는 bracket 을 잃을 수 있다 → hybrid 가 답 | bisection 의 *부호 보존* + secant 의 *속도* |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_3_3_01.ipynb`](CE_3_3_01.ipynb) | $\cos x = x$, $x e^x = 1$, $\sqrt a$ 에 secant 적용. $\log \|e_{n+1}\|$ vs $\log \|e_n\|$ 회귀로 기울기 측정 — **모든 문제에서 $1.62 \pm 0.03$** 으로 이론값 $\phi = 1.618$ 와 일치. 점화식 $e_{n+1} \sim e_n e_{n-1}$ 로부터 $p^2 - p - 1 = 0$ 의 유도까지 정리. |
| 2 | [`CE_3_3_02.ipynb`](CE_3_3_02.ipynb) | Newton 과 Secant 를 *같은 함수 평가 예산* 으로 비교. 가로축을 $N_f$ (누적 평가 수) 로 두면 secant 가 종종 *앞서* 떨어진다. per-eval order 실측치 $E_N \approx 1.41$, $E_S \approx 1.62$ — 이론치 $2^{1/2}$, $\phi$ 와 일치. **$f'$ 가 비쌀 때 secant 의 승리.** |
| 3 | [`CE_3_3_03.ipynb`](CE_3_3_03.ipynb) | **Hybrid bisection + secant** — bracket $[a, b]$ 를 유지하면서 secant 스텝이 *bracket 안* 이고 *충분 진전* 이면 채택, 아니면 bisection 으로 후퇴. $x^{20} - 1$, $e^x - 2$, $\tan x - x$ 에서 테스트. `scipy.optimize.brentq` 와 답 일치. **Brent 의 정신**: secant 가 가능하면 secant, 안 되면 bisection. |

## 한 줄 정리
> **Secant 는 도함수를 *모른 채로도* 황금비 차수 $\phi$ 의 초선형 수렴을 낸다.**
> *스텝* 단위로는 Newton 이 빠르지만 *평가* 단위로는 secant 가 더 효율적이다.
> 그러나 *bracket* 을 보장하지 못하므로, 실무에서는 bisection 과 결합한
> **hybrid** (Brent 류) 가 자연스러운 답.

## 사용 라이브러리
- NumPy (벡터화 산술, `np.log10`, `np.polyfit` 로 기울기 회귀)
- Pandas (`pd.set_option("display.float_format", ...)` 로 표 출력)
- Matplotlib (`semilogy`, 영문 라벨)
- SciPy (`scipy.special.lambertw` 로 $W(1)$ 참값, `scipy.optimize.brentq` 로 hybrid 검증)

## 다음 (Day 14)
**챕터 4 — Interpolation** 으로 넘어간다. §4.1 **다항식 보간 (Polynomial Interpolation)**:
$n+1$ 개의 점 $(x_i, y_i)$ 가 주어졌을 때 그 점들을 모두 지나는 차수 $\le n$ 의 다항식이
*유일하게* 존재한다. 이를 구성하는 두 방법 — **Lagrange 형** 과 **Newton 분할차분 (divided
differences)** 형 — 을 직접 구현하고, 둘이 *수학적으로* 같은 다항식임을 수치 비교로
확인한다.  *Runge 현상* 의 첫 등장 예고.
