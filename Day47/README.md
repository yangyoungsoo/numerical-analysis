# Day 47 — §13.2 (continued) 신뢰영역 · Gauss–Newton · Levenberg–Marquardt (Computer Exercises 4–6)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 13. Minimization of Functions
> **절**: §13.2 Minimization of Multivariate Functions (continued)

## 오늘의 큰 그림
Day 46 의 *라인서치* 전역화(최급강하·CG·Newton/BFGS)에 이어, 오늘은 **신뢰영역(trust-region)** 이라는
또 다른 전역화 틀과 그 *비선형 최소제곱 특수화*인 **Gauss–Newton / Levenberg–Marquardt** 로 넘어간다.
세 문제를 관통하는 한 가지 질문 — **2차 모델을 얼마나 *멀리* 믿을 것인가(스텝 길이/반지름 제어)**:

$$
\boxed{\;
\text{Newton step} \;\xrightarrow{\ \|\mathbf p\|\le\Delta\ }\; \text{trust-region (dogleg, } \rho\text{)}
\;\xrightarrow{\ f=\frac12\|\mathbf r\|^2,\ B\approx J^\top J\ }\;
\text{Gauss–Newton} \;\xrightarrow{\ +\lambda I\ }\; \text{Levenberg–Marquardt} \;}
$$

| 문제 | 초점 | 핵심 결과 |
|---|---|---|
| **4** | 신뢰영역(dogleg, $\rho$) | 온순한 문제엔 Newton 도 OK, **음의 곡률**에선 TR 만 안장 탈출 |
| **5** | Gauss–Newton vs Levenberg–Marquardt | 나쁜 출발점에서 **GN 발산 / LM 수렴** |
| **6** | LM 감쇠 $\lambda$ $\leftrightarrow$ 신뢰영역 $\Delta$ | $\|\mathbf p(\lambda)\|$ 단조 감소 ⇒ $\lambda\equiv\Delta$, 적응이 고정을 이김 |

## 문제 목록

| # | 파일 | 주제 | 비교/시각화 | 한 줄 결론 |
|---|---|---|---|---|
| 4 | [`CE_13_2_04.ipynb`](CE_13_2_04.ipynb) | Trust-region (dogleg) | Rosenbrock 경로, $\Delta_k/\rho_k$ 추이, 음의 곡률 안장 탈출 | $\rho$ 로 반지름을 조율해 2차 정보를 안전하게 사용 |
| 5 | [`CE_13_2_05.ipynb`](CE_13_2_05.ipynb) | Gauss–Newton vs LM | 지수 적합, 목적함수 발산/수렴 곡선 | $\lambda I$ 한 항이 전역 안정성을 산다 |
| 6 | [`CE_13_2_06.ipynb`](CE_13_2_06.ipynb) | 감쇠 $\lambda$ ↔ 신뢰영역 | $\|p(\lambda)\|$ 단조 곡선, 전략별 수렴 | $\lambda$ 선택 = $\Delta$ 선택, 적응적 제어가 핵심 |

## 수치 하이라이트 (실행 결과)
- **문제 4 (신뢰영역)**: 해석적 헤시안을 유한차분으로 검증(최대오차 $4.8\times10^{-8}$). Rosenbrock $(-1.2,1)$ 출발에서
  TR 은 **25 반복 → $(1,1)$, $f=0$**, 순수 full-step Newton 은 **8 반복** 으로 *둘 다* 수렴(헤시안 양정치인 온순한 문제).
  반면 음의 곡률 함수 $g(x,y)=x^4-x^2+y^2$ (출발점 헤시안 고유값 $[-1.73,\,2]$, 부정정)에서는 **순수 Newton 이 안장점
  $(0,0)$ 에 갇히고**(값 $0$), **신뢰영역은 도그렉의 음-곡률 분기로 진짜 최소 $(\pm1/\sqrt2,0)$, 값 $-1/4$** 로 탈출.
