# SceneWeaver — CapCut 드래프트 자동 생성 스킬

## 프로젝트 개요

ScriptForge의 마스터 대본과 FlowGenie 이미지, TTS 내레이션을 한 챕터 단위로 모아, **CapCut 데스크톱이 곧바로 열 수 있는 드래프트 폴더**를 생성하는 스킬 세트. 드래프트 빌드 전에 **사람이 자막을 손보는 단계**를 명시적으로 끼워넣어, 자동화와 편집 자유도를 동시에 확보한다.

## 파이프라인 위치

```
StoryLens → ScriptForge → FlowGenie + TTS → [SceneWeaver]
(상담/기획)  (대본 생성)   (이미지+음성)      (CapCut 조립)
```

- **상류**:
  - ScriptForge: `ch{NN}_script.json` (마스터 대본)
  - FlowGenie: `ch{NN}_{SS}_*.png` (씬별 이미지)
  - TTS: `ch{NN}_{SS}_narration.mp3` (씬별 내레이션)
- **하류**: CapCut 데스크톱 애플리케이션 (사용자가 드래프트 폴더를 불러와 최종 편집·렌더링)

## 핵심 규칙

1. **입력 JSON 스키마**: ScriptForge 마스터 대본 v1.0을 그대로 받는다. 스키마 변경 시 ScriptForge `CLAUDE.md`를 먼저 업데이트해야 한다.
2. **챕터별 단일 워크스페이스**: `workspace/ch{NN}/` 하나에 해당 챕터의 모든 자산이 모인다. 상류 3개 프로젝트를 오가지 않는다.
3. **자막 편집 개입 포인트**: `/weave-subtitle` 이후 `/weave-draft` 이전에 **사람이 SRT를 수정한다**. 자동으로 넘어가지 않는다.
4. **SRT 인코딩**: UTF-8 BOM + CRLF 줄바꿈 (한국 CapCut 관례).
5. **자막 길이 제한**: 한 줄 최대 18자(한글 기준), 두 줄까지 허용, 한 큐 최대 36자.
6. **파일명 승계**: 상류에서 온 `ch{NN}_{SS}_*` 패턴을 변형하지 않는다. 리네이밍은 하지 않는다.
7. **단계별 확인**: 전체 `/weave` 실행 시 각 단계마다 사용자 확인을 받는다. ScriptForge의 `/forge`와 동일한 철학.
8. **드래프트 폴더는 독립 산출물**: CapCut 드래프트 디렉터리(`AppData\Local\CapCut\User Data\Projects\com.lveditor.draft\`)에 복사하기 전에는 프로젝트 내부에만 존재한다. 복사는 사용자가 수동으로 한다 (또는 `/weave-draft --install` 옵션, 선택적).

## 워크스페이스 구조

```
workspace/ch{NN}/
├── script.json           # ScriptForge에서 복사(읽기 전용 취급)
├── images/               # FlowGenie 이미지 수집본
│   └── ch{NN}_{SS}_*.png
├── audio/                # TTS 내레이션 수집본
│   └── ch{NN}_{SS}_narration.mp3
├── subtitles/            # ★ 생성된 SRT(사람이 수정 가능)
│   ├── ch{NN}_{SS}.srt   # 씬별 SRT — 편집 포인트
│   └── ch{NN}_full.srt   # 합본 SRT
└── draft/                # 최종 CapCut 드래프트 출력
    ├── draft_info.json
    ├── draft_meta_info.json
    └── materials/        # 이미지·오디오 사본(CapCut 요구)
```

## CapCut 드래프트 스키마 요점

전체 구조는 `knowledge/capcut-draft-schema.md`에 서술. 요점만:

- **`draft_info.json`**: 타임라인 본체. `tracks`(비디오/오디오/자막 트랙), `materials`(소재 참조), `canvas_config`(종횡비), `duration`(마이크로초)를 가진다.
- **`draft_meta_info.json`**: 드래프트 썸네일·생성시각 등 메타데이터.
- **`materials/`**: 이미지·오디오·자막 원본 사본. CapCut은 상대경로로 참조하므로 **드래프트 폴더를 통째로 복사**해야 깨지지 않는다.
- **시간 단위**: 마이크로초(μs). `narration_seconds × 1_000_000` 로 변환.
- **자막 트랙**: SRT를 파싱하여 `text` 재료로 변환. CapCut은 내부적으로 자막을 텍스트 트랙으로 다룬다.

## scene_meta → CapCut 매핑

| ScriptForge 필드 | 값 예시 | CapCut 적용 |
|---|---|---|
| `transition_hint` | `fade_in` | 트랙 전환 효과: 페이드 인 |
| `transition_hint` | `crossfade` | 트랙 전환 효과: 디졸브 |
| `mood` | `tension` | 컬러 그레이딩 프리셋: 차가운 톤 |
| `mood` | `warm` | 컬러 그레이딩 프리셋: 따뜻한 톤 |
| `era` | `1950s` | 필터 프리셋: 모노크롬/세피아 |
| `bgm_hint` | `suspenseful_piano` | BGM 트랙 제안 슬롯(사람이 최종 선택) |
| `subtitle` | `1953년 11월, 하버드 대학교` | 씬 시작 시 상단 자막 오버레이 |
| `text_overlay` | (optional) | 추가 텍스트 레이어 |

매핑 확정 테이블은 `knowledge/transition-library.md`.

## 파일명 규칙

- 이미지: `ch{NN}_{SS}_{영문슬러그}.png` (상류 승계)
- 오디오: `ch{NN}_{SS}_narration.mp3` (상류 승계)
- SRT: `ch{NN}_{SS}.srt` (씬별), `ch{NN}_full.srt` (합본)
- 드래프트 폴더명: `ch{NN}_draft` (CapCut에 복사할 때 `{사용자정의}_{날짜}` 로 리네이밍 가능)

## 자막 타이밍 추정

- `narration_seconds`를 씬의 총 자막 큐 시간으로 사용
- 내레이션을 문장 단위로 분절(`.`, `?`, `!`, 한국어는 `。` 추가)
- 각 문장 시간 = (문장 글자수 / 씬 전체 글자수) × `narration_seconds`
- 한 큐 최소 1.5초, 최대 7초 (범위 밖이면 병합/분할 제안)
- 큐 사이 간격 0.1초(겹침 방지)

## 단계 오케스트레이션

1. **`/weave-ingest {N}`** — 상류 3개 소스에서 `workspace/ch{N}/`로 자산을 수집/검증. 누락 파일 보고.
2. **`/weave-subtitle {N}`** — `script.json`의 `narration_text`에서 SRT 생성 → `subtitles/*.srt`. 편집 포인트 안내.
3. **(사람 개입)** — `subtitles/`를 열어 줄바꿈·타이밍 미세조정.
4. **`/weave-draft {N}`** — 편집된 SRT + 이미지 + 오디오 → `draft/` 드래프트 폴더. CapCut에서 열 준비 완료.
5. **`/weave {N}`** — 위 세 단계를 순차 오케스트레이션. 각 단계 사이에 확인.
