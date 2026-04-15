---
description: 상류 3개 프로젝트(ScriptForge/FlowGenie/TTS)에서 한 챕터의 산출물을 workspace/ch{NN}/로 수집한다. 누락 자산을 리포트한다.
argument-hint: "[챕터 번호] [--script-path <path>] [--images-dir <path>] [--audio-dir <path>]"
---

# /weave-ingest

챕터 하나의 자산을 `workspace/ch{NN}/` 로 모은다.

## 실행 흐름

### Step 1: 소스 경로 결정

- 기본 ScriptForge 출력: `D:\00work\260416-scriptforge\output\ch{NN}\ch{NN}_script.json`
- FlowGenie 이미지 경로: 사용자에게 물어보거나 `--images-dir` 로 지정
- TTS 오디오 경로: 사용자에게 물어보거나 `--audio-dir` 로 지정
- 경로가 없으면 사용자에게 확인

### Step 2: 워크스페이스 준비

- `workspace/ch{NN}/` 하위 폴더 생성: `images/`, `audio/`, `subtitles/`, `draft/`
- 이미 존재하면 사용자 확인: 덮어쓰기 / 건너뛰기 / 취소

### Step 3: 파일 수집

- `script.json` 복사 (원본 수정 금지)
- `ch{NN}_*.png` 전체를 `images/` 로 복사
- `ch{NN}_*_narration.mp3` 전체를 `audio/` 로 복사
- 파일명은 변경하지 않는다

### Step 4: 검증 & 리포트

`script.json` 의 `scenes[]` 와 수집된 파일 수를 비교:

```
📥 수집 리포트: ch{NN}
   스크립트: ✓
   이미지:   N/M 씬 (누락: S2, S5)
   오디오:   N/M 씬 (누락: S5)

누락이 있으면:
  - FlowGenie로 돌아가 누락 씬 이미지 재생성
  - TTS로 돌아가 누락 씬 오디오 재생성
  - /weave-ingest 재실행
```

## 사용자 수정 대응

- "S2 이미지 파일명이 달라" → 실제 파일명을 물어보고 리네이밍 확인
- "다른 경로에서 가져와" → `--images-dir` 재지정
- "script.json이 구버전이야" → ScriptForge로 돌아가 재생성 안내

## 산출물

```
workspace/ch{NN}/
├── script.json
├── images/ch{NN}_{SS}_*.png    (M개 또는 일부)
├── audio/ch{NN}_{SS}_narration.mp3
├── subtitles/                  (빈 폴더, 다음 단계에서 채움)
└── draft/                      (빈 폴더, 다음 단계에서 채움)
```
