# Day 10 — §2.3 Tridiagonal and Banded Systems (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 2. Linear Systems
> **절**: §2.3 Tridiagonal and Banded Systems — exploit structure to drop $\mathcal{O}(n^3) \to \mathcal{O}(n)$

## 오늘의 큰 그림
일반 가우스 소거는 $\mathcal{O}(n^3)$ 비용. 하지만 행렬이 *대부분 0* 이고 띠 폭이 좁다면, 0 을 건드리지
않는 소거가 가능하다. **3중대각**(bandwidth 1) → **5중대각**(bandwidth 2) 로 확장하면서, 비용이
어떻게 *차원으로* 떨어지는지, 그 결과가 *실제 PDE* 풀이에서 어떻게 의미를 가지는지 본다.

세 문제를 관통하는 메시지:

| 관점 | 핵심 결과 | 무엇이 결정하나 |
|---|---|---|
| **구조의 가치** | Thomas: 비용 $\frac{2}{3}n^3 \to 8n$, 메모리 $n^2 \to 3n$ | bandwidth |
| **응용** | 1D Poisson $-u''=f$ 가 정확히 3중대각 SPD | 중심차분의 stencil |
| **일반화** | bandwidth 2 (penta) 도 $\mathcal{O}(n)$ — 빔 굽힘 $u^{(4)}=f$ | stencil 폭 |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 1 | [`CE_2_3_01.ipynb`](CE_2_3_01.ipynb) | `thomas(a, b, c, d)` 직접 구현. 무작위 diagonally dominant 시스템에서 잔차/오차가 NumPy 의 dense solve 와 같음을 확인. $n=2^4 \ldots 2^{12}$ 에서 wall-clock 측정 — Thomas 기울기 1, dense 기울기 3, $n=4096$ 에서 **수백 배 ~ 천 배** 속도 차이. |
| 2 | [`CE_2_3_02.ipynb`](CE_2_3_02.ipynb) | $-u''=f$ on $(0,1)$, 디리클레 BC. 2차 중심차분으로 3중대각 SPD 시스템 구성, Thomas 로 풀이. 제조해 $u^* = \sin(\pi x) + \alpha(1-x) + \beta x$ 와의 max-norm 오차가 $h$ 절반마다 4 배 감소 — 경험적 차수 ≈ **2.0**, 이론값 $\mathcal{O}(h^2)$. |
| 3 | [`CE_2_3_03.ipynb`](CE_2_3_03.ipynb) | `penta(e, a, b, c, g, d)` 5중대각 솔버. 빔 굽힘 $u^{(4)}=24$, clamped BC 에 적용, 해석해 $x^2(1-x)^2$ 와 일치 확인. dense 와 wall-clock 비교 — 기울기 1 vs 3 이 그대로 유지됨. |

## 한 줄 정리
> **구조를 알면 비용이 차원으로 떨어진다.**
> 가우스 소거의 $\mathcal{O}(n^3)$ 은 *완전 dense* 라는 가정에서 나오는 비용이고, 실제 응용에서 만나는
> 행렬들은 거의 모두 sparse — 3중/5중대각이 가장 단순한 예. Thomas 와 그 일반화는 *수치 PDE* 의
> 내부 루프로서 거의 매번 등장한다.

## 사용 라이브러리
- NumPy (`np.linalg.solve`, `np.linalg.norm`, `np.diag` 등)
- Pandas (`pd.DataFrame`)
- Matplotlib (영문 라벨)

## 다음 (Day 11)
**§3.1 Bisection Method** — 이제 챕터 3 비선형 방정식으로 옮겨간다. 가장 단순한 root finding
알고리즘 *이분법* 이 가지는 매력은 *수렴 보장*. 단, 수렴 속도가 선형이라는 한계.
$\cos x = x$, $x e^x = 1$ 같은 클래식한 방정식들로 수렴 차수를 직접 측정하고, 다중근/같은 부호
경계 같은 *실패 모드* 도 만들어 본다.
