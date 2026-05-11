# Day 08 — §2.1 Naïve Gaussian Elimination (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 2. Linear Systems
> **절**: §2.1 Naïve Gaussian Elimination — algorithm, cost, ill-conditioning

## 오늘의 큰 그림
챕터 1 에서 다룬 *부동소수점 사고* 위에 **선형대수** 를 쌓아 올린다.
오늘의 세 문제는 가우스 소거를 **세 각도** 로 분해한다.

1. **알고리즘** — forward elimination + back substitution 을 직접 구현하고, 잘 조건된 random matrix 에서 잔차를 측정해 `np.linalg.solve` 와 비교.
2. **비용** — 시간 측정으로 이론 $\mathcal{O}(n^3)$ 을 확인. 같은 차수 안에서도 BLAS 기반 LAPACK 과의 *상수* 차이가 어마어마함을 관찰.
3. **한계** — Hilbert 행렬에서 잔차는 작아도 *오차* 가 폭발. **잔차와 오차의 분리** 가 챕터 2 뒷부분의 동기.

세 문제를 관통하는 메시지:

| 관점 | 핵심 결과 | 어디서 무너지는가 |
|---|---|---|
| **알고리즘 정확성** | 잘 조건된 행렬: 잔차 $\sim n \varepsilon_{\text{mach}}$ | 작은 피봇이 나오는 행렬 |
| **알고리즘 비용** | $T(n) \sim \tfrac{2}{3} n^3$ | Python 루프는 상수 1000$\times$ |
| **문제 조건** | 오차 $\le \kappa \cdot$ 잔차 | $\kappa$ 가 $10^{16}$ 을 넘는 순간 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_2_1_01.ipynb`](CE_2_1_01.ipynb) | `naive_gauss(A, b)` 직접 구현 — forward elimination + back substitution. random $n \in \{5, 20, 100, 500\}$ 에서 잔차/오차 측정, NumPy `solve` 와 일대일 비교. 두 알고리즘 모두 $\sim n \varepsilon_{\text{mach}}$. |
| 2 | [`CE_2_1_02.ipynb`](CE_2_1_02.ipynb) | $n \in \{25, 50, 100, 200, 300, 400\}$ 에서 wall-clock 측정, 5 회 반복 최소값 채택. $(\log n, \log t)$ 회귀 → $\hat p \approx 3$ 확인. NumPy 와의 *상수* 차이 정량화. |
| 3 | [`CE_2_1_03.ipynb`](CE_2_1_03.ipynb) | Hilbert $H_{ij}=1/(i+j-1)$, $n \in \{5,8,10,12,15\}$. mpmath 50자리로 $\mathbf{b}$ 정확 생성. **잔차는 $10^{-16}$ 유지, 오차는 $n=12$ 부터 $\mathcal{O}(1)$ 폭발.** 이론선 $\kappa(H) \varepsilon_{\text{mach}}$ 와 실측 오차가 정확히 일치. |

## 한 줄 정리
> **챕터 2 의 첫 발은 "알고리즘이 좋아도 문제가 나쁘면 답이 나쁘다" 는 분리의 인식이다.**
> 잔차와 오차, 차수와 상수, 그리고 *어떤 행렬을 다루느냐* 가 §2.2 (pivoting), §2.3 (band), 챕터 8 (LU/SVD) 의 모든 처방을 정렬하는 축이다.

## 사용 라이브러리
- NumPy (`np.linalg.solve`, `np.linalg.cond`, `np.linalg.norm`)
- Pandas, Matplotlib (영문 라벨)
- mpmath (50자리 임의정밀 — Hilbert b 생성)

## 다음 (Day 09)
**§2.2 Gaussian Elimination with Scaled Partial Pivoting** — naïve 가 무너지는 *특정* 행렬 (예: 작은 피봇이 강제되는 시스템) 을 만들어 scaled partial pivoting 의 효과를 본다. 조건수와 잔차의 관계를 ill-conditioned 케이스에서 다시 측정.
