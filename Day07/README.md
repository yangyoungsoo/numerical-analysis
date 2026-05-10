# Day 07 — §1.4 Loss of Significance (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 1. Mathematical Preliminaries and Floating-Point Representation
> **절**: §1.4 Loss of Significance — Sign-selection rule, Reformulation, Library functions

## 오늘의 큰 그림
Day 06 의 마지막 문제(이차방정식 안정 풀이)에서 본 *catastrophic cancellation* 의 발견을,
오늘은 **세 단계의 일반화** 로 확장한다.

1. **부호 선택 규칙 (sign-selection)** — quadratic 의 처방을 *어떤 $b$ 부호든 자동* 으로 동작하는 한 줄 공식으로 일반화.
2. **Taylor 재작성 (reformulation)** — 다항이 아닌 함수 $\log(1+x) - x$ 에서, *함수의 지배항* 을 직접 표현해 cancellation 을 *제거*.
3. **표준 라이브러리 활용 (`expm1`, `log1p`)** — IEEE 754 가 권고하는 보조 함수가 위 두 처방을 *내부적으로* 수행함을 확인하고, 다섯 가지 빈출 패턴에서 그 효용을 정량화.

세 문제를 관통하는 공통 사고는 동일하다:
**위험한 *연산* 을 식별 → 같은 식의 *다른 표현* 으로 경로 변경 → 수학적 항등식이 정밀도를 보존.**

| 문제 | 처방 | 핵심 항등식 | 무엇이 사라지는가 |
|---|---|---|---|
| 1 | 부호 선택 + Vieta 복원 | $x_1 x_2 = c/a$ | quadratic small root 의 $b^2 \varepsilon_{\text{mach}}$ 증폭 |
| 2 | Taylor 직접 합산 | $\log(1+x) = x - \tfrac{x^2}{2} + \tfrac{x^3}{3} - \cdots$ | 빼기 자체 (지배항 직접 평가) |
| 3 | `expm1` / `log1p` (correctly rounded) | $e^x - 1$, $\log(1+x)$ as primitives | 식의 일부에 *항등원* 이 끼어 있는 모든 케이스 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_1_4_01.ipynb`](CE_1_4_01.ipynb) | $ax^2+bx+c=0$ 의 **부호 선택 공식** $q = -\tfrac{1}{2}(b + \operatorname{sign}(b)\sqrt{D})$ + Vieta. $b > 0$ / $b < 0$ / hard case $(10^{-12}, -10^{12}, 1)$ 모두에서 머신 정밀도 유지. naive 곡선이 이론선 $b^2 \varepsilon_{\text{mach}}$ 위에 정확히 올라앉음. |
| 2 | [`CE_1_4_02.ipynb`](CE_1_4_02.ipynb) | $\log(1+x) - x$ 의 **자동 절단 Taylor 합산**. naive 의 상대 오차 $\propto 1/\|x\|$ (이론선 $4\varepsilon_{\text{mach}}/\|x\|$). Taylor는 모든 $\|x\| \in [10^{-15},\,10^{-1}]$ 에서 평탄. 작을수록 항이 *적게* 든다 ($x=10^{-15}$ 면 1 항). 단순 변수 치환 $u=\log 1\!\mathrm{p}(x)$ 만으로는 cancellation 을 *옮길 뿐* 제거하지 못함. |
| 3 | [`CE_1_4_03.ipynb`](CE_1_4_03.ipynb) | 다섯 빈출 패턴 — `exp(x)-1`, `log(1+x)`, $(1+r/n)^n - 1$, softplus, log1mexp. 각 패턴에서 naive vs `expm1`/`log1p` 활용형의 상대 오차 비교. **Rule of thumb**: *naive 식 안에서 어떤 항이 자기 항등원(0 or 1)에 접근하면, 그 항을 흡수하는 라이브러리 함수를 쓰라.* |

## 한 줄 정리
> **§1.4 의 모든 처방은 같은 패턴이다 — *부동소수점에서 위험한 연산을 식별하고, 수학적으로 동치인 다른 표현으로 우회한다.*** 부호 선택, Taylor 재작성, `expm1`/`log1p` 는 *추상화의 다른 층* 일 뿐 본질은 같다.

## 사용 라이브러리
- NumPy (`np.copysign`, `np.expm1`, `np.log1p`, `np.finfo`)
- math / Python 표준 (correctly-rounded `math.expm1`, `math.log1p`)
- mpmath (50자리 임의정밀도 — 모든 참값)
- Pandas, Matplotlib

## 다음 (Day 08 / Ch 2)
챕터 1 종료. Day 08 부터 **챕터 2 *Linear Systems*** 로 진입 — §2.1 Naïve Gaussian Elimination.
$\mathcal{O}(n^3)$ 의 wall-clock 측정, Hilbert 행렬에서 작은 $n$ 에서도 발산하는 잔차, scaled partial pivoting 의
필요성을 챕터 1 에서 배운 *부동소수점 사고* 위에 쌓아 올린다.
