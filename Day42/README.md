# Day 42 — §12.1 Parabolic Problems: 양함수(조건부) · 음함수(무조건) · Crank–Nicolson(2차) (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 12. Partial Differential Equations
> **절**: §12.1 Parabolic Problems (1D Heat / Diffusion Equation)

## 오늘의 큰 그림
Day 40–41 의 BVP는 *공간만* 있는 정상 문제였다. §12.1 부터는 **시간 $t$ 가 더해진 포물형 PDE**,
1차원 열방정식 $u_t=\alpha u_{xx}$ 로 들어선다. Day 41 에서 익힌 *격자+삼중대각* 사고가 그대로
이어지되, 이제 **시간을 어떻게 전진시키느냐(time-stepping)** 가 새 변수다. 세 문제는 시간차분을
바꿔가며 같은 방정식을 푼다 — 그리고 **안정성과 정확도의 교환**을 한자리에서 드러낸다.

세 문제를 관통하는 *한 식* — **공간 중심차분 + 세 가지 시간차분**:

$$
\boxed{\;
\frac{u_i^{n+1}-u_i^{n}}{\Delta t}
=\frac{\alpha}{\Delta x^2}\Big[(1-\theta)\,\delta_x^2 u_i^{n}+\theta\,\delta_x^2 u_i^{n+1}\Big],
\qquad
\theta=0:\text{FTCS},\;\; \theta=1:\text{BTCS},\;\; \theta=\tfrac12:\text{CN}
\;}
$$

$\theta=0$(양함수)은 명시적이지만 $r=\alpha\Delta t/\Delta x^2\le\tfrac12$ 에서만 안정(문제 1),
$\theta=1$(음함수)은 삼중대각 한 번의 대가로 무조건 안정(문제 2), $\theta=\tfrac12$(Crank–Nicolson)은
무조건 안정에 시간 2차까지 얻는다(문제 3).

| 문제 | 시간차분 | 핵심 결과 |
|---|---|---|
| **1** | 양함수 FTCS ($\theta=0$) | 명시적·간단, **조건부 안정** $r\le\tfrac12$, 시간 1차 |
| **2** | 음함수 BTCS ($\theta=1$) | 매 스텝 삼중대각 `solve`, **무조건 안정**, 시간 1차 |
| **3** | Crank–Nicolson ($\theta=\tfrac12$) | **무조건 안정 + 시간 2차**, 세 방법 수렴차수 비교 |

## 문제 목록

| # | 파일 | 주제 | 비교/시각화 | 한 줄 결론 |
|---|---|---|---|---|
| 1 | [`CE_12_1_01.ipynb`](CE_12_1_01.ipynb) | 양함수 FTCS · 안정 한계 | 해 감쇠, $r=1/2$ 임계 전후 (semilog) | $r\le\tfrac12$ 에서만 안정, 넘으면 톱니 폭발 |
| 2 | [`CE_12_1_02.ipynb`](CE_12_1_02.ipynb) | 음함수 BTCS · 삼중대각 | 큰 $r$ 안정 해, FTCS vs BTCS 대비 | 삼중대각 한 번으로 무조건 안정, 큰 $\Delta t$ 허용 |
| 3 | [`CE_12_1_03.ipynb`](CE_12_1_03.ipynb) | Crank–Nicolson · 수렴차수 | 해 프로파일, $\Delta t$-수렴 log-log | CN 시간 2차(slope 2) vs FTCS/BTCS 1차 |

## 수치 하이라이트 (실행 결과)
- **문제 1 (양함수·조건부 안정)**: $\alpha=1$, $M=40$. 깨끗한 IC + $r=0.5$ 는 정확해
  $e^{-\pi^2 t}\sin\pi x$ 와 최대오차 $\sim6\times10^{-4}$ 로 일치. IC 에 $10^{-3}$ 고주파 잡음을 심으면
  $r=0.50$ 까지는 안정(`max|u|`$\approx0.61$)이나, **$r=0.55\to6.4\times10^{7}$**, $r=0.60\to6.1\times10^{15}$ 로
  폭주 — 임계 $r=\tfrac12$ 가 von Neumann 분석 그대로 재현.
- **문제 2 (음함수·무조건 안정)**: $r=0.5,5,50$ 어디서도 `max|u|` 유계. $r{=}0.5\to50$ 에서
  시간스텝이 **$160\to2$** 로 급감(큰 $\Delta t$ 허용). 단 $r$ 가 커지면 시간 1차 오차가
  $6.2\times10^{-4}\to2.6\times10^{-2}$ 로 증가 — **안정 $\ne$ 정확**. 같은 $r=2$ 에서 FTCS 는 폭발,
  BTCS 는 정확해에 부드럽게 수렴.
- **문제 3 (Crank–Nicolson·시간 2차)**: 공간오차를 상쇄하는 *자기-기준해* 비교로 순수 시간차수를
  측정 — **CN $=2.001$**, BTCS $=1.009$, FTCS $=1.031$ (이론 2 / 1 / 1 과 일치). CN 은 큰 $r$ 에서도
  무조건 안정하며 가장 적은 스텝으로 목표 정확도 달성.
- **공통**: 세 방법 모두 "공간 중심차분 + 시간차분 $\theta$" 라는 한 골격($\theta$-method)을 공유하되,
  $\theta\ge\tfrac12$ 부터 무조건 안정해지고 $\theta=\tfrac12$ 에서만 시간 2차가 된다.

## 한 줄 정리
> 포물형 PDE 의 시간전진은 안정성과 정확도의 교환이다 — 양함수는 $r\le\tfrac12$ 의 족쇄,
> 음함수는 삼중대각 한 번으로 무조건 안정, 그리고 Crank–Nicolson 은 무조건 안정에 시간 2차까지
> 얻어 §12.1 의 표준이 된다.

## 사용 라이브러리
- NumPy (FTCS 벡터화 갱신, 삼중대각 조립·`linalg.solve`, von Neumann 증폭인자, log-log 회귀 `polyfit`)
- Pandas (수렴표·안정성 스캔표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (해 감쇠, 임계 전후 semilog, FTCS/BTCS 대비, $\Delta t$-수렴 log-log — 영문 라벨)
- **외부 데이터 없음** — 모든 IC/해는 코드 안에서 생성, 재현 가능

## 다음 Day 예고
**Day 43 — Chapter 12 §12.2 Hyperbolic Problems (파동방정식) 또는 §12.3 Elliptic Problems
(푸아송/라플라스)**: §12.1 에서 익힌 $\theta$-method 와 삼중대각은, 쌍곡형에서는 **CFL 조건**과
명시적 도약(leapfrog)으로, 타원형에서는 **2D 블록 삼중대각/오중대각**(Day 10 밴드 + Day 41 차분의
재회)으로 확장된다.

---

### 📌 커리큘럼 노트
`_meta/curriculum.md` 의 *명시적* 진도표는 Day 13(§3.3)까지이며, 이후는 스케줄러가
"Ch 4 → … → Ch 11 → Ch 12" 흐름을 자동 진행 중입니다. Day 41 로 Chapter 11 §11.2(유한차분 BVP)를
마쳤고, Chapter 11 은 C&K 7판 기준 §11.1–11.2 두 절뿐이라 **Day 42 는 Chapter 12 §12.1
Parabolic Problems(열방정식)** 로 자연 진입했습니다 — Day 41 README 의 PDE 예고와 일치. 진도 방향을
직접 정하시려면 `_meta/curriculum.md` 에 `Day 14+` 항목을 추가해 주세요(스케줄러가 명시적 항목 우선).
