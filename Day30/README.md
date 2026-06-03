# Day 30 — §7.4 Implicit Runge–Kutta Methods (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 7. Initial-Value Problems
> **절**: §7.4 Implicit Runge–Kutta Methods — *stability region 을 좌반평면 전체로 확장*

## 오늘의 큰 그림
Day 27–29 의 *explicit RK* 는 *차수* 와 *적응성* 두 다이얼을 풀었다. 그러나 *stiff IVP*
(스케일이 광범위하게 분리된 시스템) 에서는 *안정 영역* 이 좁아 비현실적으로 작은 $h$ 가 강요된다.
§7.4 에서는 *implicit RK* — 한 스텝마다 *비선형 시스템* 을 푸는 대가로 *안정 영역 전체 좌반평면* 을
얻는다. 이번에는 *trade* 다이얼이 바뀐다 — *스텝 비용* ↑, *step 수* ↓.

세 문제를 관통하는 *한 식* — **암시적 RK 의 amplification factor**:

$$
\boxed{\;
R(z)\;=\;1 + z\,\mathbf{b}^{\!\top}(I - zA)^{-1}\mathbf{1},\qquad z = h\lambda.
\;}
$$

| 관점 | 결과 | 무엇이 결정하나 |
|---|---|---|
| **Forward Euler** | $R(z)=1+z$ — *원판 하나* | $\Rightarrow$ stiff 에서 *비참* |
| **Backward Euler** | $R(z)=\tfrac{1}{1-z}$ — A-, L-안정 | *order 1* — 정확도 한계 |
| **Trapezoidal** (1-stage Gauss) | $R(z)=\tfrac{1+z/2}{1-z/2}$ — A-안정, $R(-\infty)=-1$ | *order 2*, L-안정 X |
| **2-stage Gauss IRK** | Padé(2,2), $R(-\infty)=1$ | *order 4*, symplectic-friendly |
| **2-stage Radau IIA** | Padé(1,2), $R(-\infty)=0$ | *order 3*, **L-안정** — SciPy 의 기반 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_7_4_01.ipynb`](CE_7_4_01.ipynb) | **Forward vs Backward Euler** 의 amplification factor 도출 + 안정 영역 도해. $\lambda=-1000$, $T=0.05$ 에서 $h\in\{10^{-1},\ldots,10^{-4}\}$ — FE 는 $h\ge 10^{-3}$ 부터 폭주, BE 는 *전부 안정*. 같은 그림에 $h\lambda$ 점 4개를 *놓아* 시각적으로 안정 영역 안/밖 판정. A-안정 vs L-안정의 정의를 그림 한 장으로. |
| 2 | [`CE_7_4_02.ipynb`](CE_7_4_02.ipynb) | **Trapezoidal rule (= Crank–Nicolson = 1-stage Gauss IRK)** 를 *Newton 반복* 으로 구현. 비선형 stiff IVP $y'=-50(y-\cos t)$ (분석해 보유) 에서 $h=2^{-k},k=2..8$ 의 전역 오차. polyfit slope $\widehat p_{\text{trap}}\approx 2$, $\widehat p_{\text{BE}}\approx 1$ — 두 방법의 *경험 차수* 직접 산출. 안정 영역 도해로 Trap 이 A-안정이지만 $R(-\infty)=-1$ 이라 L-안정이 *아님* 을 확인. |
| 3 | [`CE_7_4_03.ipynb`](CE_7_4_03.ipynb) | **2-stage Gauss IRK** (order 4) 와 **Radau IIA** (order 3, L-안정) tableau 박제 + consistency 검증. $|R(z)|$ 를 음의 실축에서 — Gauss 는 $\to 1$, Radau IIA 는 $\to 0$ 인 *L-안정성의 시각적 정의*. *Very stiff* **Van der Pol $\mu=1000$** 에 implicit RK + 명시적 RK4 + SciPy `Radau` 비교: implicit 은 $h=0.5$ 로 limit cycle 깨끗, 명시적 RK4 는 $h\le 10^{-3}$ 가 강요됨. |

## 한 줄 정리
> **암시적 RK 는 *한 스텝의 비선형 시스템* 비용을 *안정 영역의 좌반평면 전체 확장* 으로 교환.
> A-안정성과 L-안정성의 차이가 *very stiff modes 의 진동 잔재* 를 가른다.**
> 다음 다이얼 — *한 스텝의 선형 시스템 비용* — 은 Ch 8 LU / SVD 의 영역.

## 사용 라이브러리
- NumPy (Butcher tableau 박제, `np.kron`/`np.linalg.solve` 로 stage 시스템 구성)
- SciPy (`solve_ivp(method='Radau')` 참조, `dense_output=True` 로 보간)
- Pandas (전역 오차표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (안정 영역 contour, 궤적 / phase portrait — 영문 라벨)
- `fractions.Fraction` (Radau IIA tableau 의 *유리수* 출력)

## 다음 Day 예고
**Day 31 — §8.1 LU 분해 (재방문)**: implicit RK 의 매 스텝마다 $\bigl(I - h(A\otimes J)\bigr)$ 라는
*sparse block 행렬* 의 인수분해가 필요. Crout / Doolittle / partial pivoting 의 *연산량* 과
*수치 안정성* 을 다시 본다. *implicit method 의 진짜 비용* 은 stage 수가 아니라 *행렬 인수분해의 cost*.

## 메모
* 이번 Day 부터는 `_meta/curriculum.md` 의 명시적 진도표를 넘어갔다 (커리큘럼은 Day 13 까지가 explicit, 이후는 "Ch 4–13 같은 호흡으로" 라고만 적혀 있음). 챕터 7 의 자연스러운 마지막 절로 §7.4 implicit RK 를 선택했다.
* 다음 Day 부터는 **Ch 8 — Linear Systems Revisited / LU & SVD** 로 진입.

---
