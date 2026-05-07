# Day 04 — §1.1 (continued) Computer Exercises 10–12

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 1. Mathematical Preliminaries and Floating-Point Representation
> **절**: §1.1 Introduction (continued) — *Loss of Significance*

## 오늘의 큰 그림
어제(Day 03)는 *연산 순서*, *공식 형태*, *정밀도* 의 세 변수가 누적 round-off 에 미치는 영향을 봤다.
오늘(Day 04)은 그중 **공식 형태** 단 하나에 집중한다 — 같은 함수를 평가하는 두 가지 표현이 있을 때, 한쪽은 *거의 같은 두 수의 차*를 만들고 다른 쪽은 만들지 않는다. 차를 만드는 쪽은 cancellation이 일어나 자릿수가 사라진다.

세 문제 모두 동일한 절차를 따른다:

1. **불안정 표현** 을 정의 → cancellation 발생 영역 식별
2. **항등식 / 유리화 / 라이브러리 함수** 로 등가 재작성
3. 두 결과의 절대/상대오차를 같은 그림에 올려, 어디서 자릿수가 갈리는지를 시각적으로 본다
4. cancellation 증폭배율 $\kappa = (|a|+|b|)/|a-b|$ 와 이론적 한계 $\kappa\cdot\varepsilon_{\text{mach}}$ 를 측정한 상대오차와 비교

| 각도 | 예시 | 안정 대안 |
|---|---|---|
| 작은 $x$ | $(1-\cos x)/x^2$ | 배각 항등식 $2\sin^2(x/2)/x^2$ |
| 큰 $x$ | $\sqrt{x^2+1}-x$ | 유리화 $1/(\sqrt{x^2+1}+x)$ |
| 일반 패턴 | $e^x - 1$ (작은 $x$) | Taylor 합 / `expm1` |

## 문제 목록
| # | 파일 | 핵심 내용 |
|---|---|---|
| 10 | [`CE_1_1_10.ipynb`](CE_1_1_10.ipynb) | $(1-\cos x)/x^2$ — 분자에서 $1-\cos x$ 가 $x^2/2$ 만큼 작아져 $x \approx 10^{-8}$ 에서 자릿수 소실; 배각 항등식으로 분자를 *제곱* 으로 바꾸면 회복 |
| 11 | [`CE_1_1_11.ipynb`](CE_1_1_11.ipynb) | $\sqrt{x^2+1}-x$ — 큰 $x$ 에서 두 거의 같은 수의 차; cancellation 증폭배율이 $\sim 4x^2$ 로 커져 $x \sim 3.4 \times 10^7$ 에서 자릿수 0; 유리화로 분모의 *합* 으로 옮겨 안정화 |
| 12 | [`CE_1_1_12.ipynb`](CE_1_1_12.ipynb) | $e^x - 1$ (자기 함수) — $\kappa(x)\approx 2/|x|$, naive 의 상대오차가 $\kappa\cdot\varepsilon_{\rm mach}$ 의 기울기로 발산; Taylor 부분합과 `expm1` 두 가지 안정 대안 비교 |

## 한 줄 정리
> **두 거의 같은 수를 빼지 말라 — 합, 제곱, 또는 cancellation-aware 라이브러리 함수로 우회할 수 있다.**
> 같은 함수의 두 등가 표현 중 어느 쪽도 산술적으로 틀리지 않지만, 부동소수점에서의 자릿수 보존 능력은 전혀 다르다.

## 사용 라이브러리
- NumPy (`np.exp`, `np.expm1`, `np.cos`, `np.sin`, `np.sqrt`)
- Pandas, Matplotlib

## 다음 (Day 05)
§1.2 *Mathematical Preliminaries* 로 진입. Taylor 급수의 절단오차와 반올림오차의 trade-off, $\sin(10^{10})$ 같은 큰 인수에서의 *range reduction*, MVT 수치 검증을 다룬다.
