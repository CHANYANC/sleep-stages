# 💤 EEG 기반 수면 단계 분류 모델

**Zzz Lab | 2025년 1학기 인공지능 프로젝트**
단일 체림 EEG(Pz-Oz)를 이용하여 수면 단계를 분류하고,
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

## 📊 수면 분석 리포트

분석 결과는 다음과 같이 구성됩니다:

* 예측된 수면 단계 비율 도넛 차트
* N1/N2/N3/REM 단계별 수면 비중
* 각 단계별 실제 vs 이상 비율 막대 그래프
* 수면 점수 (0\~100점)
* 자동 피드백 (N3 부족, REM 미감지 등)

피드백 생성 기준은 `feedback.py`에 정의되어 있으며,
예측 비율에 따라 조건부로 메시지가 동적으로 출력됩니다.

---

## 🌐 웹 시스템 기능

* 📤 EDF 업로드 후 자동 분석
* 📋 리포트 DB 저장 (사용자별 구분)
* 📈 수면 점수 변화 시각화 (Plotly 기반 선그래프)
* 🔐 Flask-Login 기반 로그인 / 회원가입
* 🧑 사용자별 분석 내역 조회 및 상세 리포트

> 주요 실행 경로: `/`, `/result`, `/score_chart`, `/reports`

---
