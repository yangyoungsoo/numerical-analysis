# Day 32 — §8.2 Iterative Solutions of Linear Systems (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 8. Additional Topics on Solving Linear Systems
> **절**: §8.2 Iterative Solutions — *Jacobi / Gauss–Seidel / SOR 와 수렴 조건*

## 오늘의 큰 그림
Day 31 의 §8.1 은 *한 번의 $\tfrac{2}{3}n^3$ 분해* 로 임의 개수의 시스템을 푸는 **직접법** 의
경제학을 정리했다. 그러나 implicit RK 가 남기는 $\bigl(I - h(A\otimes J)\bigr)$ 같은 행렬은
*sparse* 하다 — 0 이 압도적으로 많다. 이런 행렬에 dense LU 의 $n^3$ 를 쓰는 것은 낭비다.
§8.2 의 **반복법** 은 행렬을 *분해하지 않고* 행렬–벡터 곱만으로 해에 *수렴* 시킨다. 비용의
다이얼이 바뀐다 — *분해* 에서 *반복 횟수* 로.

세 문제를 관통하는 *한 식* — **정상 반복과 그 스펙트럼 반경**:

$$
\boxed{\;
\mathbf{x}^{(k+1)} = G\,\mathbf{x}^{(k)} + \mathbf{c},
\qquad
\text{수렴} \iff \rho(G) < 1,
\qquad
\omega_{\text{opt}} = \frac{2}{1+\sqrt{1-\rho_J^2}}.
\;}
$$

| 방법 | 반복행렬 $G$ | 모델 문제 $\rho(G)$ | 점근 비용 (반복수) |
|---|---|---|---|
| **Jacobi** | $D^{-1}(L+U)$ | $\cos\pi h$ | $\mathcal{O}(h^{-2})$ |
| **Gauss–Seidel** | $(D-L)^{-1}U$ | $\cos^2\pi h = \rho_J^2$ | $\tfrac12\mathcal{O}(h^{-2})$ |
| **SOR$(\omega_{\text{opt}})$** | $(D-\omega L)^{-1}[(1-\omega)D+\omega U]$ | $\omega_{\text{opt}}-1 \approx 1-2\pi h$ | $\mathcal{O}(h^{-1})$ |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_8_2_01.ipynb`](CE_8_2_01.ipynb) | **Jacobi vs Gauss–Seidel** 직접 구현. 모델 문제 $\mathrm{tridiag}(-1,2,-1)$, $n\in\{10,20,40,80,160\}$ 에서 반복 횟수 측정 → `iters_J/iters_GS ≈ 2.0`. 반복행렬 스펙트럼으로 **Young 의 정리** $\rho(G_{GS}) = \rho(G_J)^2$ 를 기계 정밀도($\sim10^{-14}$) 수준에서 검증. 점근 수렴률 $-\log_{10}\rho$ = *반복당 정확 자릿수* → GS 가 정확히 *두 배*. 단 $\rho_J=\cos(\pi/(n+1))\to1$ 이라 둘 다 $n$ 과 함께 느려짐 → SOR 가 필요. |
| 2 | [`CE_8_2_02.ipynb`](CE_8_2_02.ipynb) | **SOR** + **최적 완화 $\omega_{\text{opt}}$**. 2D 푸아송(5점 스텐실, $n=m^2$) 에서 $\omega\in(0.8,1.95)$ 를 휩쓸어 *U자형* 반복 곡선의 최저점 추출. 경험값 $\omega_{\text{opt}}^{\text{emp}}\approx1.65$ vs 이론 $2/(1+\sin\pi h)\approx1.61$ 일치. 격자 스케일링: GS $\propto m^2$ vs SOR$_{\text{opt}}\propto m$ — *하나의 스칼라* 로 얻는 *한 차수* 가속, 측정 speedup 이 $m$ 에 비례. |
| 3 | [`CE_8_2_03.ipynb`](CE_8_2_03.ipynb) | **수렴 조건**. (a) *엄격 대각 우세* → 두 방법 모두 $\rho<1$. (b) *독립적 반례* 두 개: $A_1$ 은 $\rho_J=0<1<\rho_{GS}=2$ (Jacobi 수렴, GS 발산), $A_2$ 는 $\rho_J=1.5>1>\rho_{GS}=0.35$ (정반대) — 실제 반복으로 잔차 $10^{61}$ 폭발 vs $10^{-17}$ 수렴 확인. (c) 무작위 **SPD** 40개: GS 는 *전부* 수렴, Jacobi 는 *30개* 발산 — **Householder–John 정리**. 모든 판정은 단 하나 $\rho(G)<1$. |

## 한 줄 정리
> **반복법의 수렴은 행렬 구조가 아니라 *반복행렬의 스펙트럼 반경* $\rho(G)<1$ 하나로 결정된다.**
> 모델 문제에서 GS 는 Jacobi 의 *두 배*($\rho_{GS}=\rho_J^2$), SOR$_{\text{opt}}$ 는 *한 차수*
> ($\mathcal{O}(m^2)\!\to\!\mathcal{O}(m)$) 빠르다. 대각 우세는 *둘 다*, SPD 는 *GS 만* 수렴을 보장한다.

## 사용 라이브러리
- NumPy (행렬 분할 $D,L,U$, `np.linalg.eigvals` 로 스펙트럼 반경, `np.kron` 으로 2D 라플라시안)
- Pandas (반복 횟수·스펙트럼 표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (잔차 수렴 semilogy, $\omega$-sweep U자 곡선, SPD 산점도 — 영문 라벨)
- 외부 데이터 없음 — 모든 행렬은 코드 안에서 생성

## 다음 Day 예고
**Day 33 — §8.3 Krylov 부분공간 / 켤레기울기(CG)**: 고정점 반복의 $\rho(G)$ 한계를 넘어선다.
SPD 행렬에서 **CG** 는 매 반복 *Krylov 부분공간* $\mathrm{span}\{r_0, Ar_0, \dots\}$ 위에서
오차의 $A$-노름을 최소화하여, SOR 의 $\mathcal{O}(\sqrt\kappa)$… 더 정확히는 조건수 $\kappa$ 의
*제곱근* 에 비례하는 반복 횟수를 달성한다. *전처리(preconditioning)* 가 그 다음 다이얼.

## 메모
* `_meta/curriculum.md` 의 명시적 진도표는 Day 13(§3.3)까지이며, 이후는 *책의 흐름* (Ch 4 → … →
  Ch 8) 을 따라 자동 진행 중. Day 31 README 의 예고(§8.2 iterative methods)를 그대로 받아 진입.
* Cheney & Kincaid 7판에서 §8.2 는 Jacobi/GS/SOR 를 다루며, 본 노트북의 *모델 문제* 와
  *Young 의 정리* ($\rho_{GS}=\rho_J^2$, $\omega_{\text{opt}}$) 는 해당 절의 표준 결과를 따른다.
* 다음 절(§8.3) 부터는 *Krylov/CG* — 직접법과 고전 반복법 사이의 *세 번째 길*.
