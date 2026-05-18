# Day 15 — §4.2 Errors in Polynomial Interpolation (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 4. Interpolation and Numerical Differentiation
> **절**: §4.2 Errors in Polynomial Interpolation — *노드 다항식 → Lebesgue 상수 → Bernstein 타원*

## 오늘의 큰 그림
Day 14 §4.1 에서 우리는 *어떤 노드를 고르느냐* 가 보간의 운명을 가른다는 사실을
Runge 현상으로 봤다. 오늘은 그 *왜* 를 세 개의 정량 도구로 분해한다.

1. **노드 다항식 $\omega_n(x)$** — Lagrange 잉여항이 직접 주는 *함수 × 노드* 의 곱.
   $\|\omega_n\|_\infty$ 를 누가 가장 작게 만드는가? — Chebyshev 의 minimax 정리가 답.
2. **Lebesgue 상수 $\Lambda_n$** — 보간 *연산자* 의 sup-norm. 최선 근사 와 보간의
   격차를 결정. 등간격은 지수 발산, Chebyshev 는 로그 성장 (Erdős 하계와 *상수까지*).
3. **Bernstein 타원** — *함수의 복소 해석적 영역* 이 정하는 수렴 반경.
   Runge 의 극점 $\pm i/5$ 가 결정하는 $\rho^* = (1 + \sqrt{26})/5$ 의 의미.

세 문제는 *같은 부등식* — $\|f - p_n\| \le \tfrac{M_{n+1}}{(n+1)!}\,\|\omega_n\|_\infty$
— 을 세 방향으로 깎아 들어간다.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **표준 오차 상계** | $\|\omega_n^{\rm equi}\| \sim n!\,(2/n)^{n+1}/4$ vs $\|\omega_n^{\rm cheb}\| = 2^{-n}$ | minimax 정리 |
| **near-best** | $\|f - L_n f\| \le (1+\Lambda_n)\|f - p^*_n\|$ | $\Lambda_n^{\rm equi} \sim 2^{n+1}/(en\log n)$ vs $\Lambda_n^{\rm cheb} \sim (2/\pi)\log n$ |
| **지수 수렴 비율** | $\|f - p_n^{\rm cheb}\| \sim \rho^{-n}$ | Bernstein 타원의 $\rho^*$ — 가장 가까운 특이점 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_4_2_01.ipynb`](CE_4_2_01.ipynb) | 노드 다항식 $\omega_n(x)$ 의 sup-norm 을 $n = 4, \dots, 28$ 에서 두 노드로 측정. 등간격은 표준 상계 $n!\,(2/n)^{n+1}/4$ 와 일치, Chebyshev 는 정확히 $2^{-n}$ — 두 비율이 $1.47^n$ 으로 지수 발산. $f(x) = \sin(\pi x)$ 에서 표준 오차 상계 $M_{n+1}\|\omega_n\|/(n+1)!$ 가 실제 max error 의 한 자릿수 이내 위에서 따라옴을 검증. |
| 2 | [`CE_4_2_02.ipynb`](CE_4_2_02.ipynb) | Lebesgue 상수 $\Lambda_n = \max_x \sum |\ell_i(x)|$ 를 $n = 2, \dots, 30$ 에서 측정. 등간격은 $2^{n+1}/(en\log n)$ 점근선과 같은 기울기로 *지수 발산* ($n=30$ 에서 $\sim 10^7$), Chebyshev 는 $(2/\pi)\log(n+1)$ 점근선과 *상수 차이* 로 나란히 ($n=30$ 에서 $\approx 2.7$). $\lambda_{20}(x)$ 시각화로 *어디에서* 차이가 나는지 — 등간격은 끝단 폭발, Chebyshev 는 균등 진동 — 직관 확립. |
| 3 | [`CE_4_2_03.ipynb`](CE_4_2_03.ipynb) | **Bernstein 타원**으로 Chebyshev 보간의 수렴 반경 예측. Runge 의 극점 $\pm i/5$ 로부터 $\rho^* = (1 + \sqrt{26})/5 \approx 1.2198$. $n = 4, 8, \dots, 80$ 에서 측정한 max error 에 $\log_{10}$ 선형회귀로 얻은 $\rho_{\rm fit}$ 가 이론값과 소수점 셋째 자리까지 일치. 복소평면에 critical Bernstein 타원과 Runge 극점을 그려 *왜 그 $\rho^*$ 인지* 시각화. |

## 한 줄 정리
> **보간 오차의 모든 비밀은 $\dfrac{M_{n+1}}{(n+1)!}\,\|\omega_n\|_\infty$ 라는 한 부등식에 있다.**
> 함수의 매끄러움 ($M_{n+1}/(n+1)!$, *Bernstein* 으로는 $\rho^*$) 과
> 노드의 분포 ($\|\omega_n\|_\infty$, *연산자* 로는 $\Lambda_n$) — 둘이 곱해져
> 수렴 속도와 안정성을 결정한다. **Chebyshev 노드** 는 두 양 모두에서 최적이다.

## 사용 라이브러리
- NumPy (`np.polyfit` 으로 $\log$–$n$ 회귀, 직접 구현한 Lagrange / Newton 평가)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`semilogy`, 복소평면 ellipse plot, 영문 라벨)
- (외부 데이터 없음 — 전부 코드 내에서 생성)

## 다음 (Day 16)
**§4.3 — Estimating Derivatives and Richardson Extrapolation**. Day 01 에서 본
forward difference 의 *U자형* 오차 곡선으로 다시 돌아간다. 이번에는 *Richardson 외삽* 으로
절단오차의 차수를 $O(h)$ → $O(h^2)$ → $O(h^4)$ 로 *체계적으로* 끌어올린다. 중심차분,
다단 Richardson, 그리고 자동으로 수렴까지 외삽을 반복하는 **Romberg 사다리** 의 첫 절반을
이번 절에서 다진다.

---

> **노트**: `_meta/curriculum.md` 는 Day 13 까지 명시되어 있고 "향후 (Day 14+)" 로만 예고가
> 되어 있다. Day 14 부터는 Day 14 README 의 *다음 (Day 15)* 예고 — §4.2 — 를 따라
> *같은 호흡* 으로 이어가고 있다. 진도가 완전히 끝나면 `_meta/curriculum.md` 를 확장하면 된다.
