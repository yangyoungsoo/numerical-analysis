# Day 12 — §3.2 Newton's Method (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 3. Nonlinear Equations
> **절**: §3.2 Newton's Method — 도함수를 활용한 *이차 수렴* 의 위력과 한계

## 오늘의 큰 그림
Day 11 의 이분법은 *부호변화* 만으로 무조건 수렴하지만 *선형* 으로 느렸다.
오늘은 도함수 정보를 추가로 사용해 *한 스텝에 자릿수가 두 배씩* 늘어나는 **Newton 방법**
을 본다. 이 방법의 *황홀한 속도* 와 그 속도가 *깨질 때의 카오스적 거동* 을 함께 다룬다.

세 문제를 관통하는 메시지:

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **속도** | 차수 $p = 2$, 점근 상수 $C = f''(r)/(2 f'(r))$ — 자릿수 두 배 | 근 근처의 매끄러움 |
| **전역 거동** | basin of attraction 의 *fractal* 경계 — "가장 가까운 근으로" 직관 깨짐 | 임계점, 동역학 |
| **실패 모드** | 변곡점/저기울기에서 $|x_0|>x_0^\star$ 면 발산 | 출발점의 위치 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_3_2_01.ipynb`](CE_3_2_01.ipynb) | $f(x)=x^2-a$ 에 Newton 적용 → **Babylonian 공식** $x_{n+1}=\tfrac{1}{2}(x_n+a/x_n)$. $a\in\{2,3,10,100,0.5\}$ 에서 약 6 회 만에 $10^{-16}$ 도달. $\log e_{n+1}$ vs $\log e_n$ 회귀로 기울기 $\approx 2$ 직접 검증 — **이차 수렴**. |
| 2 | [`CE_3_2_02.ipynb`](CE_3_2_02.ipynb) | 복소 $p(z)=z^3-1$ 의 basin of attraction 을 $[-1.5,1.5]^2$ 격자에서 색칠. 세 영역이 *fractal Julia set* 을 공유. **30 % 이상의 점에서** Newton 은 *가장 가까운 근이 아닌* 다른 근으로 수렴. 원점 (임계점 $p'(z)=0$) 부근 zoom 으로 자기유사성 확인. |
| 3 | [`CE_3_2_03.ipynb`](CE_3_2_03.ipynb) | $f(x)=\arctan x$ 의 Newton 임계값 $x_0^\star\approx 1.3917$ 을 이분법으로 측정. 이론값 $\arctan x_0 = 2x_0/(1+x_0^2)$ 의 양의 해와 소수점 10 자리 일치. 추가로 $f(x)=x^{1/3}$ 에서 $x_{n+1}=-2 x_n$ 의 *구조적* 발산 확인. |

## 한 줄 정리
> **Newton 방법은 *이차 수렴* 이라는 조건부 마법이다.**
> *근 근처에서만* 자릿수가 두 배씩 늘어나며, *그 근처의 크기* 는 함수의
> 기하학(변곡점, 임계점)에 따라 임계값으로 결정된다. 그 임계값 바깥은
> *카오스* (basin 경계 fractal) 또는 *발산* (변곡점)이다.

## 사용 라이브러리
- NumPy (벡터화된 복소 격자 반복, `np.cbrt`, `np.arctan` 등)
- Pandas (`pd.set_option("display.float_format", ...)`)
- Matplotlib (`imshow` 로 basin 시각화, 영문 라벨)
- SciPy (`brentq` 로 임계값 이론치 검증)

## 다음 (Day 13)
**§3.3 Secant Method** — 도함수를 *유한차분* 으로 근사해, *한 스텝당 함수 평가 1 회*
만 쓰면서도 **superlinear** 수렴 차수 $\phi = (1+\sqrt 5)/2 \approx 1.618$ 을 얻는
방법을 본다. Newton 의 *속도* 와 이분법의 *견고함* 사이의 trade-off 를 본격적으로
다루기 시작한다.
