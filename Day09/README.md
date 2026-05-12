# Day 09 — §2.2 Gaussian Elimination with Scaled Partial Pivoting (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 2. Linear Systems
> **절**: §2.2 Scaled Partial Pivoting — pivot selection, stability, and the wall of conditioning

## 오늘의 큰 그림
§2.1 의 *naïve* 가우스 소거 위에 **scaled partial pivoting (SPP)** 을 얹는다. 오늘의 세 문제는
"피봇팅이 어디서 효과적이고 어디서 효과 없는가" 를 *세 종류의 행렬* 로 분해한다.

1. **알고리즘 자체** — SPP 를 직접 구현하고, *피봇팅이 본질적으로 필요한* 행렬을 만들어 naïve 와 비교.
2. **조건수와의 관계** — SVD 로 조건수 $\kappa(A)$ 를 통제하며 잔차/오차/이론 상한의 관계를 실측.
3. **Hilbert 의 한계** — naturally ill-conditioned 행렬에서 SPP 가 *얼마나 도와주는가*.

세 문제를 관통하는 메시지:

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **피봇 선택의 가치** | 강제 작은 피봇에서 naïve 오차 → $\mathcal{O}(1)$, SPP 오차 → $\varepsilon_{\text{mach}}$ | *행 스케일 vs 피봇 크기* |
| **잔차 (algorithm)** | SPP 잔차는 $\kappa$ 와 무관하게 $\sim \varepsilon_{\text{mach}}$ | 후방 안정성 |
| **오차 (problem)** | 오차 $\approx \kappa(A)\cdot\varepsilon_{\text{mach}}$, 알고리즘 무관 | 조건수의 본질적 벽 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_2_2_01.ipynb`](CE_2_2_01.ipynb) | `spp_gauss(A, b)` 를 인덱스 벡터 방식으로 구현. Case A (random) 에서는 naïve ≈ SPP ≈ NumPy. **Case B** ($A_\varepsilon = [[\varepsilon,1],[1,1]]$) 에서 $\varepsilon=10^{-15}$ 일 때 **naïve 오차 0.11 vs SPP 오차 2.2e-16** — 결정적 차이. Case C 는 $s_0$ 이 큰 행이 naïve 의 첫 피봇을 망치는 케이스: 잔차 1.79 vs 2e-10. |
| 2 | [`CE_2_2_02.ipynb`](CE_2_2_02.ipynb) | $n=50$, $\kappa = 10^0 \sim 10^{14}$ 로 SVD 합성 행렬 15 개. SPP 잔차는 $\kappa$ 와 *무관* 하게 $\sim 10^{-16}$ 유지, 오차는 $\kappa$ 에 *비례* . $\log_{10}(\text{rel err})$ vs $\log_{10}\kappa$ 회귀 기울기 **0.95** (이론 1). |
| 3 | [`CE_2_2_03.ipynb`](CE_2_2_03.ipynb) | Hilbert $H_n$, $n \in \{5,8,10,12,15\}$, mpmath 50자리로 $\mathbf{b}$ 정확 생성. 세 알고리즘 (naïve / SPP / NumPy) **모두** 동일한 $\kappa \cdot \varepsilon_{\text{mach}}$ 곡선을 탄다. SPP/naïve 오차 비율 보통 1 근처 — **SPP 는 Hilbert 를 못 구한다**. |

## 한 줄 정리
> **SPP 는 *알고리즘의 한계* 를 메우고, *문제의 한계* 는 못 메운다.**
> "어디서 피봇팅이 효과적인가" 는 행 스케일과 강제 작은 피봇이 있을 때이고, 조건수 자체가 큰
> 문제 (Hilbert) 에서는 알고리즘 교체보다 *임의정밀도/regularization/재정식화* 가 답이다.

## 사용 라이브러리
- NumPy (`np.linalg.solve`, `np.linalg.cond`, `np.linalg.norm`, `np.linalg.qr`)
- Pandas, Matplotlib (영문 라벨)
- mpmath (50자리 임의정밀 — Hilbert b 생성)

## 다음 (Day 10)
**§2.3 Tridiagonal and Banded Systems** — 일반 가우스 소거 $\mathcal{O}(n^3)$ 을 3중대각 시스템에
특화하면 $\mathcal{O}(n)$ 으로 떨어진다. *Thomas 알고리즘* 을 직접 구현하고, 1D 푸아송
$-u''=f$ 의 finite-difference 행렬을 풀어 본다. 5중대각으로 확장도.
