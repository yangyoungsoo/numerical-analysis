# Day 33 — §8.3 Krylov 부분공간 / 켤레기울기 (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 8. Additional Topics on Solving Linear Systems
> **절**: §8.3 Krylov Subspace Methods — *Conjugate Gradient (CG) / 조건수 / 전처리*

## 오늘의 큰 그림
Day 31 의 §8.1 은 *직접법* ($\tfrac{2}{3}n^3$ 한 번의 LU), Day 32 의 §8.2 는 *고전 반복법*
(Jacobi/GS/SOR, $\rho(G)<1$) 을 정리했다. 둘 사이에 **세 번째 길** 이 있다 — sparse SPD 행렬에서
행렬–벡터 곱만으로, *Krylov 부분공간* 위의 최소화를 통해 수렴하는 **켤레기울기(CG)**.
SOR 의 다이얼이 *완화 매개변수* $\omega$ 였다면, CG 의 다이얼은 *스펙트럼의 모양* —
조건수 $\kappa$, 그리고 그것을 줄이는 **전처리(preconditioning)** 다.

세 문제를 관통하는 *한 식* — **CG 의 $A$-노름 최소화와 그 수렴 한계**:

$$
\boxed{\;
\mathbf{x}_k \in \mathbf{x}_0 + \mathcal{K}_k(A,\mathbf{r}_0),
\quad
\|\mathbf{e}_k\|_A = \min !
\quad\Longrightarrow\quad
\frac{\|\mathbf{e}_k\|_A}{\|\mathbf{e}_0\|_A}
\le 2\left(\frac{\sqrt{\kappa}-1}{\sqrt{\kappa}+1}\right)^{k}.
\;}
$$

| 관점 | 결과 | 무엇이 결정하나 |
|---|---|---|
| **반복당 비용** | 행렬–벡터 곱 1회 + 내적 2회 ($\mathcal{O}(\text{nnz})$) | 행렬의 희소성 |
| **수렴 한계** | $\mathcal{O}(\sqrt{\kappa})$ 반복 | 조건수 $\kappa=\lambda_{\max}/\lambda_{\min}$ |
| **유한 종료** | 서로 다른 고유값 $d$개 → $\le d$ 반복 | 스펙트럼의 *모양* |
| **전처리** | $\kappa(M^{-1}A)\ll\kappa(A)$ → 반복수 급감 | $M\approx A$ (Jacobi / IC(0)) |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_8_3_01.ipynb`](CE_8_3_01.ipynb) | **CG 직접 구현** ($\alpha,\beta$ 두 줄 점화식). 2D 푸아송 모델($n=m^2$)에서 CG vs Gauss–Seidel vs SOR$_\text{opt}$ 반복수 비교 — GS 는 $\mathcal{O}(m^2)$, CG/SOR 는 $\mathcal{O}(m)$ 이며 CG 가 일관되게 최소. **오차 $A$-노름의 단조 감소** 와 **유한 종료**($\le n$ 반복) 를 직접 확인. 잔차 곡선은 GS 의 직선(선형) 대 CG 의 *아래로 볼록* 한 초선형 하강. |
| 2 | [`CE_8_3_02.ipynb`](CE_8_3_02.ipynb) | **수렴률 vs 조건수**. 고유값을 $[1,\kappa]$ 에 배치한 SPD 행렬($Q\Lambda Q^\top$)에서 반복수가 $\sqrt{\kappa}$ 에 *선형* 임을 적합 기울기로 확인 ($\kappa\!:\!10\to10^4$ 일 때 반복수는 $\sim32$배만 증가). 실측 $A$-노름 오차가 **체비쇼프 상한** 아래에 놓임을 검증. **고유값 군집**: $\kappa$ 동일(=1000)이라도 서로 다른 고유값이 $d$개면 CG 는 $\sim d$ 반복에 종료 — *조건수가 전부가 아니다*. |
| 3 | [`CE_8_3_03.ipynb`](CE_8_3_03.ipynb) | **전처리 CG (PCG)**: $\mathbf{z}=M^{-1}\mathbf{r}$ 점화식 구현. 2D 푸아송에서 (a) 전처리 없음, (b) **Jacobi(대각)**, (c) **불완전 콜레스키 IC(0)** 비교. 측정 조건수 $\kappa(M^{-1}A)$ + 반복수 표. 대각이 균일한 푸아송에서 Jacobi 는 거의 무효($\kappa$ 불변)지만 IC(0) 는 $\kappa$ 를 낮춰 반복수를 절반 이하로. **스펙트럼 산점도** 로 $M^{-1}A$ 고유값이 1 주위로 군집함을 시각화. |

## 한 줄 정리
> **CG 는 SPD 행렬에서 Krylov 부분공간 위의 $A$-노름 최소화를 두 줄 점화식으로 구현하여 $\mathcal{O}(\sqrt{\kappa})$
> 반복으로 수렴한다.** 수렴 속도를 결정하는 진짜 지렛대는 *스펙트럼의 모양* — 고유값 군집과 전처리($M\approx A$)가
> $\kappa$ 를 깎아 반복수를 직접 줄인다. 직접법·고전 반복법·Krylov 의 삼각형이 이로써 완성된다.

## 사용 라이브러리
- NumPy (`np.kron` 2D 라플라시안, `np.linalg.eigvalsh`/`cond` 로 스펙트럼·조건수, 순수 루프 CG/PCG/IC(0))
- SciPy (검증용)
- Pandas (반복수·조건수 표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (semilogy 수렴 곡선, $\sqrt{\kappa}$ 선형 적합, 스펙트럼 산점도 — 영문 라벨)
- 외부 데이터 없음 — 모든 행렬은 코드 안에서 생성

## 다음 Day 예고
**Day 34 — Ch 9 §9.x Least Squares / 정규방정식**: 정방·비특이를 넘어 *과결정(overdetermined)*
시스템 $A\mathbf{x}\approx\mathbf{b}$ ($m>n$) 로. 정규방정식 $A^\top A\mathbf{x}=A^\top\mathbf{b}$ 의
조건수 제곱 문제, **QR 분해** 와 **특이값 분해(SVD)** 를 통한 안정적 최소제곱으로 진입.

## 메모
* `_meta/curriculum.md` 의 명시적 진도표는 Day 13(§3.3)까지이며, 이후는 *책의 흐름*
  (Ch 4 → … → Ch 8) 을 따라 자동 진행 중. Day 32 README 의 예고(§8.3 Krylov/CG)를 그대로 받아 진입.
* Cheney & Kincaid 7판에서 §8.3 은 CG/Krylov 와 전처리를 다루며, 본 노트북의 *2D 푸아송 모델 문제*,
  *체비쇼프 수렴 한계*, *IC(0) 전처리* 는 해당 절의 표준 결과를 따른다.
* **커리큘럼 종료 알림**: 명시적 항목(Day 13)은 한참 전에 끝났고, 현재는 챕터 흐름 기반 자동 진행입니다.
  진도 방향(예: 챕터 순서 변경, 특정 절 심화)을 조정하시려면 `_meta/curriculum.md` 에 `Day 34+` 항목을
  추가해 주세요 — 스케줄러가 그것을 우선합니다.
* 다음 챕터(Ch 9 최소제곱/SVD) 이후 Ch 10(Monte Carlo) → Ch 11(BVP) → Ch 12(PDE) → Ch 13(최적화) 예정.
