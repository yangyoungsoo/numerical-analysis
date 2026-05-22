# Day 18 — §5.2 Trapezoid Rule (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 5. Numerical Integration
> **절**: §5.2 Trapezoid Rule — *대칭화로 얻는 $h^2$, Richardson 한 단계로 $h^4$, 그리고 부드러움이 결정하는 모든 것*

## 오늘의 큰 그림
Day 17 §5.1 에서 우리는 *어디서* 함수를 평가하느냐가 끝점 리만합 ($\mathcal O(h)$) 과 중점법
($\mathcal O(h^2)$) 의 차수를 가른다는 사실을 봤다. 오늘은 그 *대칭화* 의 첫 산물 —
**합성 사다리꼴 공식** 을 다지고, *짝수 차수* 의 세 가지 운명을 차례로 본다.

세 문제를 관통하는 부등식은 **Euler–Maclaurin 잔여**

$$
T_n(f) - \int_a^b f\,dx \;=\; -\,\frac{(b-a)\,h^2}{12}\,f''(\xi) \;+\; (h^4 \text{ 항}) \;+\; (h^6 \text{ 항}) \;+\;\cdots
$$

— 이 한 식의 *어떤 항* 이 살아남느냐가 모든 것을 결정한다.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **기본 차수** | $T_n - I = -(b-a) h^2 f''(\xi)/12$ — 끝점 평균이 짝수 차수를 만든다 | $f \in C^2$ 가정 |
| **Richardson 한 단계** | $S_{2n} = (T_n + 2 M_n)/3$ → $\mathcal O(h^4)$ — *공짜로* 두 차수 더 | $T_n$ 과 $M_n$ 의 leading 항이 부호 반대로 정확히 $2:1$ |
| **함수의 부드러움** | 주기 부드러움 → *지수* 수렴, 끝점 특이점 → $\mathcal O(h^{\alpha+1})$ | $f^{(2k-1)}(b) - f^{(2k-1)}(a)$ 의 거동 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_5_2_01.ipynb`](CE_5_2_01.ipynb) | 합성 사다리꼴 공식 $T_n$ 을 직접 구현, $\int_0^\pi \sin x\,dx = 2$ 에 적용. $n = 2,4,\ldots,2^{14}$ 에서 측정한 log–log 기울기가 $-2.000$ — 이론값과 4자리 일치. 연속 비율 $|e_n|/|e_{2n}| \to 4$ 와 점근 상수 $\pi^2/(6 n^2)$ 까지 확인. *대칭화* 한 번으로 끝점 리만합에서 한 차수 점프하는 것을 정량화. |
| 2 | [`CE_5_2_02.ipynb`](CE_5_2_02.ipynb) | $f(x) = e^x$ on $[0,1]$. $T_n$ 과 $M_n$ 의 오차가 *부호 반대로 정확히 $2:1$*. $(T_n + 2 M_n)/3$ 로 만든 결합과 *직접 구현한 합성 Simpson* 이 round-off 한계 내 *수치적으로 일치* — Simpson 의 정체성. log–log 기울기가 $-2, -2, -4$ 로 *두 차수* 점프. 이론 점근선 $e h^2/12$, $e h^4/180$ 와 상수까지 평행. |
| 3 | [`CE_5_2_03.ipynb`](CE_5_2_03.ipynb) | 같은 사다리꼴을 정칙성이 다른 세 함수에 적용. (a) $e^{\cos 2\pi x}$ (주기·매끄러움) → 오차가 *지수적* 으로 떨어져 $n = 16$ 만 되어도 $10^{-13}$. (b) $\sqrt x$ (끝점 특이점) → 기울기 $-1.50$. (c) $\sqrt{x(1-x)}$ (양 끝점) → 같은 기울기, 상수만 두 배. **Euler–Maclaurin 의 부드러움 → 차수** 사슬을 정량 확인. |

## 한 줄 정리
> **사다리꼴 공식의 운명은 자기 자신이 아니라 *피적분 함수의 정칙성* 이 결정한다.**
> 매끄러운 함수에는 $\mathcal O(h^2)$, 매끄러운 *주기* 함수에는 *지수* 수렴,
> 끝점 특이점이 있는 함수에는 차수 $\alpha+1$. Richardson 한 단계만 더 가면 Simpson 의 $\mathcal O(h^4)$ —
> *짝수 차수 사다리* 의 첫 칸.

## 사용 라이브러리
- NumPy (벡터화 사다리꼴 / 중점 / Simpson 합)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`loglog`, `semilogy`, 영문 라벨)
- SciPy (`scipy.special.iv` 로 $I_0(1)$ — Problem 3 의 (a) 참값)
- 외부 데이터 없음 — 전부 코드 안에서 생성

## 다음 (Day 19)
**§5.3 — Simpson's Rule and Adaptive Numerical Integration**.
오늘 Problem 2 에서 본 $S_{2n}$ 을 *그 자체의 규칙* 으로 다지고, 짝수 차수 사다리의
다음 칸 ($h^6$) 으로 한 발 더 나아간다. 그리고 Problem 3 의 *끝점 특이점* 문제에
**적응형 분할** 로 대처하는 길 — *부분구간마다 다른 $h$* 를 자동으로 고르는 알고리즘 —
을 직접 구현한다.

> 본 노트북 시리즈는 `_meta/curriculum.md` 의 *향후 (Day 14+)* 예고에 따라
> 챕터 5 §5.2 로 *같은 호흡* 으로 이어간다.
> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (보간 → 수치미분 → 수치적분 → 보간/Spline → ODE → …) 을 따라 자동 진행 중이다.
