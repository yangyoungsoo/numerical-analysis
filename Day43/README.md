# Day 43 — §12.2 Hyperbolic Problems: CFL 안정성 · 시동식과 단위-CFL 무분산 · 2차 수렴/분산 (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 12. Partial Differential Equations
> **절**: §12.2 Hyperbolic Problems (1D Wave Equation)

## 오늘의 큰 그림
Day 42 §12.1 의 열방정식은 시간에 **1차** 였고, 안정 임계가 $r=\alpha\Delta t/\Delta x^2\le\tfrac12$ 였다.
§12.2 의 파동방정식 $u_{tt}=c^2u_{xx}$ 는 시간에 **2차 미분**을 가져, 시간·공간을 **둘 다 중심차분**하는
3-시간층 **leapfrog** 가 표준 출발점이 된다. 열방정식이 *감쇠(평활화)* 했다면 파동은 *진폭을 보존하며 진동*
한다. 세 문제는 같은 leapfrog 를 (1) **안정성**, (2) **시동·무분산**, (3) **수렴·분산** 세 각도로 해부한다.

세 문제를 관통하는 *한 식* — **시간·공간 중심차분(leapfrog)**:

$$
\boxed{\;
u_i^{n+1} = 2(1-r^2)\,u_i^{n} + r^2\big(u_{i-1}^{n}+u_{i+1}^{n}\big) - u_i^{n-1},
\qquad r=\frac{c\,\Delta t}{\Delta x}\;(\text{Courant number})
\;}
$$

| 문제 | 초점 | 핵심 결과 |
|---|---|---|
| **1** | CFL 안정성 | 양함수 leapfrog 는 $r\le1$ 에서만 안정 (열방정식 $\tfrac12$ 와 대비) |
| **2** | 시동식 · 단위 CFL | 첫 층 시동식 차수가 전체 정확도 결정; **$r=1$ 은 d'Alembert 해를 정확 재현(무분산)** |
| **3** | 수렴 · 분산 | 안정역 전체에서 **시간·공간 2차(slope 2)**; $r<1$ 은 짧은 파장을 느리게(분산) |

## 문제 목록

| # | 파일 | 주제 | 비교/시각화 | 한 줄 결론 |
|---|---|---|---|---|
| 1 | [`CE_12_2_01.ipynb`](CE_12_2_01.ipynb) | 양함수 leapfrog · CFL 한계 | 정상파 진동, $r=1$ 임계 전후 (semilog) | $r=c\Delta t/\Delta x\le1$ 에서만 안정, 넘으면 톱니 폭발 |
| 2 | [`CE_12_2_02.ipynb`](CE_12_2_02.ipynb) | 시동식 차수 · 단위 CFL 무분산 | 속도시동 해, $r=1$/$r<1$ 삼각펄스 | 2차 시동 필수; $r=1$ 은 무분산, $r<1$ 은 꼬리 진동 |
| 3 | [`CE_12_2_03.ipynb`](CE_12_2_03.ipynb) | 2차 수렴 · 수치분산 | 수렴 log-log, $c_{num}/c$ vs 파장 | slope≈2, 분산은 $r=1$ 에서 0·$r<1$ 에서 증가 |

## 수치 하이라이트 (실행 결과)
- **문제 1 (CFL 안정성)**: $c=1$, $M=50$. 깨끗한 IC $\sin\pi x$ + $r\le1$ 은 정상파 정확해 $\cos(\pi t)\sin\pi x$
  와 격자점 오차 $\sim2\times10^{-3}$ 로 일치. IC 에 $10^{-3}$ 고주파 잡음을 심으면 $T=1.0$ 까지 $r\le1.0$ 은
  `max|u|`$\approx1.0$ 으로 유계, 그러나 **$r=1.02\to3.8\times10^{4}$**, $r=1.05\to1.8\times10^{9}$ 로 폭주 —
  임계 $r=1$ 이 von Neumann 분석 그대로 재현. (열방정식의 $\tfrac12$ 대비 $\Delta t\sim\Delta x$ 로 *선형*, 더 관대.)
