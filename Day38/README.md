# Day 38 — §10.2 Quasi–Monte Carlo: 차원의 저주 · 저불일치 수열 · 랜덤화 QMC (Computer Exercises 1–3)

> **교재**: Cheney & Kincaid, *Numerical Mathematics and Computing* (7th ed.)
> **챕터**: 10. Monte Carlo Methods and Simulation
> **절**: §10.2 Quasi–Monte Carlo / High-Dimensional Integration

## 오늘의 큰 그림
Day 37(§10.1)에서 본 몬테카를로의 약점은 **느린 $N^{-1/2}$ 수렴**이었다. §10.2 는 이 약점을
두 방향에서 따져본다. 먼저 **왜 그 느림을 감수하는가** — 결정론적 격자법이 차원의 저주로
무너지기 때문(문제 1). 다음으로 **그 느림을 어떻게 깎는가** — 점을 무작위가 아니라 *골고루*
배치하는 **저불일치(low-discrepancy) 수열**로 차수를 $N^{-1/2}\to N^{-1}$ 까지 끌어올린다(문제 2).
마지막으로 그 결정론적 QMC 에 **오차막대를 되돌려주는** 랜덤화 QMC, 그리고 차원이 커지면
그 이득이 다시 식는다는 사실(문제 3)로 닫는다.

세 문제를 관통하는 *한 식* — **적분오차의 두 얼굴**:

$$
\boxed{\;
\underbrace{E_{\text{MC}}=\mathcal{O}(N^{-1/2})}_{\text{차원 무관, 느림}}
\qquad\text{vs}\qquad
\underbrace{|\hat I_N-I|\le V(g)\,D_N^{*}}_{\text{Koksma–Hlawka}}
\;\xrightarrow{\;D_N^{*}\sim (\log N)^d/N\;}\;
E_{\text{QMC}}=\mathcal{O}\!\big(N^{-1}(\log N)^d\big).
\;}
$$

| 문제 | 건드리는 부분 | 핵심 결과 |
|---|---|---|
| **1** | 격자 vs MC, 차원의 저주 | 격자 $N^{-2/d}$ 는 $d=4$ 에서 MC $N^{-1/2}$ 와 교차 — $d>4$ 부터 MC 승 |
| **2** | 저불일치 수열 + QMC | "골고루 채움" = 작은 불일치 = 작은 오차, 차수 $-1/2\to-1$ |
| **3** | 랜덤화 QMC + discrepancy | RQMC = QMC 정확도 + MC 오차막대, 단 고차원에서 이득 소멸 |

## 문제 목록

| # | 파일 | 주제 | 비교/시각화 | 한 줄 결론 |
|---|---|---|---|---|
| 1 | [`CE_10_2_01.ipynb`](CE_10_2_01.ipynb) | 차원의 저주: 격자 vs MC | 차원별 오차표, error vs N (d=2,8), 손익분기 기울기 | 격자는 $d>4$ 에서 무너지고 MC 가 이긴다 |
| 2 | [`CE_10_2_02.ipynb`](CE_10_2_02.ipynb) | 저불일치 수열 (Halton·Lattice) | 점 분포 비교, error vs N log-log, 경험 기울기 | 골고루 채우면 $N^{-1/2}\to N^{-1}$ |
| 3 | [`CE_10_2_03.ipynb`](CE_10_2_03.ipynb) | 랜덤화 QMC + 불일치 + 차원효과 | 분산비 표, $L_2$ discrepancy 곡선, 차원 스윕 | RQMC 는 정확도+오차막대, 고차원엔 약함 |

## 수치 하이라이트 (실행 결과)
- **문제 1 (손익분기)**: 같은 예산 $N\approx2\times10^4$ 에서 winner — $d=1,2,3,4$ 는 **grid**,
  $d=6,8$ 은 **MC**. 격자 오차는 $d=2$ 에서 $2.5\times10^{-5}$(MC $7.3\times10^{-3}$)로 압승하지만
  $d=8$ 에선 $0.20$(MC $0.086$)로 역전 — 이론적 교차점 $d=4$ 와 정확히 일치.
