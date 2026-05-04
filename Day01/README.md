# Day 01 — §1.1 Introduction (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 1. Mathematical Preliminaries and Floating-Point Representation
> **절**: §1.1 Introduction — *First Programming Experiment*

## 오늘의 큰 그림
수치미분 한 가지 예제(전진차분)를 통해, **부동소수점 산술의 본질적 한계**를 보는 것이 목표.
- 절단오차(truncation): $h$가 클수록 큼, $\mathcal{O}(h)$
- 반올림오차(round-off): $h$가 작을수록 큼, $\mathcal{O}(h^{-1})$
- 두 오차의 trade-off로 인해 어느 적정 $h^*$ 에서 총오차가 최소가 됨 (U자형 곡선).

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_1_1_01.ipynb`](CE_1_1_01.ipynb) | 본문 의사코드(`First`)를 그대로 구현 — 전진차분 오차 U자형 확인 |
| 2 | [`CE_1_1_02.ipynb`](CE_1_1_02.ipynb) | 다양한 함수($1/x$, $\log x$, $e^x$, $\tan x$, $\cosh x$, $x^3-23x$)에 같은 실험 — $h^*$ 가 $|f|, |f''|$ 에 어떻게 의존하는지 |
| 3 | [`CE_1_1_03.ipynb`](CE_1_1_03.ipynb) | 중심차분 $(f(x+h)-f(x-h))/2h$ — 절단오차 모델, 반올림오차 모델, 총오차 모델 함께 그래프로 비교 |

## 한 줄 정리
> "$h \to 0$ 이면 정확해진다"는 이상이지만, 부동소수점에서는 **$\sqrt{\varepsilon_{\text{mach}}}$** 부근에서 한계가 온다. 차분 공식의 차수를 올리면 (전진 → 중심) 그 한계 자체가 더 깊어진다 ($\varepsilon^{1/2} \to \varepsilon^{2/3}$).

## 사용 라이브러리
- NumPy, Pandas, Matplotlib

## 다음 (Day 02)
§1.1 의 나머지 Computer Exercises (4번 이후) — Maclaurin 급수, $\pi$ 추정, Vandermonde 등 다항식 계산.