- **문제 5 (GN vs LM)**: 지수 모델 $\phi=x_1e^{x_2 t}$(참값 $(2.5,-1.3)$)을 *부호가 틀린* 출발 $(2.5,2.0)$ 에서 적합.
  **Gauss–Newton 은 5 반복 만에 $f\approx7.7\times10^{46}$ 로 발산**(오버플로), **Levenberg–Marquardt 는 11 반복으로
  $f=5.4\times10^{-3}$ 수렴**. LM 의 $\lambda$ 가 위기 구간에 $3.0\times10^{-2}$ 까지 솟았다가 수렴 말기 $1.2\times10^{-4}$ 로 하강.
- **문제 6 (감쇠↔신뢰영역)**: $\|\mathbf p(\lambda)\|$ 가 $\lambda$ 에 대해 **단조 감소**(스윕에서 $3.82\to1.6\times10^{-3}$) 확인 ⇒
  "$\lambda$ 선택 = $\Delta$ 선택". 가우시안 봉우리 적합에서 전략별 외부 반복: **고정 $\lambda=10^{2}$ 는 247 회**(느림),
  **고정 $\lambda=10^{-3}$ 20 회**, **Marquardt 적응 11 회**, **신뢰영역 10 회**(단, 매 스텝 $\lambda$ 이분 탐색으로 선형풀이는 271 회).
- **공통 구조**: trust-region$\leftrightarrow$line-search(전역화 쌍), GN$\leftrightarrow$Newton(2차 모델), LM$\leftrightarrow$TR(감쇠=반지름).
  Day 11–13(근 찾기)·Day 45–46(최소화)의 *수렴차수 vs 비용 vs 안전성* trade-off 가 여기서 **강건성** 축으로 한 번 더 재현된다.

## 한 줄 정리
> 신뢰영역은 **2차 모델을 얼마나 멀리 믿을지($\Delta$)** 를 $\rho$ 로 조율하는 전역화이고, Levenberg–Marquardt 의
> 감쇠 $\lambda$ 는 바로 그 반지름의 *비선형 최소제곱판*이다 — GN 의 속도와 최급강하의 안전성을 $\lambda(=\Delta)$
> 하나로 연속 보간한다. 그래서 실전 곡선 적합/캘리브레이션의 표준 작업마는 **LM**, 범용 최적화는 **trust-region Newton-CG** 다.

## 사용 라이브러리
- NumPy (도그렉/신뢰영역, GN·LM 반복, `linalg.solve`, 헤시안 유한차분 검증, 고유분해)
- Pandas (반복 기록·전략 비교 표, `pd.set_option("display.float_format", ...)`)
- Matplotlib (등고선+경로, $\Delta_k/\rho_k$ 추이, 발산/수렴 semilog, $\|p(\lambda)\|$ 단조 곡선 — 영문 라벨)
- **외부 데이터 없음** — 모든 함수/데이터/임계점은 코드 안에서 생성, 재현 가능

## 다음 Day 예고
**Day 48 — 제약 있는 최적화 도입**: KKT 조건, 페널티/장벽(내부점) 방법, 또는 §13.2 의 마무리로서 전역 최적화의
맛보기. 오늘까지의 *무제약* trade-off(수렴차수·비용·안전성)에 **제약/전역성** 축을 더한다.

---

### 📌 커리큘럼 노트
`_meta/curriculum.md` 의 *명시적* 진도표는 Day 13(§3.3)까지이며, 이후는 스케줄러가 챕터 흐름을 자동 진행해
왔습니다. **Day 45 §13.1(한 변수 최소화) → Day 46 §13.2(다변수, 라인서치) → Day 47 §13.2(신뢰영역/최소제곱)** 로
이어집니다. 진도 방향을 직접 정하시려면 `_meta/curriculum.md` 에 `Day 14+` 항목을 추가해 주세요(스케줄러가 명시적 항목 우선).
