<div align="center">

# 🩺 RF-DETR Endoscopy

### Real-time Polyp Detection Desktop Application

**대장 내시경 영상에서 용종을 실시간 탐지하고, 탐지 결과를 실제 사용 가능한 데스크톱 워크플로우로 연결한 의료 AI 프로젝트입니다.**

![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)
![RF-DETR](https://img.shields.io/badge/Model-RF--DETR-7C3AED?style=flat-square)
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=opencv&logoColor=white)
![PyQt5](https://img.shields.io/badge/GUI-PyQt5-41CD52?style=flat-square&logo=qt&logoColor=white)

`Real-time Detection · Video Processing · Event Clip · AI Report · Desktop App`

</div>

---

## Why I built it

의료영상 AI 프로젝트에서 모델의 탐지 성능만 확인하는 것으로는 실제 사용 흐름을 충분히 보여주기 어렵다고 생각했습니다.

그래서 이 프로젝트에서는 **“용종을 찾는 모델”에서 끝내지 않고, 탐지 결과가 실제 애플리케이션에서 어떻게 사용되는지**까지 구현했습니다.

```text
Endoscopy / Webcam
        ↓
RF-DETR Inference
        ↓
Real-time Bounding Box
        ↓
Detection Event
        ├─ Highlight Clip
        ├─ Patient Metadata
        └─ Preliminary AI Report
```

핵심 목표는 **AI 추론 결과를 사용자가 바로 확인하고 기록할 수 있는 워크플로우로 연결하는 것**이었습니다.

---

## Project overview

건양대학교 창의적 종합설계 프로젝트로 진행한 **RF-DETR 기반 실시간 의료 객체 탐지 데스크톱 애플리케이션**입니다.

내시경 영상과 웹캠 스트림을 입력으로 받아 용종을 실시간으로 탐지하고, PyQt5 GUI에서 결과를 시각화합니다.

추가로 연속 탐지 이벤트를 기준으로 하이라이트 영상을 저장하고, 선택적으로 AI 예비 보고서를 생성할 수 있도록 구성했습니다.

---

## Architecture

```text
┌──────────────────┐
│ Video Input      │
│ Webcam / File    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Inference Worker │
│ RF-DETR          │
└────────┬─────────┘
         │
         ├── Bounding Box
         │
         ▼
┌──────────────────┐
│ PyQt5 UI         │
│ Real-time View   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Event Detection  │
│ Continuous Polyp │
└──────┬─────┬─────┘
       │     │
       │     └──── AI Preliminary Report
       │
       └────────── Highlight Video Clip
```

GUI와 추론 처리는 `QThread` 기반으로 분리하여 영상 추론 중에도 UI가 멈추지 않도록 구성했습니다.

---

## Key features

### 1. Real-time polyp detection

- 웹캠 실시간 스트림 지원
- 로컬 영상 파일(`.mp4`, `.avi` 등) 지원
- RF-DETR 기반 추론
- 실시간 Bounding Box 렌더링
- QThread 기반 비동기 처리

### 2. Automatic highlight clip

용종이 일정 시간 이상 연속으로 탐지되면 하나의 이벤트로 판단합니다.

기본 설정:

```text
Continuous detection trigger: 4 sec
Pre / Post event buffer:      5 sec
```

이벤트 전후 구간을 포함한 하이라이트 영상을 자동 저장합니다.

### 3. AI preliminary report

선택적으로 OpenAI API를 연결해 이벤트 발생 시 다음 데이터를 기반으로 예비 요약 보고서를 생성합니다.

- 환자 정보
- 탐지 시점
- confidence
- 이벤트 정보

보고서는 하이라이트 클립과 함께 저장됩니다.

> 생성 결과는 의료진의 최종 판단을 대체하는 진단 결과가 아니라 프로젝트 내 보조 기능입니다.

### 4. Patient information management

- 이름 / 나이 / 성별 / 생년월일 입력
- 화면 및 저장 영상에 정보 overlay
- 환자 단위 데이터 폴더 관리
- CSV / Excel 기반 기록 확장 가능

### 5. Manual recording & snapshot

- 전체 영상 수동 녹화
- 중요 순간 snapshot 저장
- 환자별 결과 파일 관리

---

## Tech stack

| Area | Stack |
| --- | --- |
| Language | Python 3.x |
| Desktop GUI | PyQt5 |
| Computer Vision | OpenCV |
| Detection | RF-DETR / Roboflow Inference SDK |
| Generative AI | OpenAI API — optional report generation |
| Data Export | CSV · openpyxl |

---

## Installation

```bash
pip install PyQt5 opencv-python inference-sdk openpyxl openai
```

---

## Environment variables

```text
ROBOFLOW_API_KEY=...
OPENAI_API_KEY=...
RFDETR_CONFIDENCE=0.3
RFDETR_CLIP_TRIGGER=4
RFDETR_ENABLE_REPORTS=0
```

| Variable | Purpose |
| --- | --- |
| `ROBOFLOW_API_KEY` | Roboflow inference 인증 |
| `OPENAI_API_KEY` | AI report 활성화 시 사용 |
| `RFDETR_CONFIDENCE` | Detection confidence threshold |
| `RFDETR_CLIP_TRIGGER` | 연속 탐지 이벤트 기준 시간 |
| `RFDETR_ENABLE_REPORTS` | AI report 활성화 여부 |

---

## Run

```bash
python RFDETR_v3.py
```

---

## What this project focuses on

이 프로젝트에서 중요하게 본 부분은 단순 모델 호출이 아니라 다음 연결 과정입니다.

```text
AI inference
→ asynchronous application processing
→ event definition
→ video artifact generation
→ user-facing workflow
```

즉, **Computer Vision 결과를 실제 소프트웨어 기능으로 연결하는 과정**을 구현한 프로젝트입니다.

---

<div align="center">

**From detection results to a usable medical imaging workflow.**

</div>