- **문제 2 (수렴 차수)**: 경험 log-log 기울기 — $d=2$ 에서 MC $-0.50$, Halton $-0.91$, Lattice $-1.17$;
  $d=4$ 에서 MC $-0.44$, Halton $-0.87$, Lattice $-0.94$. QMC 가 MC 를 압도, $d$ 증가 시 약간 완만.
- **문제 3 (RQMC 효율)**: $d=2$ 에서 분산비 MC/RQMC 가 $N$ 과 함께 **26배($N{=}256$) → 727배($N{=}8192$)**
  로 증가. RQMC 편향은 $\sim10^{-4}$ 이하(불편). $L_2$ star discrepancy 기울기 — 격자 $-0.83$, 유사난수 $-0.50$.
- **문제 3 (차원의 역습)**: RQMC 수렴 기울기가 $d{=}1$ 의 $-0.93$ 에서 $d{=}16$ 의 $-0.39$ 로 완만해져
  MC($-0.44$)와의 격차가 **소멸** — $(\log N)^d$ 부담으로 차원의 저주가 QMC 에도 돌아온다.

## 한 줄 정리
> 몬테카를로의 $N^{-1/2}$ 는 "차원에 강하지만 느린" 보험이고, 준-몬테카를로는 점을 *골고루* 두어
> 거의 $N^{-1}$ 로 빨라지지만 — 그 이득은 차원이 커지면 $(\log N)^d$ 에 잠식되어 다시 $N^{-1/2}$ 로 수렴한다.
> 결국 모든 것은 **불일치 $D_N^*$ 를 얼마나 작게 만드느냐**의 문제다.

## 사용 라이브러리
- NumPy (`default_rng`, 벡터화 radical inverse, rank-1 lattice, Warnock 불일치)
- Pandas (차원·분산비·기울기 표, `pd.set_option("display.float_format", ...)`)
- Matplotlib + (mpl 기본) — 점 분포 산점도, log-log 수렴, discrepancy 곡선 (영문 라벨)
- **외부 패키지 없음** — Sobol 대신 자체 구현 rank-1 격자(Kronecker/Roberts $R_d$)와 Halton 사용,
  모든 난수/저불일치 점은 코드 안에서 생성

> **구현 노트**: 이 환경의 SciPy 설치가 불완전(`scipy.stats` 누락)이어서, 원안의
> `scipy.stats.qmc.Sobol` 대신 **NumPy 만으로 구현한 rank-1 격자(랜덤 시프트와 궁합이 좋은
> *randomly shifted lattice rule*)** 로 대체했습니다. 저불일치 수열의 모든 학습 목표
> (균일 충전·$N^{-1}$ 수렴·랜덤화 QMC·불일치 측정)는 그대로 달성됩니다.

## 다음 Day 예고
**Day 39 — §10.3 Simulation**: 적분 추정 기법을 **확률모형 시뮬레이션**(대기행렬·랜덤워크·신뢰성)에
적용한다. 적분이 곧 "사건 확률·기대비용"의 추정이 되며, 분산감소·QMC 가 시뮬레이션 효율로 직결되는
장면으로 Ch 10 을 마무리한다.

---

### 📌 커리큘럼 노트
`_meta/curriculum.md` 의 *명시적* 진도표는 Day 13(§3.3)까지이며, 그 이후(Day 14~)는 스케줄러가
"Ch 4 보간 → … → Ch 9 최소제곱 → **Ch 10 Monte Carlo**" 흐름을 자동 진행 중입니다.
Day 37 에서 §10.1(Monte Carlo Methods)을 다뤘고, Day 37 README 의 예고대로 이번 **Day 38 은
§10.2(Quasi–Monte Carlo)** 로 이어졌습니다. 진도 방향을 바꾸시려면 `_meta/curriculum.md` 에
`Day 14+` 항목을 명시적으로 추가해 주세요 — 스케줄러가 명시적 항목을 우선합니다.
