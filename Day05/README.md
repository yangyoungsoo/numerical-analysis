# Day 05 — §1.2 Mathematical Preliminaries (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 1. Mathematical Preliminaries and Floating-Point Representation
> **절**: §1.2 Mathematical Preliminaries — Taylor's Theorem, Range Reduction, Mean Value Theorem

## 오늘의 큰 그림
지금까지(Day 01–04) §1.1에서 *floating-point 산술 자체* 의 한계 — 자릿수 손실, 합산 순서, 거의 같은 두 수의 차 — 를 봤다.
오늘부터 §1.2 로 넘어가, 그 한계 위에 **수학적 분석 도구** 가 어떻게 끼어드는지를 다룬다. 핵심 도구는 셋:

1. **Taylor 정리**: 함수 근사를 *얼마나 정확히* 할 수 있는지를 잉여항으로 정량화.
2. **주기성과 범위 축소**: 수학적으로 자명한 등식 $\sin(x) = \sin(x \bmod 2\pi)$ 가 부동소수점에서는 *얼마나 깨지는지*.
3. **Mean Value Theorem (MVT)**: 모든 오차 분석의 등뼈 — *존재만 보장하는 정리* 의 비유일성과 ill-posed 케이스.

세 문제 모두 동일한 패턴: **수학 정리를 코드로 검증 → 부동소수점에서 정리가 깨지는 지점 식별 → 우회 전략 비교.**

| 도구 | 수학적 결과 | 부동소수점에서의 한계 |
|---|---|---|
| Taylor 잉여항 | $|R_n| \le \dfrac{|x|^{n+1}}{(n+1)!}\,M_{n+1}$ | 큰 $|x|$, 수렴 반경 끝, 항 누적 round-off |
| Range reduction | $\sin(x) = \sin(x \bmod 2\pi)$ | $|x| \cdot 2^{-52}/(2\pi)$ 의 입력 정밀도 한계 |
| MVT | $\exists\,\xi: f'(\xi) = $ secant slope | $\xi$의 비유일성 + 거의 선형 함수에서의 ill-posed |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_1_2_01.ipynb`](CE_1_2_01.ipynb) | $e^x,\;\sin x,\;\ln(1+x)$ Taylor 부분합. 같은 함수라도 $|x|$가 멀수록 항 수가 폭증; $\sin(-10)$은 항이 한때 매우 커져 cancellation 위험; $\ln(1.9)$는 잉여항이 조화급수 속도로 줄어 사실상 발산. Lagrange bound 곡선과 실제 오차가 평행. |
| 2 | [`CE_1_2_02.ipynb`](CE_1_2_02.ipynb) | $\sin(10^k)$를 naive / float64 reduce / mpmath reduce 로 비교. naive는 입력 정밀도 한계 곡선 $|x| \cdot 2^{-52}/(2\pi)$ 를 따라 폭주; float64 reduce는 cancellation으로 더 빨리 망가짐; mpmath reduce만이 $k=14$까지도 머신 정밀도 유지. **문제는 알고리즘이 아니라 입력 표현**. |
| 3 | [`CE_1_2_03.ipynb`](CE_1_2_03.ipynb) | MVT 수치 검증: $\sin x,\;e^{-x^2},\;x^3-4x$ 세 함수에 대해 brentq로 $\xi$ 찾기. $x^3-4x$ 케이스는 $\xi$가 두 개 (비유일성). 거의 선형인 $f(x)=x+0.001\sin(50x)$ 에서는 $\xi$ 후보가 무수히 많아 ill-posed. |

## 한 줄 정리
> **수학 정리(Taylor / 주기성 / MVT)는 *항상* 성립하지만, 부동소수점에서 그 정리를 그대로 끌어쓰면 곳곳에서 깨진다.**
> 정리의 *어느 인자* 가 깨지는지(잉여항의 크기? 입력의 정밀도? 해의 비유일성?) 를 알아야 안전한 알고리즘을 짤 수 있다.

## 사용 라이브러리
- NumPy (`np.exp`, `np.sin`, `np.log1p`, `np.cos`)
- SciPy (`scipy.optimize.brentq`)
- mpmath (50자리 임의정밀도 — 참값과 정밀 범위 축소)
- Pandas, Matplotlib

## 다음 (Day 06)
§1.3 *Floating-Point Representation* 으로 진입. machine epsilon 직접 측정, IEEE 754의 subnormal 영역 탐색, 이차방정식 근의 catastrophic cancellation을 다룬다.
