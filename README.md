<div align="center">

# 🩺 RF-DETR Endoscopy

### Real-time Polyp Detection Desktop Application

**대장 내시경 영상에서 용종을 실시간 탐지하고, 탐지 결과를 실제 사용 가능한 데스크톱 워크플로우로 연결한 의료 AI 프로젝트입니다.**

![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)
![RF-DETR](https://img.shields.io/badge/Model-RF--DETR-7C3AED?style=flat-square)
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=opencv&logoColor=white)
![PyQt5](https://img.shields.io/badge/GUI-PyQt5-41CD52?style=flat-square&logo=qt&logoColor=white)

`Real-time Detection · Video Processing · Event Clip · Patient Workflow · Desktop App`

</div>

---

## 1. Why I built it

의료영상 AI 프로젝트를 진행하면서 **모델의 탐지 정확도만 보여주는 것으로는 실제 사용성을 설명하기 부족하다**고 느꼈습니다.

용종을 찾는 모델이 있어도 사용자가 실제로 쓰려면 다음 과정이 필요합니다.

```text
영상 입력
  ↓
AI 추론
  ↓
실시간 화면 표시
  ↓
의미 있는 탐지 이벤트 정의
  ↓
결과 저장
  ↓
환자 단위 기록
  ↓
사용자가 다시 확인할 수 있는 artifact
```

그래서 이 프로젝트에서는 모델 호출에서 끝내지 않고, **탐지 결과가 실제 데스크톱 애플리케이션의 기능이 되는 과정**까지 구현했습니다.

> **Detection model → usable workflow**

이 연결이 이 프로젝트의 핵심입니다.

---

## 2. Problem definition

실시간 의료영상 애플리케이션을 만들기 위해 해결해야 했던 문제는 크게 네 가지였습니다.

### 2.1 Real-time inference without freezing the UI

영상 추론은 연산량이 크기 때문에 GUI thread에서 그대로 수행하면 화면이 멈출 수 있습니다.

### 2.2 A detection frame is not an event

한 프레임에서 bounding box가 나타났다고 바로 임상적으로 의미 있는 이벤트라고 볼 수 없습니다. 연속 탐지를 시간 기준으로 묶어야 했습니다.

### 2.3 Detection needs context

탐지된 순간만 저장하면 전후 상황을 확인하기 어렵습니다. 이벤트 전후 영상까지 함께 남겨야 했습니다.

### 2.4 Results must be organized for a user

단순 이미지 저장이 아니라 환자 정보, 탐지 시점, confidence, 영상 clip을 하나의 작업 흐름으로 묶어야 했습니다.

---

## 3. How it works

```text
Endoscopy / Webcam
        ↓
Video Capture
        ↓
RF-DETR Inference Worker
        ↓
Bounding Box + Confidence
        ↓
Real-time PyQt5 View
        ↓
Continuous Detection Logic
        ↓
Detection Event
   ┌────┼─────────────┐
   │    │             │
   ▼    ▼             ▼
Clip  Patient     Optional AI
Save  Metadata    Preliminary Report
```

---

## 4. Architecture

```text
┌────────────────────┐
│ Video Input        │
│ Webcam / File      │
└──────────┬─────────┘
           │
           ▼
┌────────────────────┐
│ Capture / Buffer   │
│ OpenCV             │
└──────────┬─────────┘
           │
           ▼
┌────────────────────┐
│ Inference Worker   │
│ RF-DETR            │
│ QThread            │
└──────────┬─────────┘
           │
           ├── Bounding Box
           ├── Confidence
           │
           ▼
┌────────────────────┐
│ PyQt5 UI           │
│ Real-time Display  │
└──────────┬─────────┘
           │
           ▼
┌────────────────────┐
│ Event Logic        │
│ Continuous Detect  │
└───────┬─────┬──────┘
        │     │
        │     └──── Optional Preliminary Report
        │
        └────────── Highlight Clip + Metadata
```

---

## 5. Key design decisions

### 5.1 QThread for inference

GUI와 AI 추론을 같은 thread에서 처리하지 않고 `QThread` 기반 worker로 분리했습니다.

목표는:

```text
AI inference latency
≠
UI freeze
```

가 되도록 만드는 것이었습니다.

### 5.2 Continuous detection instead of single-frame trigger

단일 프레임 detection은 노이즈일 수 있기 때문에 일정 시간 이상 연속 탐지될 때 이벤트로 간주합니다.

기본 설정:

```text
Continuous detection trigger: 4 sec
```

이 로직을 통해 일시적인 bounding box보다 **지속적으로 관찰된 탐지 결과**를 artifact 생성 기준으로 사용합니다.

### 5.3 Pre / post event context

이벤트가 발생한 정확한 프레임만 남기지 않고 전후 영상을 포함한 clip을 저장합니다.

기본값:

```text
Pre / Post event buffer: 5 sec
```

이렇게 하면 사용자가 탐지 직전과 직후 상황을 함께 확인할 수 있습니다.

### 5.4 AI report is optional

생성형 AI 예비 보고서는 핵심 탐지 pipeline과 분리했습니다.

모델/API 문제가 생겨도 실시간 detection과 clip 저장은 독립적으로 동작할 수 있도록 하고, report는 선택 기능으로 두었습니다.

---

## 6. Core features

### Real-time polyp detection

- webcam 실시간 입력
- local video file 입력
- RF-DETR 기반 object detection
- bounding box / confidence 렌더링
- QThread 기반 inference worker

### Automatic highlight clip

- 연속 탐지 시간을 event trigger로 사용
- event 전후 frame context 저장
- 탐지 artifact 자동 생성

### Patient information management

- 이름
- 나이
- 성별
- 생년월일
- 환자 단위 결과 폴더
- 화면/영상 metadata overlay

### Optional preliminary AI report

환경변수로 활성화했을 때 다음 정보를 바탕으로 예비 요약을 생성할 수 있습니다.

- patient metadata
- detection timestamp
- confidence
- event context

> 이 보고서는 의료진의 진단을 대체하지 않는 프로젝트 보조 기능입니다.

### Manual recording / snapshot

- 전체 영상 수동 녹화
- 중요 순간 snapshot 저장
- 환자 단위 결과 관리

---

## 7. End-to-end workflow

```text
1. 환자 정보 입력
      ↓
2. Webcam / Endoscopy Video 선택
      ↓
3. RF-DETR 실시간 추론
      ↓
4. Bounding Box 화면 표시
      ↓
5. 연속 탐지 event 확인
      ↓
6. 전후 context 포함 highlight clip 생성
      ↓
7. 환자별 artifact 저장
      ↓
8. Optional AI preliminary report
```

이 구조를 통해 모델 결과를 사용자가 다시 확인할 수 있는 **영상 artifact와 기록**으로 연결했습니다.

---

## 8. Tech stack

| Area | Stack |
| --- | --- |
| Language | Python 3.x |
| Desktop GUI | PyQt5 |
| Computer Vision | OpenCV |
| Detection | RF-DETR / Roboflow Inference SDK |
| Concurrency | QThread |
| Generative AI | OpenAI API — optional |
| Data Export | CSV · openpyxl |

---

## 9. Why this stack?

### Python

모델 inference와 OpenCV 영상 처리 생태계를 빠르게 연결하기 위해 사용했습니다.

### PyQt5

실시간 영상, 환자 정보, recording control을 하나의 desktop UI에서 다루기 위해 선택했습니다.

### OpenCV

video capture, frame processing, overlay, clip generation을 담당합니다.

### RF-DETR

대장 내시경 용종 object detection을 위한 핵심 detector로 사용했습니다.

### QThread

추론 작업과 UI event loop를 분리해 사용성을 유지하기 위해 사용했습니다.

---

## 10. Repository focus

이 프로젝트에서 중요한 부분은 단순히 model file을 호출하는 것이 아니라 다음 흐름입니다.

```text
Computer Vision
      ↓
Async Application Processing
      ↓
Event Definition
      ↓
Artifact Generation
      ↓
User Workflow
```

즉 **AI 결과를 애플리케이션 기능으로 제품화하는 과정**을 경험한 프로젝트입니다.

---

## 11. Installation

```bash
pip install PyQt5 opencv-python inference-sdk openpyxl openai
```

---

## 12. Environment variables

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
| `OPENAI_API_KEY` | optional report 생성 시 사용 |
| `RFDETR_CONFIDENCE` | detection confidence threshold |
| `RFDETR_CLIP_TRIGGER` | 연속 탐지 event 기준 시간 |
| `RFDETR_ENABLE_REPORTS` | preliminary report 활성화 여부 |

---

## 13. Run

```bash
python RFDETR_v3.py
```

지원 입력:

- webcam
- local video (`.mp4`, `.avi` 등)

---

## 14. Verification mindset

실시간 AI 애플리케이션에서는 모델 accuracy뿐 아니라 전체 pipeline이 동작하는지 확인해야 합니다.

이 프로젝트에서 확인해야 할 흐름:

```text
Video capture
→ inference worker
→ UI rendering
→ continuous event trigger
→ clip generation
→ patient folder / metadata
→ optional report
```

한 단계라도 끊기면 사용자가 보는 결과는 완성되지 않습니다.

---

## 15. Limitations

- 연구/교육 목적의 프로젝트이며 상용 의료기기 또는 임상 진단 도구가 아닙니다.
- detection 성능은 학습 데이터와 영상 환경에 영향을 받습니다.
- 연속 탐지 시간과 confidence threshold는 실제 환경에 따라 재조정이 필요합니다.
- 생성형 AI preliminary report는 모델 출력이며 의료 판단 근거로 단독 사용해서는 안 됩니다.
- 실제 병원 시스템 연동, 의료정보 표준, 보안/개인정보 규정 준수는 별도 범위입니다.

---

## 16. What I learned

이 프로젝트를 통해 **좋은 AI 모델과 좋은 AI 제품은 같은 것이 아니다**라는 점을 배웠습니다.

실제 애플리케이션에서는:

```text
Inference accuracy
+ Real-time processing
+ UI responsiveness
+ Event logic
+ Data organization
+ User workflow
```

가 함께 동작해야 합니다.

특히 단일 frame의 prediction을 그대로 기능으로 사용하지 않고, 시간 축의 event로 정의하고 context clip으로 저장하는 과정을 통해 **AI output을 사용자에게 의미 있는 정보로 변환하는 후처리 설계**의 중요성을 경험했습니다.

---

<div align="center">

**From detection results to a usable medical imaging workflow.**

</div>
