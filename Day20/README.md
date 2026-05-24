# Day 20 — §5.4 Romberg Integration (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 5. Numerical Integration
> **절**: §5.4 Romberg Integration — *Richardson 한 단계가 Simpson 이면, 무한 단계는 Romberg*

## 오늘의 큰 그림
Day 18 의 사다리꼴 $h^2$, Day 19 의 Simpson $h^4$. 이 둘 사이의 *결합 기법* 이 Simpson 의 본질이라는 것을
Day 19 Problem 1 에서 봤다. 오늘은 그 결합을 *멈추지 않는다*. Euler–Maclaurin 전개가 *짝수 차 항만* 있다는
사실을 활용해, $h^2$ 사다리를 *한없이* 쌓아 $h^4, h^6, h^8, \ldots$ 로 *기하급수적* 차수 점프를 얻는다.

세 문제를 관통하는 한 식은

$$
T[i,j] \;=\; T[i,j-1] \;+\; \frac{T[i,j-1] - T[i-1,j-1]}{4^{j} - 1},
\qquad T[i,j] - I \;=\; \mathcal O\!\bigl(h_i^{2(j+1)}\bigr).
$$

— $j$열은 $h^2$ 항을 *$j$번* 소거. 0열은 사다리꼴, 1열은 Simpson, 2열은 Boole, … 무한히 이어지는 사다리.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **기본 차수** | $j$열 = $\mathcal O(h^{2(j+1)})$, 연속 비율 $4^{j+1}$ | $f \in C^{\infty}$, 끝점 도함수 유한 |
| **비용 모델** | $i$행까지의 표 전체 = $N_f = 2^i + 1$ 함수 평가만 | Trapezoid 의 *재귀* |
| **부드러움 의존** | 가정 위반 시 *모든 열* 의 차수가 동일 ($\sqrt x \to -3/2$, $1/\sqrt x \to -1/2$) | Euler–Maclaurin 의 짝수 차 가정 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_5_4_01.ipynb`](CE_5_4_01.ipynb) | Romberg 표 $T[i,j]$ 를 *trapezoid 재귀* 와 Richardson 점화로 직접 구현. 세 부드러운 적분 ($e^x$, $\sin x$, $1/(1+x^2)$) 에 $k_{\max}=10$ 적용. **열별 기울기** $-2(j+1)$ 와 **연속 비율** $4^{j+1}$ 을 표/그림으로 검증. *Romberg 의 $j=1$ 열 = 합성 Simpson* 항등식을 수치적으로 확인 (차이 $\sim 10^{-16}$). $T[10,10]$ 은 *기계 정밀도* 한계에 도달. |
| 2 | [`CE_5_4_02.ipynb`](CE_5_4_02.ipynb) | *비용 ($N_f$) 대 정확도* 의 공정 비교 — Trapezoid (기울기 $-2$), Simpson ($-4$), Romberg 대각 (*곡선*, 국부 기울기 $-2, -4, -6, \ldots$ 로 *증가*). 세 적분 ($e^x$, $\pi$ via $4/(1+x^2)$, $x\sqrt{1-x^2}$) 에서 Simpson→Romberg break-even $N_f \sim 9$, $|E| < 10^{-10}$ 도달 시간 Romberg 가 한 자리 이상 빠름. 끝점 비매끈성 ($x\sqrt{1-x^2}$) 이 *유효 차수* 를 일찍 멈춤. |
| 3 | [`CE_5_4_03.ipynb`](CE_5_4_03.ipynb) | Romberg 의 *실패 모드* — $\sqrt x$, $1/\sqrt x$, $\|x-1/2\|$. 끝점 특이점의 두 경우는 *모든 열의 기울기가 같다* ($-3/2$, $-1/2$) — Euler–Maclaurin 의 $h^2$ 항이 없어 *외삽 무효*. **변수 변환** $x = u^2$ 로 $\sqrt x \to 2u^2$ (다항식), $1/\sqrt x \to 2$ (상수) → 완벽한 차수 사다리 *복원*. $\|x-1/2\|$ 는 $1/2$ 가 항상 격자 노드 → 사다리꼴 자체가 *exact* — *부드러움 위반 / 그리드 정렬* 의 경계 사례. |

## 한 줄 정리
> **Romberg = Trapezoid + Richardson 의 무한 사다리.**
> 부드러우면 *기하급수적* 정확도, 깨지면 *모든 열이 같은 기울기* 가 곧 *진단 신호* 다.
> *적응형* 이 데이터로 분할을 배운다면, *변수 변환* 은 부드러움을 직접 회복한다 — 둘은 상보적.

## 사용 라이브러리
- NumPy (벡터화 Trapezoid 중간점 갱신, Romberg 표 채우기)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`loglog`, 영문 라벨 — 한글 폰트 의존 회피)
- 외부 데이터 없음 — 모든 함수와 참값을 코드 안에서 정의

## 다음 (Day 21)
**§5.5 — Gaussian Quadrature**. 균등 분할로 Simpson 의 $\mathcal O(h^4)$ 가 한계였다면, *분할점 자체* 를
자유롭게 둘 수 있을 때 차수가 어디까지 올라갈까? $n$ 개 노드와 가중치로 *$2n-1$ 차* 다항식을 정확히 적분 —
*노드 수당* 두 배 점프. Legendre 다항식의 영점, 가중 직교성, 끝점 특이점에 대한 Gauss–Jacobi 가족.
Romberg 의 *반복 외삽* 과는 다른 *비할당 노드 최적화* 의 길.

> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
