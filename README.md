# SceneWeaver

ScriptForge의 마스터 대본과 FlowGenie/TTS의 산출물을 모아, **CapCut 데스크톱이 곧바로 열 수 있는 드래프트 폴더**를 생성하는 Claude Code 스킬 세트. 영상 제작 파이프라인의 마지막 단계.

## 파이프라인 위치

```
StoryLens-27 → ScriptForge → FlowGenie + TTS → [SceneWeaver]
(상담/기획)    (대본 생성)    (이미지+음성)     (CapCut 조립)
```

- **상류**:
  - [ScriptForge](https://github.com/leedonwoo2827-ship-it/scriptforge): `ch{NN}_script.json`
  - FlowGenie: `ch{NN}_{SS}_*.png` (씬별 이미지)
  - TTS: `ch{NN}_{SS}_narration.mp3` (씬별 내레이션)
- **하류**: CapCut 데스크톱 (사람이 드래프트를 열어 최종 편집·렌더링)

## 설치 및 실행

SceneWeaver는 **Claude Code 플러그인**입니다. 두 가지 사용 방식이 있습니다.

### 방식 A — 플러그인으로 설치해서 쓰기 (일반 사용자)

Claude Code 세션에서 다음 두 명령을 실행합니다:

```
/plugin marketplace add leedonwoo2827-ship-it/sceneweaver
/plugin install sceneweaver@sceneweaver
```

설치 후 `/weave`, `/weave-ingest`, `/weave-subtitle`, `/weave-draft` 슬래시 커맨드가 활성화됩니다. 업데이트는 `/plugin update sceneweaver` 로 받을 수 있습니다.

> **중요**: 폴더를 워크스페이스로 여는 것만으로는 슬래시 커맨드가 자동 등록되지 **않습니다**. 루트의 `commands/` 와 `skills/` 는 플러그인 설치 경로로만 활성화됩니다. (프로젝트 로컬 커맨드는 `.claude/commands/` 에 두는 별개 메커니즘)

### 방식 B — 코웍(개발) 모드: 설치 + 폴더 동시 열기

스킬 로직을 구현·수정하면서 테스트할 때는 **둘 다 하는 게 편합니다**:

1. **플러그인 설치** (방식 A와 동일) — 슬래시 커맨드가 활성화됨
2. **동시에 VS Code로 프로젝트 폴더 열기** — `D:\00work\260417-sceneweaver\` 를 워크스페이스로 열어 파일을 수정
3. **수정 후**: GitHub에 push → 사용 측에서 `/plugin update sceneweaver` → 재테스트

동일 Claude Code 세션에서 양쪽 작업이 가능합니다. `CLAUDE.md` 는 워크스페이스 열었을 때 자동 로드되고, 슬래시 커맨드는 설치된 플러그인 쪽에서 동작합니다.

## 사전 준비

1. **CapCut 데스크톱** 설치 (Windows/Mac). 본 프로젝트는 CapCut 4.x 대의 드래프트 포맷을 가정한다.
2. **상류 3개 프로젝트 산출물 경로 확인**:
   - `D:\00work\260416-scriptforge\output\ch{NN}\ch{NN}_script.json`
   - FlowGenie 이미지 출력 폴더 (프로젝트에 따라 가변)
   - TTS 오디오 출력 폴더 (프로젝트에 따라 가변)
3. **Claude Code 코웍 권한**: `.claude/settings.json`이 `workspace/`, `sample/`, `output/` 경로에만 쓰기를 허용한다. 외부 경로는 사용자 확인 후 처리.

## 폴더 구조

```
sceneweaver/
├── .claude-plugin/plugin.json       # 마켓플레이스 등록
├── .claude/settings.json            # 코웍 권한 기본값
├── CLAUDE.md                        # 스킬용 상시 컨텍스트 + 스키마
├── README.md                        # (이 파일)
│
├── commands/                        # /weave 슬래시 커맨드 군
│   ├── weave.md                     #   전체 오케스트레이션
│   ├── weave-ingest.md              #   ① 상류 산출물 수집
│   ├── weave-subtitle.md            #   ② SRT 생성 + 사람 검수 안내
│   └── weave-draft.md               #   ③ CapCut 드래프트 빌드
│
├── knowledge/                       # 스킬이 참조하는 도메인 지식
│   ├── capcut-draft-schema.md       #   draft_info.json 구조 해설
│   ├── subtitle-style-guide.md      #   한국어 SRT 규칙
│   └── transition-library.md        #   scene_meta → CapCut 전환 매핑
│
├── skills/                          # 3개 스킬
│   ├── ingest-assets/SKILL.md       #   상류 파일 수집·검증·리네이밍
│   ├── build-subtitles/SKILL.md     #   narration_text → SRT
│   └── build-capcut-draft/SKILL.md  #   워크스페이스 → CapCut 드래프트
│
├── sample/                          # 한 세트짜리 레퍼런스 입력 (gitignore)
│   └── ch03/
│       ├── ch03_script.json
│       ├── images/
│       └── audio/
│
└── workspace/                       # 실제 작업 공간 (gitignore)
    └── ch{NN}/                      # 챕터별 단일 워크스페이스
        ├── script.json              # ScriptForge에서 복사
        ├── images/                  # FlowGenie 이미지 수집본
        ├── audio/                   # TTS 내레이션 수집본
        ├── subtitles/               # ★ 생성된 SRT (사람이 수정)
        │   ├── ch{NN}_{SS}.srt
        │   └── ch{NN}_full.srt
        └── draft/                   # 최종 CapCut 드래프트 출력
            ├── draft_info.json
            ├── draft_meta_info.json
            └── materials/
```

### 각 폴더의 역할

| 폴더 | 역할 | 수정 주체 |
|---|---|---|
| `commands/` | 슬래시 커맨드 정의. 오케스트레이션 로직 | 개발자 |
| `knowledge/` | 스킬이 참고할 도메인 지식. 스킬 프롬프트 바깥에 두어 재사용 | 개발자 |
| `skills/` | Claude Code 스킬 정의. 각 스킬이 한 가지 일만 함 | 개발자 |
| `sample/` | 레퍼런스 입력 세트. 차기 세션 실험용 | 사용자(수동 배치) |
| `workspace/ch{NN}/script.json` | ScriptForge 원본의 **복사본**. 원본 수정 금지 | 자동 생성 |
| `workspace/ch{NN}/images/`, `audio/` | 상류 산출물 수집본 | 자동 생성 |
| **`workspace/ch{NN}/subtitles/`** | **자동 생성된 SRT. 사람이 자유롭게 수정** | 사용자(편집) |
| `workspace/ch{NN}/draft/` | CapCut이 읽는 최종 드래프트. 재실행 시 덮어씀 | 자동 생성 |

## 작업 흐름

### 권장 순서 (챕터 하나 기준)

```
/weave-ingest 3
  └→ workspace/ch03/ 에 script.json + images/ + audio/ 수집

/weave-subtitle 3
  └→ workspace/ch03/subtitles/ch03_{SS}.srt 생성

[사람 개입]
  └→ subtitles/ 폴더를 열어 줄바꿈·타이밍 미세조정
     (예: "심리학 교수 스키너는 타자기 앞에 앉아" 가 한 줄에 안 맞으면 엔터 삽입)

/weave-draft 3
  └→ workspace/ch03/draft/ 생성 (draft_info.json + materials/)

[수동]
  └→ draft/ 폴더를 CapCut 드래프트 디렉터리로 복사
     Windows: %LOCALAPPDATA%\CapCut\User Data\Projects\com.lveditor.draft\
     → CapCut에서 새 드래프트로 인식됨
```

### 전체 자동화 (단계별 확인)

```
/weave 3
```

세 단계를 순차적으로 돌리되, 각 단계가 끝날 때마다 사용자 확인을 받는다. 자막 편집 단계에서는 SRT 파일 경로를 안내하고 **대기한다**. 사용자가 "계속"이라고 하면 `/weave-draft`로 넘어간다.

## 파일명 규칙

상류(ScriptForge, FlowGenie, TTS)에서 내려온 `ch{NN}_{SS}_*` 패턴을 **그대로 승계한다**. SceneWeaver는 파일명을 바꾸지 않는다.

| 자산 | 파일명 패턴 |
|---|---|
| 스크립트 | `ch{NN}_script.json` |
| 이미지 | `ch{NN}_{SS}_{영문슬러그}.png` |
| 내레이션 | `ch{NN}_{SS}_narration.mp3` |
| 씬 SRT | `ch{NN}_{SS}.srt` |
| 합본 SRT | `ch{NN}_full.srt` |
| CapCut 드래프트 | `ch{NN}_draft/` (폴더) |

## 자막 편집 규칙 (요약)

전체 규칙은 [knowledge/subtitle-style-guide.md](knowledge/subtitle-style-guide.md) 참조. 핵심만:

- **UTF-8 BOM + CRLF** 로 저장 (CapCut 호환)
- 한 줄 **최대 18자**(한글 기준), **최대 2줄**
- 한 큐 **1.5초 이상 7초 이하**
- 조사(은·는·이·가·을·를·에·에서) 앞에서 줄바꿈 금지
- 문장 단위로 큐 분리, 긴 문장은 접속사(그러나, 하지만, 그래서) 경계에서 분할

## 문제 해결

- **"script.json을 찾을 수 없다"** — ScriptForge 프로젝트의 `output/ch{NN}/` 경로 확인. `/weave-ingest` 실행 시 `--script-path` 로 직접 지정 가능.
- **"이미지 씬 번호와 JSON 씬 번호가 안 맞는다"** — FlowGenie가 일부 씬만 생성했을 수 있음. `/weave-ingest`가 누락 리포트를 출력하니, FlowGenie로 돌아가서 재생성.
- **"CapCut에서 드래프트가 안 열린다"** — `materials/` 폴더에 이미지·오디오 사본이 모두 있는지 확인. `draft_info.json`의 절대경로가 상대경로로 바뀌어야 한다. 드래프트 폴더를 통째로 CapCut 디렉터리에 복사했는지 확인.
- **"자막이 CapCut에서 깨진다"** — 인코딩 확인(UTF-8 BOM + CRLF). SRT를 일반 텍스트 에디터에서 열어 한 큐의 타이밍 포맷이 `HH:MM:SS,mmm` 인지 확인.

## 라이센스

MIT
