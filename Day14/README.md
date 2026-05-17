# Day 14 — §4.1 Polynomial Interpolation (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 4. Interpolation and Numerical Differentiation
> **절**: §4.1 Polynomial Interpolation — *Lagrange 형, Newton 분할차분, 그리고 Runge 의 경고*

## 오늘의 큰 그림
Day 13 까지 우리는 *한 변수 비선형 방정식* 의 근을 찾는 세 알고리즘을 다뤘다.
이제 챕터 4 로 넘어가 **데이터를 보간하는 다항식** 을 본다.

핵심 정리는 깔끔하다 — $n+1$ 개의 서로 다른 노드 위에서 차수 $\le n$ 의 보간
다항식은 *유일* 하게 존재한다. 그래서 *어떤 형식* 으로 쓰든 결과는 같다. 그러나
*어떻게 쓰느냐*, *어디에 노드를 두느냐* 가 알고리즘적 / 수치적 운명을 가른다.

세 문제를 관통하는 메시지:

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **유일성과 두 표현** | Lagrange 형 = Newton 형 (수치적으로도 일치) | $n+1$ 개의 서로 다른 노드 |
| **증분 비용** | Newton $O(N^2)$ vs Lagrange $O(N^3)$ | *nesting* 가능 여부 |
| **노드 선택** | 등간격은 Runge 로 *발산*, Chebyshev 는 *지수* 수렴 | $\|\omega_n\|_\infty$ |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_4_1_01.ipynb`](CE_4_1_01.ipynb) | Lagrange 형과 Newton 분할차분 형을 직접 구현. $\sin x$ on $[0, \pi]$ 에서 두 형이 dense grid 위에서 $\sim 10^{-14}$ 까지 *동일*. $n = 2, \dots, 13$ 에서 max error 가 $(n+1)!$ 분의 지수적 감쇠로 떨어지는 것을 확인. **두 형은 같은 함수의 다른 표기**라는 사실을 정량으로. |
| 2 | [`CE_4_1_02.ipynb`](CE_4_1_02.ipynb) | 노드를 *한 개씩 추가* 하는 시나리오에서 두 형의 누적 비용 측정. Lagrange 는 매번 *재구축* 하므로 $O(N^3)$, Newton 은 *증분 갱신* 으로 $O(N^2)$. log–log 그래프에서 기울기 3 vs 2 가 명확. *값* 은 매 단계 $10^{-13}$ 이내로 일치. |
| 3 | [`CE_4_1_03.ipynb`](CE_4_1_03.ipynb) | **Runge 함수** $f(x) = 1/(1+25 x^2)$ on $[-1, 1]$. 등간격 노드는 $n$ 을 키울수록 *끝단 진동* 이 폭발 (max error 가 증가). Chebyshev 노드 ($T_{n+1}$ 의 근) 는 같은 차수에서 *지수* 수렴. $\omega_n(x)$ 의 sup-norm 비교로 *이유* 까지 확인 — Chebyshev 의 $2^{-n}$ vs 등간격의 폭발. |

## 한 줄 정리
> **보간 다항식은 유일하지만, *어떤 형식* 으로 쓰느냐 와 *어디에 노드를 두느냐* 가
> 알고리즘 비용과 수치 안정성을 결정한다.** Newton 형은 *nesting*, Chebyshev 노드는
> *minimax* — 둘이 합쳐지면 보간이 진짜 실용 도구가 된다.

## 사용 라이브러리
- NumPy (벡터화 산술, `np.polyfit` 류는 *쓰지 않고* 직접 구현)
- Pandas (분할차분 표, 결과 비교용 DataFrame)
- Matplotlib (`semilogy`, `loglog`, 영문 라벨)
- (표준 `time.perf_counter` 로 wall-clock 시간 측정)

## 다음 (Day 15)
**§4.2 — Errors in Polynomial Interpolation**.  오늘 본 *노드 다항식*
$\omega_n(x)$ 의 거동을 정량화한다 — Lebesgue 상수 $\Lambda_n$, Chebyshev 의
*minimax* 정리, 그리고 *Bernstein 타원* 으로 본 수렴 반경.
보간 오차의 상계가 *어디에서 오는지* 정리/증명/실험으로 다진다.

> 본 노트북 시리즈는 `_meta/curriculum.md` 의 *향후 (Day 14+)* 예고에 따라
> 챕터 4 §4.1 부터 *같은 호흡* 으로 이어간다.
