---
description: 한 챕터의 자산 수집부터 CapCut 드래프트 빌드까지 전체 플로우를 오케스트레이션. 자막 편집 개입 포인트를 반드시 거친다.
argument-hint: "[챕터 번호]"
---

# /weave

상류 3개 프로젝트의 산출물을 모아 CapCut 드래프트까지 만드는 전체 파이프라인.

## 실행 흐름

### Step 1: 자산 수집 (/weave-ingest)

`ingest-assets` 스킬을 실행:
- ScriptForge의 `ch{NN}_script.json`을 찾아 `workspace/ch{NN}/script.json` 으로 복사
- FlowGenie 이미지를 `workspace/ch{NN}/images/` 로 수집
- TTS 오디오를 `workspace/ch{NN}/audio/` 로 수집
- 누락 자산 리포트 출력

**사용자 확인**:
- "좋아" → Step 2로
- "3번 씬 이미지만 다시 받을게" → FlowGenie 재생성 안내 후 대기
- 누락 자산이 있으면 **반드시** 사용자에게 진행 여부 확인

### Step 2: 자막 생성 (/weave-subtitle)

`build-subtitles` 스킬을 실행:
- `script.json`의 `scenes[].narration_text` 를 문장 단위로 분절
- 씬별 SRT (`subtitles/ch{NN}_{SS}.srt`) + 합본 SRT (`subtitles/ch{NN}_full.srt`) 저장
- 한 줄 초과·타이밍 초과 큐를 **편집 포인트 목록**으로 제시

**사용자 확인 + 편집**:
- SRT 파일 경로 안내: `workspace/ch{NN}/subtitles/`
- 편집 포인트 목록 제시 (예: "ch03_02.srt 3번 큐: 22자 → 줄바꿈 필요")
- 사용자가 에디터에서 SRT를 수정
- "수정 끝났어" / "계속" → Step 3으로

### Step 3: CapCut 드래프트 빌드 (/weave-draft)

`build-capcut-draft` 스킬을 실행:
- 편집된 SRT + 이미지 + 오디오를 조합하여 `workspace/ch{NN}/draft/` 생성
- `draft_info.json`, `draft_meta_info.json`, `materials/` 구성
- scene_meta → CapCut 전환·필터 매핑 적용
- 드래프트 폴더 경로와 CapCut 설치 위치 안내

**완료 보고**:
- 드래프트 위치: `workspace/ch{NN}/draft/`
- CapCut 드래프트 디렉터리: `%LOCALAPPDATA%\CapCut\User Data\Projects\com.lveditor.draft\`
- 복사 명령 예시 출력

## 진행 원칙

1. **각 단계마다 사용자 확인** — 자동으로 다음 단계로 넘기지 않는다
2. **자막 편집 단계는 의무** — Step 2 완료 후 "계속" 확인 없이 Step 3으로 가지 않는다
3. **되돌아가기 가능** — "자막 다시 뽑아줘" → Step 2 재실행 (draft/ 는 나중에 덮어씀)
4. **누락 경고** — 이미지·오디오 씬 수가 script.json 씬 수와 다르면 강한 경고

## 최종 산출물

```
workspace/ch{NN}/
├── script.json
├── images/ch{NN}_*.png
├── audio/ch{NN}_*_narration.mp3
├── subtitles/ch{NN}_*.srt            ← 사람이 수정한 최종본
└── draft/                            ← CapCut에 복사
    ├── draft_info.json
    ├── draft_meta_info.json
    └── materials/
```
