# Day 34 — §9.1 Linear Least Squares: 정규방정식·QR·SVD (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 9. The Method of Least Squares
> **절**: §9.1 Linear Least Squares — *정규방정식 / QR 분해 / SVD·절단 SVD*

## 오늘의 큰 그림
Ch 8 까지는 *정방·비특이* 선형계 $A\mathbf{x}=\mathbf{b}$ 였다. 이제 **과결정(overdetermined)** 으로
넘어간다 — $A\in\mathbb{R}^{m\times n}$, $m>n$ 이라 일반적으로 정확한 해가 없고, 대신 잔차의 2-노름을
최소화한다. 같은 문제를 푸는 **세 가지 사다리**가 있고, 셋의 차이는 전부 *조건수*로 설명된다.

세 문제를 관통하는 *한 식* — **최소제곱의 정규방정식과 조건수 제곱**:

$$
\boxed{\;
A^\top A\,\mathbf{c}=A^\top\mathbf{y}
\quad\Longrightarrow\quad
\kappa_2(A^\top A)=\kappa_2(A)^2 ,
\qquad
\mathbf{x}^+=\sum_i\frac{\mathbf{u}_i^\top\mathbf{b}}{\sigma_i}\mathbf{v}_i .
\;}
$$

| 방법 | 비용 | 정확도(상대오차) | 언제 쓰나 |
|---|---|---|---|
| **정규방정식** | $\tfrac12 mn^2$ (가장 쌈) | $\sim\kappa(A)^2\,\varepsilon$ | 조건 좋고 빠른 적합 |
| **QR 분해** | $\sim 2mn^2$ | $\sim\kappa(A)\,\varepsilon$ | 표준 — 조건 나빠도 안정 |
| **SVD / TSVD** | $\sim$ 가장 비쌈 | $\sim\kappa(A)\,\varepsilon$ + 정칙화 | rank 부족 / ill-posed / 잡음 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_9_1_01.ipynb`](CE_9_1_01.ipynb) | **다항식 최소제곱 + 정규방정식**. 잡음 데이터에 차수 $d=1\!\to\!15$ 다항식을 정규방정식으로 적합. 잔차 노름은 단조 감소(과소→과대적합)하지만 $\kappa_2(A^\top A)\approx\kappa_2(A)^2$ 가 폭증해 $d\!\approx\!12$–$15$ 에서 $1/\varepsilon_{\text{mach}}\!\approx\!10^{16}$ 에 근접 — *적합력 vs 수치안정성* trade-off 를 표·그래프로 확인. |
| 2 | [`CE_9_1_02.ipynb`](CE_9_1_02.ipynb) | **정규방정식 vs QR**. 참해를 아는 **Läuchli 행렬**에서 $\kappa(A)$ 를 $10^0\!\to\!10^{16}$ 로 키우며 상대오차 측정. log-log 에서 정규방정식은 *기울기 2*($\kappa^2\varepsilon$), QR·SVD 는 *기울기 1*($\kappa\varepsilon$). 정규방정식은 $\kappa\!\approx\!10^8$($\sqrt{\varepsilon_{\text{mach}}}$)에서 $A^\top A$ 가 수치적 특이가 되어 붕괴, QR 은 $10^{16}$ 까지 버팀. |
| 3 | [`CE_9_1_03.ipynb`](CE_9_1_03.ipynb) | **SVD / 의사역행렬 / 절단 SVD(TSVD)**. 매끄러운 가우시안 커널을 이산화한 ill-posed 문제($\kappa\!\approx\!2\times10^{16}$). 전체 의사역해는 작은 $\sigma_i$ 로 나눈 잡음이 폭증해 상대오차 $\sim\!10^{12}$. **TSVD**($k^\star\!=\!18$)는 작은 특이값을 잘라 매끄러운 참해를 상대오차 $4.5\times10^{-3}$ 로 복원. **Picard 그림**과 편향–분산 L-curve 로 최적 절단점 시각화. |

## 한 줄 정리
> **최소제곱의 세 사다리(정규방정식·QR·SVD)는 모두 같은 해를 노리지만, 정규방정식은 $A^\top A$ 를 형성하며
> 조건수를 제곱하고, QR 은 $A$ 를 직접 직교화해 $\kappa$ 수준 손실에 그치며, SVD/TSVD 는 작은 특이값을 통제해
> rank 부족·잡음 문제까지 정칙화한다.** 조건수가 모든 차이를 설명한다.

## 사용 라이브러리
- NumPy (`np.vander`, `np.linalg.solve/qr/svd/lstsq/cond`, 순수 행렬 연산)
- SciPy (검증용 비교군)
- Pandas (잔차·조건수·유효자릿수·Picard 계수 표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (적합 곡선, log-log 조건수 기울기, 특이값 스펙트럼·L-curve — 영문 라벨)
- 외부 데이터 없음 — 모든 행렬/데이터는 코드 안에서 생성

## 다음 Day 예고
**Day 35 — §9.x 비선형/연속 최소제곱**: 선형 모델을 넘어 **가우스–뉴턴 / 레벤버그–마쿼트**(비선형 적합)와
**직교다항식 기반 연속 최소제곱**(정규방정식의 조건수 문제를 기저 선택으로 회피)으로 확장. 이후 Ch 10(Monte Carlo)로 진입.

## 메모 (사용자 알림)
* **커리큘럼 종료 알림**: `_meta/curriculum.md` 의 *명시적* 진도표는 Day 13(§3.3)까지이며, 그 이후(Day 14~)는
  책의 챕터 흐름(Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → Ch 8 선형계 추가주제 → **Ch 9 최소제곱**)을
  따라 스케줄러가 자동 진행 중입니다. 오늘은 Day 33(§8.3) README 의 예고대로 **Ch 9 §9.1** 로 진입했습니다.
* 진도 방향(챕터 순서·특정 절 심화 등)을 바꾸시려면 `_meta/curriculum.md` 에 `Day 34+` 항목을 추가해 주세요 —
  스케줄러가 명시적 항목을 우선합니다.
* 다음 예정: Ch 9 마무리 → Ch 10(Monte Carlo) → Ch 11(BVP) → Ch 12(PDE) → Ch 13(최적화).
