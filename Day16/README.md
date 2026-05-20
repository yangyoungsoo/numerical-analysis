# Day 16 — §4.3 Estimating Derivatives and Richardson Extrapolation (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 4. Interpolation and Numerical Differentiation
> **절**: §4.3 Estimating Derivatives and Richardson Extrapolation — *수치미분 → 짝수 차수 외삽 → 적응형 사다리*

## 오늘의 큰 그림
Day 01 에서 우리는 전진차분의 *U자형 오차 곡선* — 절단오차와 round-off 의 trade-off —
을 처음으로 봤다. Day 14–15 에서 보간을 다지고, 오늘 *§4.3* 에서 그 자리로 돌아온다.
이번에는 *대칭성* 과 *외삽* 이라는 두 도구로 같은 부동소수점 정밀도 위에서 *몇 자릿수
더* 정확한 미분을 *체계적으로* 만든다.

세 문제를 관통하는 정리는 하나의 부등식

$$
|D_h f(x) - f'(x)| \;\le\; \underbrace{C_{\rm trunc}\,h^p}_{\text{절단오차 (decreasing)}}
                              + \underbrace{C_{\rm round}\,\frac{\varepsilon}{h}}_{\text{round-off (increasing)}}
$$

— 의 *모양* 을 결정하는 두 양, 차수 $p$ 와 상수 $C$ 를 어떻게 만지느냐 다.

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **대칭성 이득** | 전진→중심으로 $p$ 를 $1 \to 2$, 최적 오차를 $\varepsilon^{1/2} \to \varepsilon^{2/3}$ | 짝수 차수 항만 남기는 대칭 구성 |
| **체계적 외삽** | Richardson 한 단계마다 $p \to p+2$ | $h$ 와 $h/2$ 의 선형 결합 |
| **적응형 정지** | 사용자가 tolerance 만 주면 *최소 $f$ 평가* 로 정확도 달성 | 대각 수렴 판정 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_4_3_01.ipynb`](CE_4_3_01.ipynb) | $f(x)=\sin x$, $x=1.0$ 에서 전진차분과 중심차분의 오차를 $h=2^{-1}\dots 2^{-52}$ 에 걸쳐 비교. log–log 그래프의 절단오차 쪽 기울기가 $-1$ vs $-2$, U자 바닥은 $h^*\approx\sqrt{\varepsilon}\approx 10^{-8}$ vs $h^*\approx\varepsilon^{1/3}\approx 6\times 10^{-6}$, 달성 가능한 최소 오차는 $10^{-8}$ vs $10^{-11}$ — *세 자릿수 이득*. 이론 점근선과 측정 최소점을 한 그래프에 겹쳐 검증. |
| 2 | [`CE_4_3_02.ipynb`](CE_4_3_02.ipynb) | $f(x)=e^x$, $x=1$. 중심차분 $D(h)$ 의 짝수 차수 전개를 이용해 Richardson 점화식 $D^{(k)}(h)=(4^k D^{(k-1)}(h/2)-D^{(k-1)}(h))/(4^k-1)$ 로 사다리 5 column 채움. 각 column 의 log–log 기울기가 차례로 $2,4,6,8,10$ 으로 가팔라지는 모습 확인. *최선 오차* 는 $10^{-11}\to 10^{-14}$ 까지 *세 자릿수* 개선되다가 round-off 벽에서 정체. |
| 3 | [`CE_4_3_03.ipynb`](CE_4_3_03.ipynb) | 적응형 `richardson_deriv(f, x, h0, tol, kmax)` 직접 구현. 사다리 행 단위 갱신 + *대각 수렴* 정지 기준. $f(x)=\tan x$, $x=1$, 참값 $\sec^2 1$. tolerance $10^{-3}\dots 10^{-12}$ 스윕으로 (행 수, $f$ 평가 수, 실제 오차) 표 비교. 같은 tolerance 에서 naive 중심차분이 *어떤 $h$ 를 골라도 도달 못하는* $10^{-14}$ 자리까지 외삽이 뚫고 들어감을 시각화. |

## 한 줄 정리
> **수치미분의 정확도는 $h$ 를 작게 만든다고 좋아지지 않는다 — *어떤 차수의 절단오차* 와
> *어떤 외삽* 을 쓰느냐가 결정한다.** 대칭이 차수를 한 번 올리고, Richardson 이 *몇 번 더*
> 올리고, 적응형 정지가 그 비용을 최소화한다. 세 도구가 합쳐지면 부동소수점 정밀도 벽
> 바로 *앞* 까지 자동으로 도달할 수 있다.

## 사용 라이브러리
- NumPy (벡터화 산술, `np.finfo(float).eps`)
- Pandas (`pd.set_option("display.float_format", ...)` 으로 과학적 표기)
- Matplotlib (`loglog`, `semilogy`, 영문 라벨)
- `math.factorial` (NumPy 2.x 에서 `np.math` 가 제거되었으므로 표준 모듈 사용)
- (외부 데이터 없음 — 전부 코드 내에서 생성)

## 다음 (Day 17)
**§4.4 — Richardson Extrapolation Applied to the Integral**.  오늘의 *짝수 차수 소거*
트릭은 사다리꼴 적분에 그대로 옮겨져 **Romberg 사다리** 가 된다. $T(h) \to S(h) \to B(h) \to \cdots$
— 사다리꼴이 Simpson 으로, Simpson 이 Boole 로 *공짜로* 승급된다. 적응형 정지 기준은
오늘의 `richardson_deriv` 와 형태가 *동일* 하다. 챕터 4 마지막 절을 마무리하고 챕터 5
**Numerical Integration** 으로 자연스럽게 넘어간다.

---

> **노트**: `_meta/curriculum.md` 는 Day 13 까지만 명시. Day 14 부터는 직전 Day README 의
> *다음* 예고에 따라 *같은 호흡* 으로 이어가는 중. 진도가 끝나면 `_meta/curriculum.md` 를
> 확장하면 된다.
