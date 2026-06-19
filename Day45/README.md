# Day 45 — §13.1 One-Variable Minimization: 황금분할 · 포물선 보간 · Newton (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 13. Minimization of Functions
> **절**: §13.1 One-Variable Case

## 오늘의 큰 그림
Day 42–44 의 PDE(§12) 절을 마치고, *방정식 풀이*에서 **목적함수 최적화**로 전환한다. 한 변수 최소화의
세 고전 기법 — **황금분할(golden section)**, **연속 포물선 보간(successive parabolic interpolation)**,
**Newton** — 을 같은 시험함수에 적용해 *수렴차수 ↔ 함수평가 비용 ↔ 안전성*의 삼각 trade-off 를 해부한다.
이 세 기법은 Day 11–13 의 root finding(이분법·secant·Newton)을 *한 번 미분한* 형태로 그대로 대응된다.

세 문제를 관통하는 *한 함수* — 고전 사차다항식:

$$
\boxed{\;
f(x)=x^4-14x^3+60x^2-70x,\qquad
f''(x)=12(x-2)(x-5)
\;}
$$

전역 최소 $x^\*\approx0.7809\,(f=-24.37)$, 국소 최소 $x\approx5.957\,(f=11.96)$, 그 사이 최대 $x\approx3.762$.

| 문제 | 초점 | 핵심 결과 |
|---|---|---|
| **1** | 황금분할 탐색 | 구간폭 $\tau=0.618$ 배 축소, **선형 수렴**, 반복당 함수평가 1회(안전) |
| **2** | 연속 포물선 보간 | **초선형**(order 1.32), 함수평가 격감 — 단 *평탄 최소*의 $\sqrt{\varepsilon}$ 정밀도 바닥 |
| **3** | Newton 최소화 | $f'=0$ 에 Newton → **2차 수렴·기계정밀도**, 그러나 전역성 없음(국소 최소로 이탈) |

## 문제 목록

| # | 파일 | 주제 | 비교/시각화 | 한 줄 결론 |
|---|---|---|---|---|
| 1 | [`CE_13_1_01.ipynb`](CE_13_1_01.ipynb) | Golden section search | 구간폭/오차 semilog | 도함수 없이 안전하게 가두나 $\tau$-선형이라 느림 |
| 2 | [`CE_13_1_02.ipynb`](CE_13_1_02.ipynb) | Parabolic interpolation | 오차-평가비용, golden 대비 배속 | 초선형으로 평가 격감, 평탄 최소서 $\sqrt{\varepsilon}$ 바닥 |
| 3 | [`CE_13_1_03.ipynb`](CE_13_1_03.ipynb) | Newton + 3-way 비교 | good/bad 출발, 세 방법 오차 | 2차 수렴·기계정밀도, 단 전역 최소 비보장 |

## 수치 하이라이트 (실행 결과)
- **문제 1 (황금분할·선형)**: $[0,2]$ 에서 관측 평균 축소인자 **0.618034** 가 이론 $\tau=(\sqrt5-1)/2$ 와
  정확히 일치. tol $10^{-8}$ 까지 **함수평가 42회**, 반복당 새 평가 1회(점 재사용). 중점오차도 $\tfrac12L_n$ 으로
  같은 선형률 — root finding 의 *이분법*에 대응하는 안전한 기준선.
- **문제 2 (포물선 보간·초선형)**: 슬라이딩 윈도(최근 3점)로 꼭짓점 도약. 오차가 $2.2\times10^{-6}\to1.7\times10^{-7}
  \to1.2\times10^{-10}$ 로 *가속*(이론 order **1.3247**, plastic number). 같은 정확도 도달 함수평가가
  golden 대비 tol $10^{-2}/10^{-4}/10^{-6}$ 에서 **1.33×/1.56×/2.70× 배속** — 조일수록 격차 확대. 단,
  *평탄한 최소*($f\approx f^\*+\tfrac12f''\Delta x^2$) 탓에 오차가 **$2.95\times10^{-11}$ 에서 바닥**에 부딪힌다
  (도함수 없는 모든 방법의 $\sqrt{\varepsilon}$ 한계).
- **문제 3 (Newton·2차)**: $x_0=1.5$ 에서 **7회 만에 $|err|=3.3\times10^{-16}$**(기계정밀도), 경험적 수렴차수
  **$p\to1.998$**($\approx2$). $f'=0$ 의 근은 평탄하지 않아($f''\ne0$) 문제 2 의 바닥이 없다. **실패 모드**:
  $x_0=2.02$($f''\approx0$)에서 출발하면 첫 도약이 $x\approx49.5$ 로 폭발, 이후 오른쪽 분지로 걸어 들어가
  *국소* 최소 **$x=5.957\,(f=11.96)$** 로 수렴 — 전역 최소 $-24.37$ 을 놓친다. Newton 은 최소/최대도, 전역/국소도
  구분 못 하고 bracketing 안전망도 없다.
- **공통**: golden$\leftrightarrow$이분법(선형·안전), parabolic$\leftrightarrow$secant(초선형·도함수無),
  Newton-min$\leftrightarrow$Newton-root(2차·도함수要) — Day 11–13 구조의 *미분된* 재현.

## 한 줄 정리
> 한 변수 최소화는 **속도(Newton 2차)·도함수 자유(parabolic 초선형)·안전성(golden 선형)** 의 삼각 trade-off 다 —
> 도함수를 안 쓰면 *평탄한 최소*의 $\sqrt{\varepsilon}$ 정밀도 바닥을 만나고, Newton 은 빠르고 정확하지만 전역 최소를
> 보장하지 못한다. 그래서 실전 최소화기는 *golden + parabolic(Brent)* 또는 *Newton + 안전 가드*로 묶는다.

## 사용 라이브러리
- NumPy (다항식 근 `roots`, 황금분할/포물선/Newton 반복, `polyfit` 보조)
- Pandas (수렴표·함수평가 비교표·종합 비교표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (함수/브래킷, 구간폭·오차 semilog, 오차-평가비용, 세 방법 수렴 비교 — 영문 라벨)
- **외부 데이터 없음** — 모든 함수/임계점은 코드 안에서 생성, 재현 가능

## 다음 Day 예고
**Day 46 — §13.2 Multivariate / Steepest Descent (다변수 최소화의 시작)**: 한 변수의 세 축(안전·초선형·2차)을
고차원으로 확장한다. 도함수는 *기울기 벡터*, 2계도함수는 *헤시안 행렬*이 되어 경사하강·켤레기울기·Newton/BFGS 로
이어지며, 오늘의 trade-off(수렴차수 vs 비용 vs 안전성)가 다차원에서 재현된다.

---

### 📌 커리큘럼 노트
`_meta/curriculum.md` 의 *명시적* 진도표는 Day 13(§3.3)까지이며, 이후는 스케줄러가 "Ch 4 → … → Ch 12"
흐름을 자동 진행해 왔습니다. Day 42–44 에서 Chapter 12(PDE)를 마무리했고, **Day 45 는 Chapter 13
(Minimization) §13.1 One-Variable Case** 로 새 장을 엽니다. 진도 방향을 직접 정하시려면
`_meta/curriculum.md` 에 `Day 14+` 항목을 추가해 주세요(스케줄러가 명시적 항목 우선).
