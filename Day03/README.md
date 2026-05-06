# Day 03 — §1.1 (continued) Computer Exercises 7–9

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 1. Mathematical Preliminaries and Floating-Point Representation
> **절**: §1.1 Introduction (continued)

## 오늘의 큰 그림
같은 *값*을 계산하더라도 **(1) 연산 순서**, **(2) 공식의 형태**, **(3) 정밀도**의 세 가지 변수에 의해 결과 자릿수가 크게 달라진다. 셋 모두 부동소수점 산술의 *비결합성*과 *상쇄(cancellation)*에서 출발하는 같은 이야기이다.

| 각도 | 예시 | 교훈 |
|---|---|---|
| 합의 **순서** | $\sum 1/k$ forward vs backward | 비슷한 크기끼리 먼저 더해라 |
| 공식의 **형태** | variance 2-pass vs 1-pass | 큰 값들의 차로 작은 값을 구하지 말라 |
| **정밀도** 자체 | float32 vs float64 | rel. error ≈ κ · ε — 정밀도는 한정 자원 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 7 | [`CE_1_1_07.ipynb`](CE_1_1_07.ipynb) | 조화합 $\sum_{k=1}^{N} 1/k$ 의 누적 순서가 누적 round-off에 미치는 영향 — backward(작은→큰)가 일관되게 더 정확 |
| 8 | [`CE_1_1_08.ipynb`](CE_1_1_08.ipynb) | 분산의 2-pass(안정) vs 1-pass(catastrophic cancellation) — $\mu/s$가 커질수록 1-pass는 부호조차 음수로 |
| 9 | [`CE_1_1_09.ipynb`](CE_1_1_09.ipynb) | 다항식 평가 / forward difference / 조화합 세 실험을 float32와 float64로 함께 — *correct decimal digits* 정량 비교 |

## 한 줄 정리
> **수학적 동등성 ≠ 수치적 동등성**. 같은 결과를 향한 여러 길이 있고, 그중 어떤 길은 부동소수점 산술의 비결합성에 *덜* 노출되어 있다 — 그 길을 고르는 것이 수치해석의 역할이다.

## 사용 라이브러리
- NumPy, Pandas, Matplotlib, `fractions.Fraction` (조화합 기준값으로 일부 사용)

## 다음 (Day 04)
§1.1 의 마지막 묶음 — *loss of significance* 패턴들: $(1-\cos x)/x^2$, $\sqrt{x^2+1}-x$, 그리고 일반적인 작은 두 수의 차에서 발생하는 cancellation. 항등식 재작성으로 자릿수를 *복원*하는 연습으로 이어진다.
