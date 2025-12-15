# Bitcoin Price Prediction & Algorithmic Trading with PatchTST

## 1. Project Overview
본 프로젝트는 비트코인(BTC)의 가격 변동성을 예측하고, 단순 보유(Buy & Hold) 대비 초과 수익을 달성하기 위한 딥러닝 기반 트레이딩 시스템 구축을 목표로 합니다.
기존 시계열 모델의 한계를 극복하기 위해 최신 **PatchTST (Patch Time Series Transformer)** 아키텍처를 도입하였으며, 모델의 예측 신호와 기술적 분석(이동평균선)을 결합한 **하이브리드 트레이딩 전략**을 통해 하락장에서의 손실을 방어하고 수익을 극대화하는 성과를 입증했습니다.

## 2. Methodology

### 2.1 Model Architecture: PatchTST
시계열 데이터를 이미지 패치처럼 처리하여 장기 의존성(Long-term dependency)을 학습하고 연산 효율을 높인 PatchTST 모델을 구축했습니다. 과적합(Overfitting)이 심한 금융 데이터 특성을 고려하여 모델을 경량화했습니다.

* **Structure:** Patch Embedding $\rightarrow$ Transformer Encoder $\rightarrow$ Flatten $\rightarrow$ Binary Classification Head
* **Key Parameters:**
    * `seq_len`: 60 (과거 60일 데이터 입력 30일로 진행했을 시에 결과값이 좋지 않아 적정값을 찾아보았습니다.)
    * `patch_len`: 10 / `stride`: 5 (50% Overlap Patching 적용)
    * `d_model`: 24 (모델 복잡도 최소화)
    * `num_layers`: 1 (얕은 신경망으로 일반화 성능 확보)
    * `dropout`: 0.6 (강력한 규제를 통한 과적합 방지)
    * `optimizer`: Adam (lr=0.0005)

### 2.2 Trading Strategy (Hybrid Approach)
단순히 모델의 예측값만 따르는 것이 아니라, 추세 추종(Trend Following) 개념을 결합하여 가짜 신호(False Signal)를 필터링했습니다.

1.  **Basic Strategy:** 모델 예측 확률 > 0.5 (단순 매수)
2.  **Confidence Strategy:** 모델 예측 확률 > 0.55 (확신 시 매수)
3.  **Hybrid Strategy (Proposed):**
    * **Condition A:** 모델 예측 확률 > 0.5 (상승 예측)
    * **Condition B:** 현재 가격 > 20일 이동평균선(MA20) (상승 추세)
    * $\rightarrow$ **Action:** A와 B를 모두 만족할 때만 진입, 그 외에는 현금 보유 (Risk Free)

## 3. Experimental Results

### 3.1 Model Performance (Test Dataset)
과적합을 억제한 결과, 테스트 셋에서 안정적인(Generalization) 성능을 확보했습니다. 재현율(Recall)이 낮은 것은 모델이 확실한 상승 구간에서만 보수적으로 예측함을 의미합니다.

| Metric | Value |
| :--- | :--- |
| **Accuracy** | 51.91% |
| **Precision** | 52.43% |
| **Recall** | 29.83% |
| **F1 Score** | 0.3803 |

### 3.2 Backtesting Profitability (Key Achievement)
테스트 기간 동안 실제 트레이딩 시뮬레이션을 진행한 결과(수수료 0.1% 적용), 제안한 **하이브리드 전략**이 압도적인 성과를 기록했습니다.

* **시장 상황:** 테스트 기간 동안 비트코인 가격은 하락세(**-13.02%**)를 기록했습니다.
* **전략 성과:** 하이브리드 전략은 하락장에서 매매를 멈추고 현금을 보유하여 손실을 방어했으며, 확실한 상승 추세에서만 수익을 누적하여 **+144.85%**의 수익률을 달성했습니다.

| Strategy | Cumulative Return (%) | Trade Count | Note |
| :--- | :---: | :---: | :--- |
| **Buy & Hold** | **-13.02%** | 1 | 시장 수익률 (Benchmark) |
| Basic Strategy | -39.70% | 366 | 잦은 매매로 인한 손실 |
| High Confidence | -38.79% | 344 | - |
| **Hybrid Strategy** | **+144.85%** | **172** | **Best Performance** 🏆 |

> **Conclusion:** PatchTST 모델 단독 사용 시 정확도는 51.9% 수준이었으나, 이동평균선(MA20) 필터를 결합함으로써 시장 대비 **+157.86%p**의 초과 수익을 달성했습니다. 이는 AI 예측 모델을 보조 지표와 결합했을 때 실전 투자에서의 효용성이 극대화됨을 시사합니다.
