# Day 67 — §14.11 Pipeline Integration & Operations (온라인 학습 파이프라인의 운영·통합)

Day63–65(§14.7–14.9)에서 **학습률 손잡이**를 편향-분산 축으로 조율했고, Day66(§14.10)에서
그 조율 자체를 **자동화**했다. 이제 §14.11 은 마지막 한 걸음 — 이 모든 조각을 **하나의 운영 파이프라인**으로
묶는다. 세 방어층으로 본다: **감지**(loss 분포 변화), **안전 배포**(champion-challenger + SPRT + rollback),
그리고 두 층을 결합한 **종단 파이프라인**.

$$\text{자동 튜닝}\ \xrightarrow{\text{감지 트리거}}\ \text{Drift Detector}
\ \xrightarrow{\text{shadow + SPRT gate}}\ \text{Safe Deployment}
\ \xrightarrow{\text{PH 모니터}}\ \text{Auto-Rollback}.$$

> **공통 축**: §14.7–14.10 과 똑같이 **편향-분산·리스크**다. 파이프라인 통합이란 결국 이 U자의 좋은 지점을
> **감지·게이팅·롤백** 세 방어층으로 **실전 리스크 아래에서** 유지하는 일이다.

## 학습 목표
- **Page-Hinkley** 와 **ADWIN-style** 두 온라인 drift 검출기를 구현·비교하고, **민감도-지연 상충**을 파라미터 $\delta$(PH)·창 크기(ADWIN)로 정량화한다. Abrupt vs Gradual 시나리오에서 두 알고리즘의 강점을 확인한다.
- **Champion-Challenger** + **SPRT gating** + **PH-based auto-rollback** 프로토콜을 구현해, 잘못된 승격의 **평균 손실**과 **회귀 리스크**를 always-promote baseline 대비 정량 비교한다. 승격 후에도 언제든지 되돌릴 수 있는 **다이나믹 방어**를 검증한다.
- **감지·게이팅·롤백**을 §14.8 적응 학습률·§14.10 자동 튜닝 위에 얹은 **End-to-end 파이프라인** 을 다중 레짐 스트림에서 4개 구성(NoMonitor / DetectOnly / CCOnly / Full)과 비교해, 세 방어층이 서로 다른 실패 모드를 어떻게 상쇄하는지 관찰한다.

## 문제 목록

| # | 노트북 | 주제 | 핵심 기법 | 핵심 결과 |
|---|--------|------|-----------|-----------|
| 1 | `CE_14_11_01.ipynb` | Concept Drift Detection (PH + ADWIN) | Page-Hinkley $m_t - M_t$, ADWIN 호에프딩 컷 | Abrupt jump: **PH mean_delay ≈ 15** ($\delta{=}0.01$), ADWIN($n_{\max}{=}500$) **≈ 89**. Gradual ramp: PH 136, ADWIN($n_{\max}{=}1000$) 441. 두 검출기 모두 **FAR=0**(null 스트림)에서 안정. 민감도-지연 상충은 $\delta$·창 크기로 조절 |
| 2 | `CE_14_11_02.ipynb` | Safe Deployment (Shadow + SPRT + Rollback) | Wald SPRT, PH-based rollback | `cha_better`: safe promote **97%**, regret ratio **0.763**. `cha_worse`: safe reject **100%**, regret ratio **0.720**. `cha_regress`: 승격 후 회귀 발생, **auto-rollback 100%**, regret ratio **0.534** — 세 방어층이 세 실패 모드 모두 방어 |
| 3 | `CE_14_11_03.ipynb` | End-to-End Integrated Pipeline | RMSProp + PH + LR range test + SPRT + rollback | 다중 레짐 스트림(스위치 $t{=}1500, 3000$). NoMonitor **cum_loss 430**, DetectOnly **539**(과잉 스위칭), CCOnly **430**, **Full 344** (최저). Full 이벤트가 실제 change point ($t{=}1500, 1526, 3003$)에 정확히 정렬 |

## 한 줄 정리
> **파이프라인 통합 = "감지 + 게이팅 + 롤백"** — 개별 부품(§14.7–14.10)을 이 세 방어층으로 묶으면
> 각 부품의 실패 모드를 서로 상쇄하며 하나의 편향-분산-리스크 이야기의 세 얼굴이 완성된다.
> 다중 레짐 스트림에서 Full 파이프라인만이 개별 baseline 각각의 취약점을 모두 방어한다.

## 다음 Day 예고
§14.11 로 **온라인 학습 사례연구** 챕터(Day57–67)를 마감했다. 다음 Day 는 (a) 정규 교재 절 순서로 복귀 —
`_meta/curriculum.md` 확장 후 Ch 4 (보간) 이후 절로, (b) 다른 응용 챕터로의 확장 — Ch 12 (PDE) 또는
Ch 13 (최적화) — 중 하나로 이어 갈 수 있다.

---
> ℹ️ **노트**: `_meta/curriculum.md` 의 명시적 일정은 Day 13(§3.3)까지이며, Day14 이후는 스케줄러가
> 로드맵을 절 단위로 자동 진행해 왔다. Day57(§14.1)부터 **응용 사례연구(온라인 학습) 챕터**가 이어졌고,
> 본 Day67 은 Day66 README 예고("전체 파이프라인 통합·운영 체크리스트")에 따라 **§14.11** 로 진도를
> 이은 것이다. 정규 교재 절 순서를 다시 잡으려면 `_meta/curriculum.md` 를 확장하면 된다.
>
> ⚙️ **작성 메모(자동 실행)**: 세 노트북의 모든 수치(PH mean_delay 15·136, ADWIN 89·441, safe promote 97%,
> regret ratio 0.534·0.720·0.763, Full cum_loss 344, 이벤트 타임라인 $t{=}1500,1526,3003$)는 표준 CPython
> (nbclient/kernel 미사용, 셀별 exec + 캡처 렌더러) 로 실제 실행해 얻은 값이며, 본문 해석과 일치하도록 검증했다.
> 특히 Problem 2 는 초안(rollback 이 3.4% 로만 발동)이 rollback window R=1500 < regression 시점(2000)이라
> 창 바깥에서 발생했음을 확인하고, R=3000 (스트림 끝까지 감시) 로 재정의해 **rollback recall 100%** 를 확인·재실행했다.
