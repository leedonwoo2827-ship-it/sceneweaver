---
description: 수집된 자산과 편집된 SRT를 조합하여 CapCut 데스크톱이 열 수 있는 드래프트 폴더(draft_info.json 포함)를 생성한다.
argument-hint: "[챕터 번호] [--install]"
---

# /weave-draft

`workspace/ch{NN}/` 의 모든 자산을 CapCut 드래프트로 조립한다.

## 사전 조건

- `/weave-ingest` 완료 (이미지·오디오 있음)
- `/weave-subtitle` 완료 + **사람의 SRT 편집 완료**
- `script.json`, `subtitles/*.srt`, `images/`, `audio/` 모두 유효

## 실행 흐름

### Step 1: 자산 검증

- `script.json` 의 씬 수 = 이미지 수 = 오디오 수 인지 확인
- SRT 큐 타이밍이 `narration_seconds` 합계와 일치하는지 확인
- 불일치 시 **실행 중단**하고 원인 리포트

### Step 2: materials/ 구성

`workspace/ch{NN}/draft/materials/` 에 이미지·오디오·SRT 사본 배치. CapCut은 드래프트 폴더 내부 상대경로로 참조하므로 사본이 필수.

```
draft/materials/
├── ch{NN}_{SS}_*.png
├── ch{NN}_{SS}_narration.mp3
└── ch{NN}_{SS}.srt
```

### Step 3: draft_info.json 빌드

`knowledge/capcut-draft-schema.md` 와 `knowledge/transition-library.md` 를 참조하여:

- `canvas_config`: `video_meta.aspect_ratio` → 캔버스 해상도
- `duration`: `scenes[].narration_seconds` 합계 × 1_000_000 (μs)
- `tracks`:
  - **비디오 트랙**: 각 씬 이미지를 순차 배치, `scene_meta.transition_hint` 적용
  - **오디오 트랙**: 내레이션 mp3 순차 배치
  - **자막 트랙**: 편집된 SRT를 텍스트 큐로 변환
  - **(옵션) BGM 트랙**: `bgm_hint` 기반 슬롯만 생성 (사람이 CapCut에서 선택)
- `materials`: `materials/` 내부 파일들을 재료로 등록
- 필터·컬러 그레이딩: `mood`, `era` → 프리셋 적용

### Step 4: draft_meta_info.json 빌드

- 드래프트 ID (UUID)
- 생성 시각
- 썸네일 후보(첫 이미지)
- 드래프트 이름: `ch{NN}_draft_{YYYYMMDD}`

### Step 5: 설치 안내 또는 자동 설치

**기본**: 드래프트 위치만 안내.

```
✓ 드래프트 생성 완료
  위치: D:\00work\260417-sceneweaver\workspace\ch{NN}\draft\

CapCut에서 열기:
  1. %LOCALAPPDATA%\CapCut\User Data\Projects\com.lveditor.draft\ 열기
  2. 위 draft/ 폴더 전체를 복사 (폴더명: ch{NN}_draft_{YYYYMMDD})
  3. CapCut 재시작 → 드래프트 목록에 표시됨
```

**`--install` 옵션**: 자동 복사. 기존 동명 드래프트가 있으면 사용자 확인 후 덮어쓰기.

## 재실행

`draft/` 폴더는 재실행 시 **통째로 덮어쓴다**. CapCut에서 이미 편집 중인 드래프트가 있다면 CapCut 쪽 드래프트를 먼저 닫고 다시 복사해야 충돌 없이 반영된다.

## 문제 해결

- 실행 중단 시 가장 흔한 원인: **SRT 큐 시간 합계가 오디오 총 길이와 다름** — `/weave-subtitle` 재실행 또는 수동 타이밍 조정
- CapCut에서 "미디어를 찾을 수 없음" 표시 — `materials/` 상대경로 확인
- 자막이 안 보임 — SRT 인코딩(UTF-8 BOM) 확인
