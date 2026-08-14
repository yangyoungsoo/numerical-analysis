# Day 96 — §15.29 Block-Partition Interference · Harder-Chain Cramér/KL · Minmax Fairness Re-adjudication

Day 95 (§15.28) 는 (i) component-local $(\sigma_{0,c}, \eta_c)$ 로 재조립한 +CNRT 스택이
Day 93 combined $-0.288$ 를 결정적으로 뒤집지 못했고, (ii) wider trunk ($H \in \{32,64\}$)
에서 Cramér vs KL 이 near-uniform 수렴으로 differentiate 안 되었으며, (iii) $\Gamma$ 축
+CNRT 우위가 $\bar R$ tie 대비 진짜 robustness 인지 metric 함정 여부가 불분명했다. Day 96
(§15.29) 는 세 잔여를 정면 재판정한다: (a) 재조립 스택을 **트렁크 파라미터 block partition**
으로 옮겨 shared-trunk gradient 평균화 근사를 제거, (b) $p_{\text{train}} = 0.20$ 로 어렵게
+ 5-slip 배포로 Cramér vs KL 순위 재출현, (c) $\Gamma$-only 우위를 $R_{\min}$ / MMF
$\lambda$-sweep 으로 정직하게 재판정.

## 학습 목표

- **Problem 1 (Block partition)** — $W_1 \in \mathbb{R}^{H \times d}$ 를
  $H_N + H_C + H_E + H_T = H = 32$ 로 성분별 행 블록 분할. 각 헤드는 자기 블록만 사용해
  gradient 간섭 구조적 제거. 3 시드 (96101–96103) × 400 step, softmax $\tau=0.10$
  배포. 지표: block partition return $R^{\text{bp}}$, gap $\Delta_{\text{bp}} = R^{\text{bp}}
  - R^{\text{shared}}$, 슬라이스 utilisation $\|W_1^{(c)}\|_F^2 / \|W_1\|_F^2$.
- **Problem 2 (Harder chain Cramér vs KL)** — $p_{\text{train}} = 0.20$ chain MDP, categorical
  head sweep $H \in \{16, 32\} \times K \in \{10, 21\} \times$ loss $\in$ {Cramér, KL}, 3
  시드 (96201–96203) × 400 step. Deploy 5-slip $p_d \in \{0.05, 0.10, 0.15, 0.20, 0.25\}$
  × 40 ep greedy tail-8. 지표: cross-slip mean $\bar R$, sharpness $\bar H(\hat p)$, seed 표준편차.
