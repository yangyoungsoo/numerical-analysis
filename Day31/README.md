# Day 31 — §8.1 Matrix Factorizations: LU 분해 (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 8. Additional Topics on Solving Linear Systems
> **절**: §8.1 Matrix Factorizations — *Doolittle / 부분 피벗팅 / 다중 RHS 재사용*

## 오늘의 큰 그림
Day 30 의 implicit RK 가 *한 스텝 = 하나의 비선형 시스템* 이라는 청구서를 남겼다. 그 청구서를
$\bigl(I - h(A\otimes J)\bigr)$ 라는 *sparse block 선형 시스템* 으로 깎고 나면 결국 모든 비용이
**LU 분해의 비용** 으로 환원된다. 챕터 8 의 출발점인 §8.1 은 그 *LU 분해* 자체를 다시 본다 —
*피벗 없는* Doolittle, *부분 피벗팅* (PA = LU), 그리고 *분해 재사용* (한 번 분해, 여러 번 풀이).

세 문제를 관통하는 *한 식* — **PA = LU 와 그 비용**:

$$
\boxed{\;
P A = L U,
\qquad
\text{factorize: } \tfrac{2}{3} n^3,
\qquad
\text{each new RHS: } 2 n^2,
\qquad
\rho_n = \frac{\max|U|}{\max|A|}.
\;}
$$

| 관점 | 결과 | 무엇이 결정하나 |
|---|---|---|
| **분해 비용** | $\tfrac{2}{3} n^3$ flop, $\mathcal{O}(n^3)$ 시간 | 행렬 크기 $n$ |
| **추가 RHS 비용** | $2 n^2$ flop per RHS | 전진 + 후진 대입 |
| **부분 피벗팅** | $|L_{ij}| \le 1$ 보장 | 매 단계 절댓값 최대 행 선택 |
| **Growth factor** | $\rho_n \le 2^{n-1}$ (최악), $\sim\sqrt n$ (평균) | 분해 도중 원소의 크기 변화 |
| **속도 향상 비율** | $\frac{\tfrac{2}{3} m n^3}{\tfrac{2}{3} n^3 + 2 m n^2} \to \tfrac{n}{3}$ | 다중 RHS 의 *수* $m$ |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_8_1_01.ipynb`](CE_8_1_01.ipynb) | **Doolittle LU** (피벗 없이) 직접 구현. $n \in \{50,100,200,400,800\}$ 에서 wall-clock 측정 → `polyfit` 으로 경험 지수 $\widehat p \approx 3$ 산출. 분해 정확도 $\|LU - A\|_F / \|A\|_F$ 가 모든 크기에서 $\sim 10^{-14}$ — 대각 우세 행렬에서는 *피벗 없이도 안정*. 단, $u_{ii} = 0$ 일 때 즉시 실패 — 다음 문제에서 *부분 피벗팅* 으로 해결. |
| 2 | [`CE_8_1_02.ipynb`](CE_8_1_02.ipynb) | **부분 피벗팅 LU** (PA = LU) 구현 + **growth factor** 분석. 세 *family* 비교: (a) **Hilbert** ($\rho_n \sim 1$ 그러나 $\kappa_\infty \sim 10^{0.5 n}$), (b) **Wilkinson kite** ($\rho_n = 2^{n-1}$ — PP 최악 경우 *도달*), (c) **Random Gaussian** ($\rho_n \sim \sqrt n$). 한 그래프에 세 family 의 $\rho_n$ + 참조선 $2^{n-1}$ / $\sqrt n$ — *부분 피벗팅은 평균 안전하지만 최악 경우 보장 X*. |
| 3 | [`CE_8_1_03.ipynb`](CE_8_1_03.ipynb) | **LU 재사용**: (a) 단일 RHS 의 잔차, (b) **다중 RHS** 의 *재인수분해* vs *LU 재사용* wall-clock — $n=200, m=100$ 에서 속도 향상 $\sim 60\times$ (이론 $n/3 \approx 67$ 와 일치), (c) **역행렬** $A^{-1}$ 을 $A X = I$ 로 ($\sim 8/3 n^3$), (d) **행렬식** $\det(P)^{-1}\prod U_{ii}$ + overflow-safe $\sum \log\|U_{ii}\|$. *시스템을 풀 거라면 $A^{-1}$ 를 명시적으로 만들지 말 것* — LU 만 보관. |

## 한 줄 정리
> **$PA = LU$ 한 번의 분해 ($\tfrac{2}{3} n^3$) 가 임의 개수의 시스템 풀이, 역행렬, 행렬식 모두를
> $\mathcal{O}(n^2)$ 의 추가 비용으로 해결한다.** 부분 피벗팅은 *평균* 안전 ($\rho_n \sim \sqrt n$) 하지만
> Wilkinson kite 같은 *최악 경우* 에는 $2^{n-1}$ 까지 자랄 수 있다 — *growth factor 모니터링* 이 표준 관행.

## 사용 라이브러리
- NumPy (`np.linalg.solve`, `np.linalg.cond` 와 비교용 / 자체 구현은 순수 파이썬 루프)
- SciPy (`scipy.linalg.hilbert` 로 Hilbert 행렬 생성, `scipy.linalg.lu` 와 검증)
- Pandas (`pd.set_option("display.float_format", ...)` 로 표 출력)
- Matplotlib (loglog 스케일링 그래프, growth factor 비교 — 영문 라벨)
- 외부 데이터 없음 — 모든 행렬은 코드 안에서 생성

## 다음 Day 예고
**Day 32 — §8.2 Iterative Solutions of Linear Systems**: Jacobi / Gauss–Seidel / SOR 의 *고정점 반복* 으로
$\mathcal{O}(n^3)$ 의 직접 분해를 *sparse* 행렬에서는 $\mathcal{O}(n)$ per iter 로 대체. *수렴 조건*
(spectral radius $< 1$) 과 *완화 매개변수* $\omega$ 의 최적값. Day 30 의 implicit RK 의 *Jacobian 재인수분해*
를 피하는 또 다른 길.

## 메모
* `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이고 그 이후는 *책의 흐름* (Ch 4 → … → Ch 8 LU/SVD)
  을 따라 자동 진행 중이다. Day 30 README 의 예고 (§8.1 LU 재방문) 를 그대로 받아 진입.
* 다음 절 (§8.2 iterative methods) 는 Cheney & Kincaid 7판의 자연스러운 순서.
* §8.3–§8.4 (LU 재해석 / SVD) 이후 Ch 9 (최소제곱) 로 진입 예정.
