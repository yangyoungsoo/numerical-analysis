# 학습 커리큘럼

매일 3문제씩, Cheney & Kincaid의 *Numerical Mathematics and Computing* (7판) 흐름을 따라간다.
스케줄러는 매일 이 파일을 읽고, 현재까지 푸시된 가장 높은 `DayNN` 폴더를 보고 다음 Day의 토픽을 결정한다.

## 챕터 1 — Mathematical Preliminaries and Floating-Point Representation

### Day 02 — §1.1 (continued)
* Maclaurin 급수로 $\arctan(1) = \pi/4$ 계산: 수렴 속도, 부분합 오차의 거동
* **Wilkinson 다항식** $p(x) = \prod_{k=1}^{20}(x-k)$의 계수에 작은 섭동을 주었을 때 근의 민감도
* **Horner 방식 vs 일반 다항식 평가**: 부동소수점 오차와 연산 횟수 비교

### Day 03 — §1.1 (continued)
* **합의 순서**가 결과에 미치는 영향: $\sum_{k=1}^{N} 1/k$를 작은→큰 순서, 큰→작은 순서로 더해 비교
* **분산(variance)** 계산: 2-pass 공식 vs 1-pass 공식의 수치 안정성
* **단정도(float32) vs 배정도(float64)**: 같은 계산을 두 정밀도로 돌려 자릿수 손실 비교

### Day 04 — §1.1 (continued)
* `(1 - cos x)/x²` 의 *loss of significance* 와 항등식 $2\sin^2(x/2)/x^2$ 로의 재작성
* `sqrt(x²+1) - x` 의 큰 $x$에서의 자릿수 손실, $1/(\sqrt{x²+1}+x)$ 로 재작성
* 작은 두 수의 차로 발생하는 cancellation을 자기 함수 선택해서 시각화

### Day 05 — §1.2 Mathematical Preliminaries
* Taylor 급수 절단오차 실험: $e^x$, $\sin x$, $\ln(1+x)$의 부분합과 참값 비교 (다양한 $x$)
* **Range reduction** for $\sin$: $\sin(10^{10})$ 같이 큰 인수에서의 정확도 회복
* Mean-Value Theorem 수치 검증: 임의 함수 + 구간에서 $f'(\xi)$를 만족하는 $\xi$ 찾기

### Day 06 — §1.3 Floating-Point Representation
* **Machine epsilon** 직접 측정: 1 + ε ≠ 1 인 가장 작은 ε
* IEEE 754 부동소수점의 **subnormal** 영역 탐색
* 대표적인 함수에서 **catastrophic cancellation** 발생 지점 찾기 (예: 이차방정식의 근)

### Day 07 — §1.4 Loss of Significance
* 이차방정식 $ax² + bx + c = 0$의 안정적 풀이: $-b ± \sqrt{b²-4ac}$ 의 부호 선택
* $\log(1+x) - x$ 의 작은 $x$에서의 안정 평가
* `expm1`, `log1p` 같은 표준 라이브러리 함수의 효용 비교

## 챕터 2 — Linear Systems

### Day 08 — §2.1 Naïve Gaussian Elimination
* $n \times n$ 무작위 행렬에 대해 **forward elimination + back substitution** 직접 구현
* 계산 횟수 $\mathcal{O}(n³)$ 측정 — 다양한 $n$에서 wall-clock vs 이론
* **Hilbert 행렬** $H_{ij} = 1/(i+j-1)$ — 작은 $n$에서도 발산하는 잔차 관찰

### Day 09 — §2.2 Gaussian Elimination with Scaled Partial Pivoting
* 위의 naïve elimination에 **scaled partial pivoting** 추가; 어떤 행렬에서 차이가 큰지
* **조건수(condition number)** 와 잔차의 관계 실험
* 일부러 만든 ill-conditioned 행렬 (예: Hilbert)에서 pivoting 효과 비교

### Day 10 — §2.3 Tridiagonal and Banded Systems
* **Thomas 알고리즘** 직접 구현 (3중대각, $\mathcal{O}(n)$)
* 1D 푸아송 방정식 $-u'' = f$ 의 finite-difference 행렬을 만들고 풀기
* 5중대각 (penta-diagonal) 시스템으로 확장

## 챕터 3 — Nonlinear Equations

### Day 11 — §3.1 Bisection Method
* $\cos x = x$, $x e^x = 1$ 등 root finding을 bisection으로
* **수렴 차수** 측정: $|x_{n+1} - r| / |x_n - r|$ 비율을 그래프로
* 다중근 / 함수 부호가 같은 경계 처리 (failure mode)

### Day 12 — §3.2 Newton's Method
* Newton 반복으로 $\sqrt{a}$ 계산 (Babylonian)
* **수렴 영역(basin of attraction)** 시각화: 복소 다항식의 fractal
* Newton이 발산하는 케이스 (변곡점 근처) 만들어 보기

### Day 13 — §3.3 Secant Method
* Secant로 위의 문제들 풀기, **superlinear** 수렴 차수 (≈ 1.618) 측정
* Newton vs Secant **비용/정확도 trade-off**: function eval 횟수 비교
* **Hybrid bisection + secant** (Brent 비슷한) 직접 만들기

## 향후 (Day 14+)
Ch 4 보간, Ch 5 적분, Ch 6 spline, Ch 7 ODE, Ch 8 LU/SVD, Ch 9 최소제곱, Ch 10 Monte Carlo, Ch 11 BVP, Ch 12 PDE, Ch 13 최적화 — 같은 호흡으로 절(section) 단위 진행.

> 스케줄러가 진도가 끝까지 도달하면, 이 파일을 다시 확장하면 됨.
