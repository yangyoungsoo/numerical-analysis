# Day 53 — §13.8 Nonlinear Least Squares (비선형 최소제곱: Gauss–Newton · Levenberg–Marquardt)

챕터 13(함수의 최적화) **데이터 적합** 절. Day52(§13.7 신뢰영역법)까지 무제약·제약 최소화를 다뤘다.
이번 절은 잔차의 제곱합 $f(\mathbf x)=\tfrac12\lVert\mathbf r(\mathbf x)\rVert^2$ 라는 **특수 구조**를 활용해, 헤시안을
야코비안만으로 근사하는 **Gauss–Newton** 과 그 감쇠판인 **Levenberg–Marquardt(LM)** 를 정면으로 구현한다.
세 노트북이 (1) GN 의 원리와 한계, (2) LM 의 감쇠·적응, (3) 두 방법의 강건성·조건수를 차례로 공략한다.

> **공통 구조**: $\nabla f=J^\top\mathbf r$, $\ \nabla^2 f=J^\top J+\sum_i r_i\nabla^2 r_i$.
> Gauss–Newton 은 둘째 항(곡률항 $S$)을 버린다: $J^\top J\,\boldsymbol\delta=-J^\top\mathbf r$.
> Levenberg–Marquardt 는 감쇠로 안정화: $(J^\top J+\lambda D)\,\boldsymbol\delta=-J^\top\mathbf r$, 이득비 $\rho$ 로 $\lambda$ 조절.

## 학습 목표
- 비선형 최소제곱의 구조($\nabla^2 f\approx J^\top J$)를 이해하고 **Gauss–Newton** 을 구현, **full Newton** 과 비교해 *작은 잔차 → 거의 2차 수렴, 큰 잔차 → 둔화* 를 확인한다.
- **Levenberg–Marquardt** 를 이득비 기반 적응 $\lambda$ 로 구현해, **나쁜 초기값·특이 $J^\top J$** 에서도 강건히 수렴함을 GN·최급강하와 비교한다.
- 초기값 격자에 대한 **수렴 성공맵(basin of attraction)** 과 야코비안 **조건수**로 GN 대 LM 의 강건성을 정량 비교한다.

## 문제 목록
| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_13_8_01.ipynb` | Gauss–Newton — 원리와 small/large residual | $J^\top J\boldsymbol\delta=-J^\top\mathbf r$, full Newton 의 곡률항 $S$ 비교 | $y=a e^{bt}$ 적합: small-residual **GN 6회 ≈ Newton 5회**(거의 2차); large-residual 은 **13회**로 둔화 |
| 2 | `CE_13_8_02.ipynb` | Levenberg–Marquardt — 감쇠·이득비 적응 | $(J^\top J+\lambda D)\boldsymbol\delta=-J^\top\mathbf r$, $\rho$ 로 $\lambda$ 갱신 | 가우시안 봉우리·나쁜 초기값: **LM 30회 수렴**, 순수 GN 은 폭주, 최급강하는 수백~수천 회 |
| 3 | `CE_13_8_03.ipynb` | 강건성·조건수 — GN vs LM 정량 비교 | 초기값 격자 성공맵, $\kappa(J^\top J)$ 추적 | basin 성공률 **GN 91.4% vs LM 100%**; 이중지수합에서 $\kappa$ 가 **2.1×10⁸ → 1.6×10⁷**(감쇠) 로 하락 |

## 한 줄 정리
> **비선형 최소제곱 = "$\nabla^2 f\approx J^\top J$ 라는 공짜 헤시안 근사"** — Gauss–Newton 은 잔차가 작을 때 거의 2차로 수렴하지만, Levenberg–Marquardt 는 감쇠 $\lambda$ 로 GN(빠름)과 최급강하(안전)를 보간해 조건수를 낮추고 수렴 영역을 넓혀 표준 도구가 된다.

## 다음 Day 예고
챕터 13의 직선탐색·신뢰영역·제약(SQP/내부점)·비선형 최소제곱(GN·LM)을 모두 거쳤다. 다음 Day 는
**§13.9 도함수 없는 최적화(Nelder–Mead·좌표강하)** 또는 커리큘럼 확장(새 챕터)으로 이어간다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, 이후 Day14–53 은 스케줄러가
> 챕터 로드맵(Ch 1–13)을 따라 절 단위로 자동 진행해 온 것이다. 본 Day53(§13.8 비선형 최소제곱)은
> Day52(§13.7 신뢰영역법) README 의 예고대로 챕터 13 의 **데이터 적합 절**을 이어간 것이다. 새 챕터/주제로
> 진도를 잇고 싶다면 `_meta/curriculum.md` 를 확장하면 된다.
