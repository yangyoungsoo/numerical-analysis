# Day 27 — §7.1 Taylor Series Methods for IVPs (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 7. Initial-Value Problems
> **절**: §7.1 Taylor Series Methods — *Euler = Taylor 1, 그리고 차수 / 안정성 두 다이얼*

## 오늘의 큰 그림
Day 23–26 의 *spline* 으로 *함수* 를 표현하는 도구를 다 갖췄다면, 이제 챕터 7 에서는 *시간에 따라
변하는 해* 자체를 *점화식* 으로 만든다. 출발점은 가장 단순한 **Euler 법** — 곧 Taylor 1 차 — 이고,
같은 아이디어를 *차수* 의 다이얼로 늘리면 임의로 높은 수렴 차수 ($\mathcal O(h^p)$) 가 얻어진다.

그러나 *수렴 차수* 와 *수치 안정성* 은 *독립된 두 다이얼* 이다. 두 다이얼을 따로 잡는 시각이 오늘의 핵심.

세 문제를 관통하는 한 식 — **Taylor 점화식과 그 증폭 인자**:

$$
\boxed{\;
y_{n+1} \;=\; \sum_{j=0}^{p} \frac{h^j}{j!}\,y^{(j)}(t_n, y_n),
\qquad
\text{선형 시험식에서 증폭 인자 } R_p(z) = \sum_{j=0}^{p} \frac{z^j}{j!},
\quad z = h\lambda.
\;}
$$

— *합의 길이 $p$* 가 *전역 수렴 차수* $\mathcal O(h^p)$ 를, *$|R_p(z)| \le 1$* 가 *절대 안정 영역* 을 결정한다.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **수렴 차수** | $\|y_N - y(T)\| = \mathcal O(h^p)$, 인접 비율 $\to 2^p$ | Taylor 합의 길이 $p$ |
| **Round-off 한계** | $h^\star \sim \varepsilon_{\text{mach}}^{1/(p+1)}$ | LTE 와 누적 round-off 의 균형 |
| **절대 안정 영역** | $\{z : |R_p(z)| \le 1\}$, 음 실축 한계 $\alpha_p^\star$ | $R_p$ 의 모양 (차수와 무관하게 *유한* 한 영역) |
| **Stiffness 비용** | $h \le \alpha_p^\star / |\lambda|$ 강제 | $\|\lambda\|$ (자연 시간 척도) |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_7_1_01.ipynb`](CE_7_1_01.ipynb) | $y' = -y,\, y(0)=1$ 에 Euler 법 적용, $h = 2^{-k}\, (k=2..16)$ 로 *전역* 오차의 U 자형 측정. 인접 비율 $\approx 2$, 경험 기울기 $+1.000$ — Taylor 1 차의 $\mathcal O(h)$ 가 표에서 깔끔하게. $h \le 10^{-5}$ 영역에서 round-off 가 절단오차를 *역전*, 최적 $h^\star \approx \sqrt{\varepsilon} \sim 10^{-8}$ 부근에서 오차 최소. 절단/반올림 두 참조선을 같이 그려 U 자의 두 다리를 시각적으로 분리. |
| 2 | [`CE_7_1_02.ipynb`](CE_7_1_02.ipynb) | $y' = t - y^2,\, y(0)=1$ 에 Taylor $p = 1, 2, 3, 4$ 적용. $y'', y''', y^{(4)}$ 를 *연쇄 법칙* 으로 손유도 ($y'' = 1 - 2yy'$ 등) 후 직접 계산. 인접 비율 표 — $p$ 별로 정확히 $2, 4, 8, 16$. SciPy `DOP853` (rtol=1e-13) 의 기준해와 비교. *같은 스텝수* 에서 $p$ 가 1 → 4 로 가면 정확도가 약 *5 자릿수* 향상. $p = 4$ 의 가장 작은 $h$ 에서는 round-off 가 $10^{-13}$ 부근에서 *평탄화*. |
| 3 | [`CE_7_1_03.ipynb`](CE_7_1_03.ipynb) | (a) Dahlquist 시험식 $y' = \lambda y$ 의 증폭 인자 $R_p(z) = \sum z^j/j!$ 로 음 실축 한계 직접 계산 ($\alpha_1^\star = \alpha_2^\star = 2.000$, $\alpha_3^\star = 2.513$, $\alpha_4^\star = 2.785$). (b) $\lambda = -10$ 에서 $h = 0.05 \to 0.25$ 로 *안정 경계* 를 가로지르며 적분 — $h \le 0.20$ 에서는 지수 감쇠를 따라가지만 $h \ge 0.21$ 에서는 *부호 반전 + 진폭 폭발*. (c) 복소 평면 안정 영역 등고선 ($p = 1..4$) — 차수가 올라도 음 실축 방향은 거의 안 늘어남, 허수축 방향은 확장. (d) $\lambda = -10, -50, -100$ 에서 *최소 스텝 수* 표 — $p$ 를 올려도 stiffness 비용은 *동급*. *implicit* 방법의 필요성 명확화. |

## 한 줄 정리
> **Taylor 법의 두 다이얼: *합의 길이 $p$* = 수렴 차수 $\mathcal O(h^p)$, *$|R_p|$* = 절대 안정 영역.**
> 차수만 올려도 stiffness 문제는 풀리지 않는다 — *implicit* 방법이 그 답이며, 그 전에 §7.2 Runge–Kutta
> 가 *해석적 미분 없이 같은 차수* 를 얻는 길을 연다.

## 사용 라이브러리
- NumPy (Euler / Taylor 점화식 직접 구현, `polyfit` 으로 경험 기울기, 복소 평면 mesh)
- SciPy (`solve_ivp(method='DOP853', rtol=1e-13)` 로 기준해, `brentq` 로 안정 한계 정확 계산)
- Pandas (`pd.set_option("display.float_format", ...)` + pivot 으로 수렴 비율 표)
- Matplotlib (`loglog` U 자형 그림, $|R_p|$ 등고선, 영문 라벨 — 한글 폰트 의존 회피)
- 외부 데이터 없음 — 모든 함수 / 도함수 / 평가 격자는 코드 내에서 생성

## 다음 (Day 28)
**§7.2 — Runge–Kutta Methods.** Taylor $p$ 의 *해석적 미분* 부담을 *$f$ 의 다단 평가* 로 바꾼다.
RK2 (Heun, midpoint), RK4 (고전) 의 수렴 차수 + 한 스텝의 비용 측정. 안정 영역은 Taylor 의 그것과
정확히 같지만 (다항식 $R(z)$ 가 같음), 해석적 미분이 필요 없어 *비선형* / *시스템* 문제로 즉시 확장.
이후 §7.3 의 *적응형 RK* (RK45, Fehlberg, Dormand–Prince) 에서 *오차 자가 진단* 으로 $h$ 동적 조절.

> *Note*: `_meta/curriculum.md` 의 명시적 항목은 Day 13 까지이며, 그 이후는 책의 흐름
> (Ch 4 보간 → Ch 5 적분 → Ch 6 spline → Ch 7 ODE → ...) 을 따라 자동 진행 중이다.
> Day 26 README 에서 예고했던 §6.5 "Tensor-product B-spline / NURBS" 는 본 7th edition Ch 6 의
> 정식 절이 아니어서 *생략* 하고 자연스럽게 Ch 7 의 첫 절 (§7.1) 로 진입했다.
> 사용자 커리큘럼 보강이 필요하면 `_meta/curriculum.md` 에 Day 14+ 항목을 추가하면 됨.
