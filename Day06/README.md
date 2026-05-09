# Day 06 — §1.3 Floating-Point Representation (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 1. Mathematical Preliminaries and Floating-Point Representation
> **절**: §1.3 Floating-Point Representation — Machine epsilon, Subnormal numbers, Catastrophic cancellation

## 오늘의 큰 그림
지금까지(Day 01–05)는 *수학적 정리* 가 부동소수점에서 어떻게 깨지는지를 봤다 (Taylor 절단, 합산 순서, MVT의 비유일성).
오늘은 한 단계 더 들어가, 그 깨짐의 *물리적 근원* — IEEE 754 비트 레이아웃 — 을 직접 들여다본다.
세 문제는 모두 같은 골격이다: **표현(Representation) → 한계(Limit) → 한계가 폭발하는 곳(Failure mode)**.

| 문제 | 표현 측면 | 한계 측면 | 폭발 케이스 |
|---|---|---|---|
| 1 | 가수부 52비트 | $\varepsilon_{\text{mach}} = 2^{-52}$ | 큰 $a$에서 $a + \varepsilon$ 흡수 |
| 2 | 지수 $-1022$ 하한 | ULP가 $2^{-1074}$로 평탄 | subnormal 영역의 상대 정밀도 손실 |
| 3 | 64bit 산술 round-off | 거의 같은 두 수의 차 | $b^2 \varepsilon_{\text{mach}}$ 증폭 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_1_3_01.ipynb`](CE_1_3_01.ipynb) | $1 + \varepsilon \ne 1$ 의 최소 $\varepsilon$ 을 이분 탐색으로 측정. `float64`는 $2^{-52}$, `float32`는 $2^{-23}$. 기준점 $a = 2^k$ 별 $\varepsilon(a)$ 가 $|a|$에 *비례* — machine epsilon이 *상대* 단위임을 시각적으로 확인. |
| 2 | [`CE_1_3_02.ipynb`](CE_1_3_02.ipynb) | $2^{-1022}$ 에서 절반씩 내려가며 subnormal 영역 탐사. 가장 작은 양수 $= 2^{-1074}$. ULP는 정상에서 $\propto x$, 비정상에서 평탄. 상대 간격 $\text{ulp}/x$ 는 정상에서 $2^{-52}$ 일정, 비정상에서 1까지 폭증 — *gradual underflow*. |
| 3 | [`CE_1_3_03.ipynb`](CE_1_3_03.ipynb) | $x^2 + bx + 1 = 0$ 의 small root을 *naive* vs *Vieta-based stable* 로 비교. naive 의 상대 오차는 정확히 $b^2 \varepsilon_{\text{mach}}$ 직선 위. $b = 10^8$ 에서 유효 자릿수 0; stable 은 머신 정밀도 유지. |

## 한 줄 정리
> **부동소수점은 *표현* 의 한계와 *연산* 의 한계가 동시에 존재한다.** 표현 한계는 ULP/subnormal로 정량화되고,
> 연산 한계는 그 표현 한계 위에서 *cancellation* 을 통해 폭발한다. 안정 알고리즘은 *공식의 변형* 으로 이를 회피한다.

## 사용 라이브러리
- NumPy (`np.finfo`, `np.nextafter`, `np.float32` 캐스팅)
- mpmath (50자리 임의정밀도 — 이차근의 참값)
- Pandas, Matplotlib

## 다음 (Day 07)
§1.4 *Loss of Significance* 로 진입. 오늘의 quadratic 케이스를 확장해 $\log(1+x) - x$, $1 - \cos x$ 같은
빈번한 cancellation 패턴과 `expm1` / `log1p` 등 표준 라이브러리 함수의 효용을 비교한다.
