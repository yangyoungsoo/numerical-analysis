# Numerical Analysis Daily

매일 *Numerical Mathematics and Computing* (Cheney & Kincaid, 7판)의
**Computer Exercises** 를 3문제씩 Python 으로 풀어 업로드합니다.

## 진행 방식
- **하루 3문제**, 챕터 순서대로 (현재 §1.1 부터)
- 각 문제는 **Jupyter Notebook (`.ipynb`)** 으로 작성
- 노트북 구성: 문제 원문 → 한국어 정리 → 수학적 배경 (LaTeX 수식) → 풀이 흐름 → 코드 + 실행 결과 → 결과 해석
- 매일 오전 9시 자동 푸시

## 진행 상황
| Day | 절 | 문제 | 폴더 |
|---|---|---|---|
| 01 | §1.1 Introduction | CE 1–3 (전진/중심차분) | [`Day01/`](Day01/) |

## 챕터 로드맵
1. **Ch 1** — Floating-point, 오차, Taylor 급수, Loss of significance
2. **Ch 2** — 선형시스템 (Gauss elimination, Pivoting, Tridiagonal)
3. **Ch 3** — 비선형방정식 (Bisection, Newton, Secant)
4. **Ch 4** — 보간법 + 수치미분 (Lagrange, Newton form, Richardson)
5. **Ch 5** — 수치적분 (Trapezoid, Romberg, Simpson, Gauss quadrature)
6. **Ch 6** — Spline 함수
7. **Ch 7** — 초기값 문제 (Euler, Runge-Kutta, Adams-Bashforth-Moulton)
8. **Ch 8** — 선형시스템 II (LU, Cholesky, SVD, Iterative)
9. **Ch 9** — 최소제곱과 Fourier 급수
10. **Ch 10** — Monte Carlo
11. **Ch 11** — 경계값 문제 (Shooting, Finite-difference)
12. **Ch 12** — 편미분방정식 (Heat, Wave, Poisson)
13. **Ch 13** — 함수의 최적화

## 환경
```bash
pip install numpy scipy matplotlib pandas jupyter
```