- **문제 2 (시동식·무분산)**: **(a)** 변위·속도 모두 비영인 해 $\sin\pi x[\cos\pi t+\sin\pi t]$ 에서, 곡률
  보정을 포함한 **2차 시동식은 관측차수 2.00**, 단순 $u^1=u^0+\Delta t\,g$ 의 **1차 시동식은 차수 1.00** —
  단 한 층의 시동 오차가 전 시간층으로 전파. **(b)** 삼각 펄스 전파에서 **$r=1$ 의 최대오차 $\sim1.7\times10^{-15}$**
  (기계정밀도, 무분산), $r<1$ 은 $\sim1\!\sim\!2\times10^{-2}$ 로 꼬리 진동.
- **문제 3 (2차 수렴·분산)**: $r$ 고정·격자 세분에서 log-log 기울기 **$r=0.5\to2.000$, $r=0.9\to2.018$**
  (안정역 차수는 $r$ 무관). 이산 분산관계 $c_{num}/c=\arcsin(r\sin\tfrac{k\Delta x}{2})/(r\tfrac{k\Delta x}{2})$ 로
  계산한 위상속도비는 **$r=1$ 에서 모든 파장 = 1**(무분산), $r<1$ 은 짧은 파장(파장당 4점)에서 0.90 수준까지
  처짐 — 펄스 꼬리 진동의 정량적 정체.
- **공통**: 세 문제 모두 "시간·공간 중심차분(leapfrog)" 한 골격을 공유하되, **$r=1$** 이 안정 한계이면서 동시에
  *무분산* 특이점이라는 쌍곡형의 이중적 의미를 드러낸다.

## 한 줄 정리
> 쌍곡형 PDE 의 시간전진은 격자를 특성곡선에 맞추는 일이다 — leapfrog 는 Courant 수 $r=c\Delta t/\Delta x\le1$
> 에서만 안정하고, 그 안에서 시간·공간 2차로 수렴하며, **$r=1$ 에서는 d'Alembert 해를 정확히 재현(무분산)**
> 한다. 안정과 무분산이 같은 한 점에서 만나는 것이 §12.2 의 핵심이다.

## 사용 라이브러리
- NumPy (leapfrog 벡터화 갱신, 2차/1차 시동식, von Neumann 증폭인자, 이산 분산관계, log-log 회귀 `polyfit`)
- Pandas (수렴표·안정성 스캔표·분산표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (정상파 진동, 임계 전후 semilog, 삼각펄스 무분산/분산, 수렴 log-log — 영문 라벨)
- **외부 데이터 없음** — 모든 IC/해는 코드 안에서 생성, 재현 가능

## 다음 Day 예고
**Day 44 — Chapter 12 §12.3 Elliptic Problems (Poisson/Laplace 방정식)**: 시간이 없는 정상 타원형 문제로,
2D 라플라시안을 **블록 삼중대각/오중대각**(Day 10 밴드시스템 + Day 41 차분의 재회)으로 풀고, 반복법
(Jacobi·Gauss–Seidel·SOR)의 수렴을 다룬다 — §12.1–12.2 의 *시간전진*이 *공간 반복*으로 바뀌는 전환점.

---

### 📌 커리큘럼 노트
`_meta/curriculum.md` 의 *명시적* 진도표는 Day 13(§3.3)까지이며, 이후는 스케줄러가 "Ch 4 → … → Ch 12"
흐름을 자동 진행 중입니다. Day 42 로 §12.1 Parabolic(열방정식)을 마쳤고, Day 42 README 의 예고대로
**Day 43 은 §12.2 Hyperbolic Problems(파동방정식)** 로 진입했습니다. 진도 방향을 직접 정하시려면
`_meta/curriculum.md` 에 `Day 14+` 항목을 추가해 주세요(스케줄러가 명시적 항목 우선).
