# Day 46 — §13.2 Multivariate Minimization: 최급강하 · 켤레기울기 · Newton/BFGS (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 13. Minimization of Functions
> **절**: §13.2 Minimization of Multivariate Functions

## 오늘의 큰 그림
Day 45 의 *한 변수* 최소화(황금분할·포물선·Newton)를 **다변수**로 끌어올린다. 도함수는 *기울기 벡터* $\nabla f$,
2계도함수는 *헤시안 행렬* $\nabla^2 f$ 가 되며, 어제의 삼각 trade-off(안전성·도함수 자유·2차 수렴)가 고차원에서
그대로 재현된다. 세 문제를 관통하는 한 가지 질문 — **곡률의 비등방성(조건수)을 어떻게 다루는가**:

$$
\boxed{\;
\text{steepest descent: } \tfrac{\kappa-1}{\kappa+1}\ \text{(선형)}\quad\longrightarrow\quad
\text{CG: } \tfrac{\sqrt\kappa-1}{\sqrt\kappa+1}\ (\le n\ \text{스텝})\quad\longrightarrow\quad
\text{Newton: 2차}
\;}
$$

| 문제 | 초점 | 핵심 결과 |
|---|---|---|
| **1** | 최급강하법(steepest descent) | 정확한 직선탐색, **선형 수렴** $\tfrac{\kappa-1}{\kappa+1}$, 큰 $\kappa$ 에서 지그재그 |
| **2** | 켤레기울기법(CG) | $A$-직교 방향, **≤ $n$ 스텝** 유한 종료, $\sqrt\kappa$ 의존 |
| **3** | Newton / BFGS (Rosenbrock) | Newton **2차·기계정밀도**(헤시안要), BFGS **초선형**(기울기만) |

## 문제 목록

| # | 파일 | 주제 | 비교/시각화 | 한 줄 결론 |
|---|---|---|---|---|
| 1 | [`CE_13_2_01.ipynb`](CE_13_2_01.ipynb) | Steepest descent | 조건수-반복수 표, 2D 지그재그, 오차 semilog | 기울기만 쓰면 안전하나 $\kappa$ 에 지배되는 선형 수렴 |
| 2 | [`CE_13_2_02.ipynb`](CE_13_2_02.ipynb) | Conjugate gradient | 유한 종료표, CG vs SD 오차곡선 | $A$-직교로 ≤ $n$ 스텝, 지그재그 제거 |
| 3 | [`CE_13_2_03.ipynb`](CE_13_2_03.ipynb) | Newton & BFGS | Rosenbrock 경로, 수렴차수, 오차곡선 | Newton 2차(헤시안要), BFGS 초선형(기울기만) |

## 수치 하이라이트 (실행 결과)
- **문제 1 (최급강하·선형)**: $n=20$, SPD 스펙트럼을 $[1,\kappa]$ 전체에 펼친 스윕에서 tol 도달 반복수가
  $\kappa=2,10,50,100,500$ 에 대해 **16 → 81 → 373 → 692 → 3143** 로 *단조 급증*. $n=20,\ \kappa=50$ 에서
  $A$-노름 오차의 후반부 **점근 축소인자 0.9606** 이 이론 $\tfrac{\kappa-1}{\kappa+1}=0.9608$ 과 거의 일치
  (스펙트럼이 퍼져 상한이 tight). 2D($\kappa=50$)에서는 등고선 위 **90° 꺾임 지그재그** 경로로 골 바닥 직진 실패를 시각화.
- **문제 2 (CG·유한 종료)**: $n=8,\ \kappa=1000$ SPD 에서 CG 의 $A$-노름 오차가 **스텝 8(=$n$)에서 $1.96\times10^{-12}$**
  로 붕괴 — 켤레 방향이 각 좌표를 한 번씩 정리하는 유한 종료성. 같은 행렬에서 **최급강하는 12,281 반복**을 기어가는 반면
  CG 는 8 반복. 수렴인자 $\tfrac{\kappa-1}{\kappa+1}\approx0.998$ 대 $\tfrac{\sqrt\kappa-1}{\sqrt\kappa+1}\approx0.939$,
  $\sqrt\kappa$ 의존이 압도적 격차를 만든다.
- **문제 3 (Newton·2차 / BFGS·초선형)**: Rosenbrock($a=1,b=100$, 최소 $(1,1)$)을 출발 $(-1.2,1)$ 에서.
  **Newton 22 반복 → $f=0,\ \|err\|=0$**(기계정밀도), 말기 경험적 수렴차수 $p\to1.9$($\approx2$). **BFGS 36 반복 →
  $\|err\|=1.7\times10^{-12}$**(초선형, 헤시안 미계산). 백트래킹 라인서치가 두 방법을 먼 출발점에서 전역화.
- **공통**: steepest$\leftrightarrow$이분법(선형·안전), CG$\leftrightarrow$비선형CG(곡률 기억), Newton-min$\leftrightarrow$
  Newton-root(2차·헤시안要), BFGS$\leftrightarrow$secant(준-Newton·도함수 절약) — Day 11–13 / Day 45 구조의 *다차원* 재현.

## 한 줄 정리
> 다변수 최소화는 **곡률의 비등방성(조건수)을 얼마나 영리하게 쓰는가**의 게임이다 — 기울기만 쓰는 최급강하는
> $\kappa$ 에 끌려 지그재그, CG 는 $A$-직교로 ≤ $n$ 스텝, Newton 은 헤시안으로 2차 수렴, BFGS 는 기울기 차로
> 헤시안을 *학습*해 초선형. 그래서 실전 최적화기는 **L-BFGS / Newton-CG / trust-region** 으로 수렴차수·비용·안전성을 절충한다.

## 사용 라이브러리
- NumPy (SPD 생성·QR, 최급강하/CG/Newton/BFGS 반복, 헤시안 풀이 `linalg.solve`)
- Pandas (조건수-반복수 표·유한종료 표·수렴차수 표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (등고선+경로, 오차 semilog, Rosenbrock 골짜기 — 영문 라벨)
- **외부 데이터 없음** — 모든 행렬/함수/임계점은 코드 안에서 생성, 재현 가능

## 다음 Day 예고
**Day 47 — §13.2 (continued): Trust-Region / Levenberg–Marquardt 와 비선형 최소제곱**. 라인서치의 대안적 전역화로
*신뢰영역*을 도입하고, 음/특이 곡률을 안전하게 다루며, 비선형 최소제곱(Gauss–Newton → LM)으로 이어 오늘의
trade-off(수렴차수 vs 비용 vs 안전성)를 한 단계 더 정교화한다.

---

### 📌 커리큘럼 노트
`_meta/curriculum.md` 의 *명시적* 진도표는 Day 13(§3.3)까지이며, 이후는 스케줄러가 챕터 흐름을 자동 진행해
왔습니다. **Day 45 가 Chapter 13 §13.1(한 변수 최소화)** 를 열었고, **Day 46 은 §13.2(다변수 최소화)** 로
이어집니다. 진도 방향을 직접 정하시려면 `_meta/curriculum.md` 에 `Day 14+` 항목을 추가해 주세요(스케줄러가 명시적 항목 우선).
