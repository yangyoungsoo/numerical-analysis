# Day 52 — §13.7 Trust-Region Methods (신뢰영역법)

챕터 13(함수의 최적화) **심화 절**. Day50(§13.5 SQP)·Day51(§13.6 내부점법)에서 제약 최적화를
*직선탐색/장벽*으로 다뤘다. 이번 절은 스텝의 **길이와 방향을 모델 신뢰도로 동시에 조절**하는
**신뢰영역(trust-region)** 골격을 무제약 최소화에서 정면으로 구현한다. 세 노트북이 같은 시험함수
(Rosenbrock)와 부분문제(TRS)를 (1) Cauchy·dogleg, (2) 정확 secular 해, (3) Steihaug–Toint CG의
세 관점으로 공략한다.

> **공통 부분문제 (TRS)**: `min  m(p)=g'p + ½ p'B p   s.t.  ||p|| ≤ Δ`
> 최적성: 어떤 `λ≥0` 에 대해 `(B+λI)p=-g`, `λ(Δ-||p||)=0`, `B+λI ⪰ 0`.
> 일치비율 `ρ = (실제감소)/(예측감소)` 로 스텝 수락·반경 갱신.

## 학습 목표
- 신뢰영역 골격(모델 + 반경 + $\rho$ 갱신)을 구현하고, 부분문제를 **Cauchy 점(1차)** 과 **dogleg(뉴턴 활용)** 으로 풀어 수렴 속도 차이를 본다.
- TRS를 **정확히** 풀어(Levenberg–Marquardt 모수 $\lambda$ 의 **secular 방정식**) **음의 곡률**과 **hard case**($g\perp q_{\min}$)까지 처리한다.
- 대규모용 **Steihaug–Toint truncated CG**(행렬-벡터 곱만)로 부분문제를 근사해 풀고, 신뢰영역법과 직선탐색법의 총 반복수·강건성, 초기반경 $\Delta_0$ 민감도를 비교한다.

## 문제 목록
| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_13_7_01.ipynb` | 신뢰영역 골격 — Cauchy 점 vs dogleg | $\rho$ 기반 반경 갱신, dogleg 경로(Cauchy→Newton) | Rosenbrock $(-1.2,1)\to(1,1)$: **dogleg 24회**(2차 수렴) vs **Cauchy 19,490회**(최급강하급) |
| 2 | `CE_13_7_02.ipynb` | 정확 TRS — secular 방정식·음의 곡률·hard case | $(B+\lambda I)p=-g$, 고유분해 $\phi(\lambda)=\Delta$ 이분법 | 부정부호 $B$: $\lambda^\*\approx3.03$, $\\|p^\*\\|=1$, 잔차 $0$; hard case $\lambda^\*=2=-\lambda_{\min}$ 에 $q_1$ 보정 |
| 3 | `CE_13_7_03.ipynb` | Steihaug–Toint CG·TR vs 직선탐색 | truncated CG(경계/음의곡률/잔차 종료), $\Delta_0$ 스윕 | Steihaug **24회**=dogleg 24회≈line-search 21회; $\Delta_0$ 100배 변화에도 23–29회로 **둔감(robust)** |

## 한 줄 정리
> **신뢰영역법 = "스텝의 방향과 길이를 일치비율 $\rho$ 로 동시에 조절"** — 같은 TRS를 Cauchy(1차)·dogleg($B\succ0$)·정확 secular(모든 경우)·Steihaug CG(대규모)로 정확도/비용을 달리해 풀며, 음의 곡률을 자동 처리하고 초기 반경에 둔감하다.

## 다음 Day 예고
챕터 13(최적화)의 직선탐색·제약·SQP·내부점·신뢰영역 골격을 모두 거쳤다. 다음 Day는
**§13.8 비선형 최소제곱(Gauss–Newton·Levenberg–Marquardt)** 또는 커리큘럼 확장(새 챕터)으로 이어간다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, 이후 Day14–52 는 스케줄러가
> 챕터 로드맵(Ch 1–13)을 따라 절 단위로 자동 진행해 온 것이다. 본 Day52(§13.7 신뢰영역법)는
> Day51(§13.6 내부점법)에 이은 챕터 13의 **추가 심화 확장**이다. 새 챕터/주제로 진도를 잇고 싶다면
> `_meta/curriculum.md` 를 확장하면 된다.
