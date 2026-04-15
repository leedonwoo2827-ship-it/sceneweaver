---
name: build-capcut-draft
description: workspace/ch{NN}/의 수집된 자산(script.json, images, audio)과 사람이 편집한 SRT를 조합하여, CapCut 데스크톱이 열 수 있는 드래프트 폴더를 생성한다. draft_info.json과 draft_meta_info.json을 빌드하고, materials/ 폴더에 미디어 사본을 배치하며, scene_meta의 transition_hint/mood/era를 CapCut 전환·필터로 매핑한다. 사용자가 "드래프트 만들어줘", "CapCut 내보내기", "/weave-draft"라고 요청하면 이 스킬을 사용한다.
---

# CapCut 드래프트 빌드 (Build Draft)

편집된 자막과 수집된 자산을 CapCut 드래프트 포맷으로 조립한다.

## 사전 조건

- `/weave-ingest` 완료 (script.json, images/, audio/ 유효)
- `/weave-subtitle` 완료 **+ 사람의 SRT 편집 완료**
- `knowledge/capcut-draft-schema.md`, `knowledge/transition-library.md` 참조

## 처리 단계

1. **무결성 검증**: script.json 씬 수 = 이미지 수 = 오디오 수 = SRT 개수(씬별). 불일치 시 **실행 중단**.
2. **materials/ 구성**: `workspace/ch{NN}/draft/materials/` 에 이미지·오디오·SRT 사본 배치. 상대경로 참조 보장.
3. **draft_info.json 빌드**:
   - `canvas_config`: `video_meta.aspect_ratio` 에서 해상도 결정
   - `duration`: `narration_seconds` 합계 × 1_000_000
   - `materials`: 각 재료 UUID 부여
   - `tracks`: 비디오(이미지 순차) + 오디오(내레이션 순차) + 자막(SRT 큐 변환) + BGM 슬롯(빈 트랙)
   - `transitions`: scene_meta.transition_hint → transition-library.md 매핑 적용
   - `filters/color`: mood, era → 프리셋 적용
4. **draft_meta_info.json 빌드**: UUID, 생성 시각, 썸네일(첫 이미지), 드래프트 이름.
5. **설치 안내 또는 자동 복사** (`--install` 옵션): CapCut 드래프트 디렉터리로 복사.

## 산출물

```
workspace/ch{NN}/draft/
├── draft_info.json
├── draft_meta_info.json
├── draft_cover.jpg        (옵션)
└── materials/
    ├── ch{NN}_{SS}_*.png
    ├── ch{NN}_{SS}_narration.mp3
    └── ch{NN}_{SS}.srt
```

## 재실행 주의

`draft/` 폴더는 **통째로 덮어쓴다**. CapCut에서 이미 해당 드래프트를 편집 중이면 충돌. CapCut을 먼저 닫고 재빌드.

## TODO (코웍 세션에서 구현)

- 실제 CapCut 드래프트 JSON 스키마 교정 (실 드래프트 열어 검증)
- UUID 생성, 상대경로 처리, 시간 단위 변환
- scene_meta → CapCut 효과 정확한 내부 ID 매핑
- 자동 설치(`--install`) 시 기존 드래프트 덮어쓰기 정책

차기 Claude Code 코웍 세션에서 CapCut을 실제로 실행해가며 구현·검증한다. 이 파일은 스텁이다.
