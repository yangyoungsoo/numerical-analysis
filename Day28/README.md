# Day 28 — §7.2 Runge–Kutta Methods (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 7. Initial-Value Problems
> **절**: §7.2 Runge–Kutta Methods — *도함수 없는 Taylor, 그러나 안정성은 그대로 유한*

## 오늘의 큰 그림
Day 27 의 *Taylor 법* 은 차수 $p$ 의 다이얼을 *손유도 도함수* 로 돌렸다. 오늘 §7.2 에서는 같은
$\mathcal{O}(h^p)$ 정확도를 **$f$ 의 함수 평가만으로** 얻어내는 *Runge–Kutta* 의 trick 을 본다.
2-stage 로 2차, 4-stage 로 4차 — 차수 다이얼은 *Butcher tableau 의 차수 조건* 으로 풀린다.

그러나 *수치 안정성* 의 다이얼은 그대로다. 명시적 RK 의 증폭 인자

$$
R_p(z) = \sum_{j=0}^{p} \frac{z^j}{j!}, \qquad z = h\lambda
$$

는 Taylor 와 *동일* — 따라서 *절대 안정 영역도 동일* (음 실축 한계 $\alpha_4^\star \approx 2.785$).
*도함수 노동만 사라지고, stiffness 비용은 그대로* 라는 점이 §7.2 의 핵심 *한계*.

세 문제를 관통하는 한 식 — **RK $p$ 차의 두 얼굴**:

$$
\boxed{\;
\underbrace{y_{n+1} = y_n + h\sum_{i=1}^{s} b_i\,k_i}_{\text{order } p \;:\; \text{Butcher 조건}}
\quad\Longleftrightarrow\quad
\underbrace{|R_p(h\lambda)| \le 1}_{\text{abs. stability: } \alpha_p^\star/|\lambda|}.
\;}
$$

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **차수 다이얼** | $\|y_N - y(T)\| = \mathcal{O}(h^p)$, 인접 비율 $\to 2^p$ | Butcher tableau (stages, weights) |
| **RK 가족 자유도** | 같은 $p$ 에서 여러 tableau (Heun, Midpoint, …) | order 조건 $<$ 미지수 — 1+-매개변수 가족 |
| **도함수 vs 평가** | Taylor $p$ ↔ RK $s = p$ 등가 ($p \le 4$) | 사람-노동 ↔ 컴퓨터-노동 trade-off |
| **절대 안정 영역** | $\{z : |R_p(z)| \le 1\}$ — Taylor 와 동일 | $R_p$ 의 모양 (차수와 무관하게 *유한*) |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_7_2_01.ipynb`](CE_7_2_01.ipynb) | $y' = -y, y(0)=1$ 에 **Heun (explicit trapezoidal)** 과 **Midpoint** RK2 를 동시 적용. $h = 2^{-k}, k=2..16$ 의 전역 오차표 — 둘 다 인접 비율 $\to 4$, 경험 기울기 $\approx +2.01$. 두 방식은 *같은 차수의 형제* 임을 (상수만 다름) log–log 그림으로 시각화. *RK2 의 1-매개변수 가족* 이라는 사실을 order 조건 $b_1+b_2=1, b_2 c_2 = 1/2, b_2 a_{21} = 1/2$ 로 유도. |
| 2 | [`CE_7_2_02.ipynb`](CE_7_2_02.ipynb) | $y' = t - y^2, y(0)=1$ 의 비선형 IVP 에 **classical RK4** 와 Taylor $p=1..4$ 를 같이 적용. SciPy `DOP853` (rtol=1e-13) 기준해 대비 *전역 오차* 측정. RK4 의 인접 비율 $\to 16$ — $\mathcal{O}(h^4)$ 확인. **두 축으로 그림** — *step size 축* vs *work 축 (steps × evals/step)*. 같은 일량에서 RK4 와 Taylor 4 가 동급이며, 차이는 *구현 비용* (RK4 는 $f$ 만 알면 됨). |
| 3 | [`CE_7_2_03.ipynb`](CE_7_2_03.ipynb) | (a) $R_p(z) = \sum z^j/j!$ 로부터 $\alpha_p^\star$ 를 `brentq` 로 정확히 계산: $\alpha_1^\star=\alpha_2^\star=2.000$, $\alpha_3^\star\approx 2.513$, $\alpha_4^\star\approx 2.785$. (b) $\lambda=-10$ 에서 $h$ 를 $h^\star \approx 0.2785$ 의 양옆으로 — $h<h^\star$ 에서는 지수 감쇠, $h>h^\star$ 에서는 *부호 반전 + 진폭 폭발*. (c) 복소 평면 안정 영역 등고선 ($p=1..4$) — 차수가 올라도 음 실축 한계는 거의 안 늘어남, 허수축 방향만 확장. (d) $\lambda = -10, -50, -100, -1000$ 의 *최소 스텝 수* 표 — stiffness 비용은 RK4 의 차수와 *독립*. *implicit* 의 필요성. |

## 한 줄 정리
> **RK $p$ 차는 도함수 없는 Taylor $p$ 차 — *차수 다이얼* 은 풀렸지만 *안정성 다이얼* ($\alpha_p^\star$) 은 그대로 유한.**
> Stiffness 문제에서 차수만 올려서는 답이 안 나온다 — *implicit* 으로 가야 좌반평면 전체가 안정 영역이 된다.

## 사용 라이브러리
- NumPy (RK2/RK4/Taylor 직접 구현, `polyval`, `polyfit`, 복소 평면 mesh)
- SciPy (`solve_ivp(method='DOP853', rtol=1e-13)` 로 비선형 IVP 기준해, `brentq` 로 $\alpha_p^\star$ 정확 계산)
- Pandas (인접 비율 표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (log–log 오차곡선, 안정 영역 등고선, 영문 라벨 — 한글 폰트 의존 회피)

## 다음 Day 예고
**Day 29 — §7.3 Adaptive Runge–Kutta (embedded methods)**: RK45 / Cash–Karp / Dormand–Prince
의 *embedded pair* 로 정확도 *자동조절*. 같은 격자에서 *두 차수* 의 추정치를 동시에 계산,
local error 추정 → 다음 step 크기 *자동 결정*. SciPy `RK45` 의 내부를 손으로 재현.

---
*Note*: `_meta/curriculum.md` 의 명시적 Day-by-Day 항목은 Day 13 까지로 끝나며, Day 14 이후는
*"Ch 4 보간, Ch 5 적분, Ch 6 spline, Ch 7 ODE, …"* 의 큰 흐름만 표시되어 있다. 본 Day 28 은
Day 27 (§7.1 Taylor) 의 자연스러운 연속으로 §7.2 를 선택했다 — 챕터 7 의 흐름이 Taylor → RK → 적응형 →
multi-step → implicit 으로 이어지는 표준 순서.
