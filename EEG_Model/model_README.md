# 💤 SleepBot: EEG 기반 수면 단계 분류 시스템

SleepBot은 단일 체림 EEG(Pz-Oz)를 이용하여 수면 단계를 분류하고,
웹 기반으로 시각화 및 피드래프을 제공하는 **디플러링 + 웹 통합 수면 분석 플랫폼**입니다.

---

## 📌 프로젝트 요약

| 항목             | 설명                        |
| -------------- | ------------------------- |
| 📅 EDF 업로드     | 사용자 EEG 수면 파일 (.edf) 업로드  |
| 🧠 수면 단계 예측    | CNN+BiLSTM 기반 수면 단계 분류    |
| 📊 결과 시각화      | 각 수면 단계 비율 + 이상적 수면 비율 비교 |
| 💡 피드백 시스템     | 부족한 단계에 대한 원인 분석 및 보완 제안  |
| 👤 계정 기반 분석 기록 | 회원가입, 로그인, 분석 리포트 저장      |
| 📈 점수 추이 시각화   | 수면 점수 변화를 선그래프로 표현        |

---

## 🧬 모델 구조

* **입력 형태**: (B, 1, T)
* **모델 구조**:

  * Conv1D → BatchNorm → ReLU → MaxPool (×3)
  * Bidirectional LSTM (2-layer, 256 hidden)
  * Dropout → Fully Connected(5-class)

```text
   입력 (B, 1, T)
        │
    [Conv1D 1→32]
        ↓
    [BatchNorm + ReLU]
        ↓
    [MaxPool1d]
        ↓
    [Conv1D 32→64]
        ↓
    [BatchNorm + ReLU]
        ↓
    [MaxPool1d]
        ↓
    [Conv1D 64→128]
        ↓
    [BatchNorm + ReLU]
        ↓
    [MaxPool1d]
        ↓
    ┌─────────────────────────────┐
    │  Bidirectional LSTM × 2    │
    │   hidden=256, dropout=0.5  │
    └─────────────────────────────┘
        ↓
    [Dropout + FC (5-class)]
        ↓
     수면 단계 (0~4 클래스)
```

📁 정의 위치: `model_architecture.py`

---

## 🧪 전처리 흐름

* 채널 선택: EEG Pz-Oz
* 필터링: FIR 필터 (0.5–30Hz) 적용
* 윈도우 분할: 30초 단위, 15초 stride 슬라이딩
* 주기 추출: FFT 기반 PSD peak → cycle 계산
* 정규화: 윈도우별 Z-score

📁 관련 구현: `process_edf()` in `app.py`

---
