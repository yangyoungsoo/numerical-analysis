# Day 02 — §1.1 (continued) Computer Exercises 4–6

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 1. Mathematical Preliminaries and Floating-Point Representation
> **절**: §1.1 Introduction (continued)

## 오늘의 큰 그림
"수학적으로 옳은 표현"이 부동소수점 산술에서 **수치적으로는 위험하거나 비효율** 일 수 있다는 점을, 세 가지 다른 각도에서 실험한다.

| 각도 | 예시 | 교훈 |
|---|---|---|
| 급수의 **수렴 속도** | $\arctan(1)$ Maclaurin vs Machin | 같은 함수도 *어디서 전개하느냐*에 따라 비용이 천차만별 |
| 다항식 **근의 민감도** | Wilkinson polynomial | monomial 기저는 ill-conditioned |
| 다항식 **평가의 효율/안정** | Horner vs naïve | $\mathcal{O}(n)$ 곱셈 + backward stable |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 4 | [`CE_1_1_04.ipynb`](CE_1_1_04.ipynb) | Gregory–Leibniz 급수로 $\pi$ 추정 — $\mathcal{O}(1/N)$ 의 느린 수렴, Machin 공식과의 비교 |
| 5 | [`CE_1_1_05.ipynb`](CE_1_1_05.ipynb) | Wilkinson 다항식 $\prod_{k=1}^{20}(x-k)$ 의 계수 $a_{19}$ 에 $2^{-23}$ 상대 섭동 — 일부 근이 복소쌍으로 갈라짐 |
| 6 | [`CE_1_1_06.ipynb`](CE_1_1_06.ipynb) | 차수 50 다항식을 naïve / iter / Horner 세 방법으로 평가, `Decimal` 기준 절대오차 + 연산 횟수 / 시간 비교 |

## 한 줄 정리
> 같은 수식이라도 **표현 / 평가 순서 / 기저** 의 선택이 결과 정확도를 좌우한다 — *수학과 수치는 다르다*.

## 사용 라이브러리
- NumPy, Pandas, Matplotlib, `decimal.Decimal` (고정밀 기준값)

## 다음 (Day 03)
§1.1 의 마지막 묶음 — 합산 순서가 결과에 미치는 영향, 분산의 1-pass vs 2-pass 공식의 안정성, 단정도 vs 배정도 비교.
