# Day 29 — §7.3 Adaptive Runge–Kutta Methods (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 7. Initial-Value Problems
> **절**: §7.3 Adaptive Runge–Kutta Methods — *embedded pair 로 한 격자 위에서 두 차수, 자동 step 조절*

## 오늘의 큰 그림
Day 28 의 *고정 스텝 RK4* 는 *차수 다이얼* 을 풀었지만, *어떤 $h$ 를 써야 하는가* 는 사용자의 몫이었다.
오늘 §7.3 에서는 *한 격자 위에서 두 차수의 추정치* 를 같은 stage 집합으로 *공짜로* 얻는
**embedded pair** 를 본다. 두 추정치의 차이가 곧 *국소 오차 추정자*, 그리고 그것을 *피드백 신호* 로
다음 스텝의 $h$ 를 결정 — *adaptive RK*.

세 문제를 관통하는 *한 식* — **embedded pair 의 두 가지 역할**:

$$
\boxed{\;
\underbrace{y^{(p)}_{n+1}, \; y^{(\tilde p)}_{n+1} \;=\; y_n + h\sum_i b_i k_i,\; y_n + h\sum_i \tilde b_i k_i}_{\text{같은 } \{k_i\}, \text{ 두 가중치}}
\quad\Longrightarrow\quad
\underbrace{\widehat\tau_n = h\textstyle\sum_i (\tilde b_i - b_i) k_i}_{\text{free local-error estimator}}
\quad\Longrightarrow\quad
\underbrace{h_{n+1} = h_n \cdot \sigma\,(\tau_{\text{tol}}/\widehat\tau_n)^{1/(p+1)}}_{\text{step-size feedback}}.
\;}
$$

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **Embedded pair** | $\|y^{(\tilde p)} - y^{(p)}\| = \mathcal O(h^{p+1})$ — $y^{(p)}$ 의 local error 의 비편향 추정자 | $(b - \tilde b)$ 벡터, 차수 조건 |
| **Step controller** | I: 단순 비례 / **PI**: 이력까지 — 진동 감소 (Gustafsson 1991) | $(\sigma, \alpha, \beta, f_{\min}, f_{\max})$ |
| **FSAL** | $a_{s,j} = b_j^{(\tilde p)}$ — 마지막 stage = 다음 스텝의 첫 stage | DOPRI5(4) 의 핵심 trick |
| **Local extrapolation** | 보고하는 해는 *higher-order* 추정치 ($y^{(\tilde p)}$), 다른 하나는 *estimator 전용* | SciPy `RK45` 의 default |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_7_3_01.ipynb`](CE_7_3_01.ipynb) | **RKF45 (Fehlberg 1969)** 의 정확한 Butcher tableau (rational) 박제. $y'=-y, y(0)=1$ 에 *고정 스텝* 으로 6 stages = 4차 + 5차 update 를 동시에. $h=2^{-k}, k=2..12$ 의 전역 오차표 — 인접 비율 $\to 16, 32$, 경험 기울기 $\widehat p_4 \approx 4, \widehat p_5 \approx 5$. *단일 스텝* 에서 $\widehat\tau = \|y^{(5)} - y^{(4)}\|$ 가 *국소* 4차 오차와 $h^5$ 동급 — *free estimator* 의 비편향성 시각적 증명. |
| 2 | [`CE_7_3_02.ipynb`](CE_7_3_02.ipynb) | CE 7.3-1 의 $\widehat\tau$ 를 **PI controller** (Gustafsson, $\alpha=0.7/(p+1), \beta=0.4/(p+1)$) 의 측정값으로. *Sharp tanh transition* IVP $y' = 10(1-y^2), y(0) = \tanh(-10), t\in[0,2]$, 해석해 $y=\tanh(10(t-1))$. tolerance sweep $\{10^{-4}, 10^{-6}, 10^{-8}\}$ → $f$-evals 이 $\propto \tau_{\text{tol}}^{-1/(p+1)}$. **$h(t)$ 그림** 이 핵심: 전이 구간 ($\|t-1\| \lesssim 0.1$) 에서 $h$ 가 *수십~수백 배 작게* (max/min $\approx 140$). *국소 적응성* 의 시각적 증명. |
| 3 | [`CE_7_3_03.ipynb`](CE_7_3_03.ipynb) | **Dormand–Prince 5(4) (DOPRI5)** — SciPy `RK45` 의 내부. 7 stages 지만 *FSAL* ($a_{7,j} = b_j^{(5)}$) 로 *accepted step* 당 $f$-eval 6 회. *Local extrapolation* — 보고하는 해는 5차. $y'=t-y^2, y(0)=1, t\in[0,2]$ 의 비선형 IVP 에 RKF45 / 수제 DOPRI5 / SciPy `RK45` 를 같이. **Work–precision diagram** — DOPRI5 곡선이 RKF45 보다 일관되게 *좌하단* (같은 정확도에 $1.2$–$1.6\times$ 적은 평가). FSAL 감사로 *effective evals/step* $\approx 6.0$ (이론치) 확인. |

## 한 줄 정리
> **Embedded pair 의 가중치 차 $\tilde b - b$ 는 *free local-error estimator*, 그것을 PI controller 의
> 측정값으로 받으면 *step 크기 자동 결정*. FSAL 까지 더하면 7-stage 가 effective 6 evals/step
> 의 *SciPy `RK45` default*.**
> 다음 다이얼 — *비용* — 은 §7.4 implicit RK 의 영역.

## 사용 라이브러리
- NumPy (RKF45/DOPRI5 Butcher tableau 직접 박제, `polyfit` 으로 차수 측정)
- SciPy (`solve_ivp(method='DOP853', rtol=1e-13)` 기준해, `solve_ivp(method='RK45')` 와 수제 DOPRI5 비교)
- Pandas (tolerance vs $f$-evals 표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (work–precision diagram, step-size $h(t)$, local-error 검증 — 영문 라벨)

## 다음 Day 예고
**Day 30 — §7.4 Implicit Runge–Kutta (stiff problems)**: Gauss / Radau IIA / SDIRK. 명시적 RK 의
*안정 영역* (Day 27–28) 이 좌반평면 *전체* 로 확장 — stiffness 의 *진정한* 해법. 대가는
*매 스텝 비선형 시스템* 풀이.

---