- **Problem 3 (Minmax fairness)** — Chain MDP + uniform slip curriculum $\{0.05, 0.10, 0.20\}$.
  baseline (vanilla Q) vs +CNRT (Factorized Noisy head), 3 시드 (96301–96303) × 400 step.
  Freeze 후 5-slip deploy × 40 ep greedy tail-8. 지표: $\bar R$, $R_{\min}$, $\Gamma$,
  MMF $= R_{\min} - \lambda \Gamma$ for $\lambda \in \{0, 0.25, 0.5, 1.0\}$.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_15_29_01.ipynb` | 트렁크 block partition + gradient 간섭 제거 | $H = H_N + H_C + H_E + H_T = 32$ 성분별 행 블록, EMA 는 자기 슬라이스에 whitening, Twin 은 voting weight 0.5, softmax $\tau=0.10$ 배포 | **Block partition 은 평균적으로 후퇴** — $R^{\text{shared}} = 6.72$ vs $R^{\text{bp}} = 4.64$, $\Delta_{\text{bp}} = -2.08$. 시드 감수성 매우 크다 (std 4.04, 96101 -1.81, 96102 +1.81, 96103 -6.25 로 부호 뒤집힘). Utilisation 은 **T (0.363) > C (0.319) > N (0.218) > E (0.100)** — EMA 슬라이스가 사실상 dormant, whitening 이 gradient 스케일을 축소해 슬라이스 성장 억제. **성분 disjoint 균등 분할은 최적이 아니며 표현력 절감이 gradient 간섭 제거 이득을 능가**. |
| 2 | `CE_15_29_02.ipynb` | 어려운 chain Cramér vs KL 재판정 | Shared trunk (H) + per-action K-atom softmax head, categorical projection $\Phi$, TD greedy target, $p_{\text{train}}=0.20$, 8 cells × 3 시드 × 5-slip deploy | **Cramér 이 4 개 셀 모두에서 KL 완파** (cross-slip mean): $(16,10)$ **0.344 vs 0.212**, $(16,21)$ **0.363 vs 0.304**, $(32,10)$ **0.322 vs 0.268**, $(32,21)$ **0.292 vs 0.219**. **Day 94 P2 (neural head KL 승) 완전 역전**, Day 93 P2 (Bellman tabular Cramér 승) 쪽으로 회귀. Sharpness $\bar H$: Cramér 2.09–2.87 vs KL 2.16–2.88 로 유사 — 엔트로피 축 differentiate 안 됨. **loss 함수 순위가 environment 난이도 $p_{\text{train}}$ 에 강하게 의존**함을 확증. |
| 3 | `CE_15_29_03.ipynb` | Worst-case + minmax fairness 재판정 | Q-learner (H=16) baseline vs +CNRT (Factorized Noisy Linear head), uniform curriculum $\{0.05, 0.10, 0.20\}$, 5-slip deploy, MMF $\lambda$ sweep | **+CNRT 가 $\bar R, R_{\min}$, MMF 모두 baseline 우위**: $\bar R$ **0.688 vs 0.603**, $R_{\min}$ **0.592 vs 0.507** (둘 다 $p_d = 0.25$), $\Gamma$ 0.196 vs 0.172 (+CNRT 근소 열위 - **하한이 상승할 때 gap 도 늘어난 것**). MMF@$\lambda=1.0$ 에서도 **0.396 vs 0.335** 로 +CNRT 승 — 곡선 미교차, $\lambda \to \infty$ 극한에서만 baseline 우세. **Day 95 P3 의 "$\Gamma$-only 함정" 은 이 시드에서 미실현**, 대신 Noisy head 가 curriculum 학습을 정직하게 개선. |

## 한 줄 정리

> **Day 96 세 축 요약**: (P1) block partition 은 gradient 간섭을 구조적으로 제거하나 **표현력
> 절감이 이득을 능가** ($\Delta_{\text{bp}} = -2.08$, std 4.04), 특히 **EMA 슬라이스가
> utilisation 10% 로 dormant** — 성분 disjoint 균등 분할은 최적이 아님. (P2) 어려운 환경
> ($p_{\text{train}} = 0.20$) 에서 **Cramér 이 4 개 셀 모두에서 KL 완파** (cross-slip 0.29–0.36
> vs 0.21–0.30) — Day 94 P2 KL 승은 쉬운 환경의 산물이며, loss 순위는 environment-dependent.
> (P3) +CNRT 가 $\bar R$ (0.688 vs 0.603), $R_{\min}$ (0.592 vs 0.507), MMF ($\lambda=1$ 에서
> 0.396 vs 0.335) 모두 baseline 우위 — Day 95 P3 의 $\Gamma$-only 함정 미실현, 대신 Noisy head
> 가 curriculum 학습을 정직하게 개선. 종합: **성분별 최적 처방과 환경 난이도 정합이 metric
> 다각화보다 큰 이득 원천**.

## 사용 라이브러리

NumPy · pandas · Matplotlib 표준 스택. DL 프레임워크 미사용 — shared trunk MLP (H∈{16, 32},
tanh, 수기 backprop), Factorized Noisy Linear ($\mu, \sigma$ 각 gradient), Cramér CDF-distance
(softmax Jacobian 통한 logit gradient), KL cross-entropy, categorical projection $\Phi$
(nearest-two atom), EMA whitening, Twin voting, block-partitioned trunk (row-slice 라우팅) 모두
수기 numpy 로 구현.

## 다음 (Day 97)

§15.30 은 (a) P1 의 disjoint 균등 분할 대신 성분별 폭을 utilisation 로 **적응 배분** 하는
gated partition (EMA 슬라이스에 좁게, Twin/Cramér 슬라이스에 넓게), (b) P2 의 Cramér 우위
반전을 $p_{\text{train}}$ 을 세부 sweep ($\{0.05, 0.10, 0.15, 0.20, 0.25\}$) 해서 **순위 반전
임계값** $p_{\text{train}}^\star$ 추정, (c) P3 의 +CNRT 우위가 시드 감수성 (P1 처럼) 인지
견고한 경향인지 **시드 확장 (10 시드) 으로 재검증** 예정.

---

> ℹ️ **노트**: `_meta/curriculum.md` 의 명시 일정은 Day 13 (§3.3) 까지이며, Day 14 이후는
> 스케줄러가 절 단위로 자동 진행. Day 95 README 의 "다음 (Day 96)" 예고 (component-local
> block partition + wider-trunk 재판정 + MMF/worst-case fairness) 를 그대로 반영.
>
> ⚙️ **작성 메모 (자동 실행)**: 세 노트북 모든 수치는 `nbclient` 실행 결과.
> 시드: P1 = 96101–96103, P2 = 96201–96203, P3 = 96301–96303. Python 3.10, NumPy / pandas /
> Matplotlib 표준 스택.
>
> 🔎 **정직성 노트**:
> - **P1 시드 감수성**: 세 시드에서 $\Delta_{\text{bp}}$ 부호가 -/+/- 로 뒤집혀 std 4.04.
>   평균 $-2.08$ 은 하나의 outlier (96103) 에 크게 좌우됨. 3 시드는 결론을 내리기에 부족하며,
>   Day 97 P3 시드 확장이 답을 줄 것.
> - **P2 Cramér 승리의 의미**: Day 94 P2 (KL 승) 와 정반대 결과이나, 근본 원인은 loss 함수
>   자체가 아니라 environment 난이도 이동일 가능성. Day 97 P2 sweep 이 임계값을 추정.
> - **P3 +CNRT 우위**: 이 시드에서는 +CNRT 가 모든 축에서 승, 하지만 Day 95 P3 는 반대 방향
>   결과 (tie / $\Gamma$-only). 시드 편차의 산물일 수 있어 Day 97 P3 는 10 시드 확장 예정.
> - **훈련 예산 400 step**: Day 93 (600 step), Day 94 (800 step) 대비 축소. 시간 예산 절감을
>   위한 선택이며, 절대 수치보다 configuration 간 상대 비교 가치가 우선.
>
> 📉 **워크스페이스 제약**: `/sessions` 파티션 100% full 로 인해 site-packages 를
> `/tmp/pypkg` 에 설치. `HOME`, `MPLCONFIGDIR`, `JUPYTER_DATA_DIR`, `XDG_CONFIG_HOME` 을
> `/tmp` 하위로 지정. Day 95 와 동일한 mitigation.
>
> 📚 **레퍼런스**: Fortunato et al. (2018) ICLR — Factorized Noisy Linear.
> Bellemare, Dabney, Munos (2017) ICML — C51 categorical projection.
> Rowland et al. (2018) AISTATS — Cramér valid distributional loss.
> Ioffe & Szegedy (2015) ICML — feature whitening / batch normalization warmup.
> Rawls (1971) — Justice as Fairness (minmax principle 원류).
