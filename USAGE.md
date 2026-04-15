# SceneWeaver 사용설명서

커맨드 중심 실행 가이드. 프롬프트 예문과 함께.

> 이 문서는 플러그인이 **설치되어 있다는 전제**로 작성됨.
> 설치: `/plugin marketplace add leedonwoo2827-ship-it/sceneweaver` → `/plugin install sceneweaver@sceneweaver`
> 설치 후 `/weave`, `/weave-ingest`, `/weave-subtitle`, `/weave-draft` 활성화.

---

## 빠른 시작 (한 챕터 조립)

```
/weave-ingest 3
     ↓
/weave-subtitle 3
     ↓
(사람 편집: workspace/ch03/subtitles/*.srt)
     ↓
/weave-draft 3
```

또는 한 번에:

```
/weave 3
```

(각 단계에서 확인 요청 — 자동으로 넘어가지 않음)

---

## 커맨드별 상세

### 1. `/weave-ingest {챕터번호}` — 자산 수집

**하는 일**: ScriptForge의 `script.json`, FlowGenie의 이미지, TTS의 오디오를 `workspace/ch{NN}/` 로 모은다. 누락 자산을 보고한다.

**기본 실행**:
```
/weave-ingest 3
```

**경로를 직접 지정하고 싶을 때** (프롬프트 예문):

```
/weave-ingest 3
ScriptForge 경로는 D:\00work\260416-scriptforge\output\ch03\ch03_script.json 으로,
이미지는 D:\flowgenie\output\ch03\ 에서,
오디오는 D:\voice-harness\output\ch03\ 에서 가져와줘.
```

**누락 자산이 있을 때 (예상 응답)**:
```
📥 수집 리포트: ch03
   스크립트: ✓
   이미지:   5/7 씬 (누락: S3, S6)
   오디오:   7/7 씬 ✓

계속할까요? 아니면 FlowGenie에서 S3, S6 재생성 후 다시 실행할까요?
```

이 때 선택:
- "계속" → 일단 있는 자산만으로 다음 단계 진행 (권장 X)
- "멈춰. FlowGenie 다시 돌리고 올게" → 작업 중단

**후속 프롬프트 예문**:
- "누락된 씬 번호만 한번 더 확인해줘"
- "script.json의 씬 수하고 이미지/오디오 개수를 나란히 표로 보여줘"

---

### 2. `/weave-subtitle {챕터번호}` — 자막 생성

**하는 일**: `script.json`의 `narration_text` 를 **씬별 SRT 파일**로 변환한다. `workspace/ch{NN}/subtitles/` 에 저장.

**⚠ 자막 폴더는 이 단계에서 생성됨** — `/weave-ingest` 시점에 빈 폴더만 만들어두고, 실제 SRT 파일은 여기서 채워진다.

**기본 실행**:
```
/weave-subtitle 3
```

**생성 결과**:
```
workspace/ch03/subtitles/
├── ch03_01.srt       ← 씬 1 자막 (로컬 타임: 0초부터)
├── ch03_02.srt       ← 씬 2 자막
├── ...
├── ch03_07.srt
└── ch03_full.srt     ← 합본 (전역 타임라인)
```

**예상 응답**:
```
📝 자막 생성 완료: ch03
   7개 씬, 42개 큐, 예상 길이 180초

⚠ 편집 권장 큐 (3건)
  ch03_02.srt #3: 22자 → 18자 초과 (줄바꿈 필요)
    "심리학 교수 스키너는 타자기 앞에 앉아"
  ch03_05.srt #1: 0.8초 → 너무 짧음 (다음 큐와 병합 권장)
  ch03_07.srt #4: 8.2초 → 너무 김 (분할 권장)

편집 위치: workspace/ch03/subtitles/
완료 후 /weave-draft 3
```

**자주 쓰는 후속 프롬프트 예문**:

```
ch03_02.srt 3번 큐 "심리학 교수 스키너는 타자기 앞에 앉아"
"앉아" 앞에서 줄바꿈 해줘.
```

```
편집 권장 큐 3개를 다 자동으로 고쳐줘.
18자 초과는 어절 경계에서 줄바꿈,
너무 짧은 건 앞 큐와 병합,
너무 긴 건 문장 경계에서 분할.
```

```
ch03_05.srt 첫 큐와 두번째 큐를 하나로 합쳐줘.
타이밍은 첫 큐의 시작 시각부터 두번째 큐의 끝 시각까지.
```

```
합본 SRT(ch03_full.srt)를 다시 만들어줘.
씬별 파일에서 수정한 내용 반영해서.
```

---

### (자동화 아님 — 사람 개입)

에디터로 `subtitles/` 열어서 눈으로 확인 + 수정. VS Code라도 `.srt` 는 기본 편집 가능.

체크 포인트:
- 한 줄 18자 초과? → 어절 경계에서 엔터
- 조사(은/는/이/가/을/를) 앞에 엔터가 있으면? → 앞으로 옮기기
- 너무 짧은 큐(1.5초 미만)? → 앞/뒤와 병합
- 너무 긴 큐(7초 초과)? → 문장 경계에서 분할

수정 끝나면 **Claude에게 "계속" 또는 `/weave-draft 3`** 라고 말함.

---

### 3. `/weave-draft {챕터번호}` — CapCut 드래프트 빌드

**하는 일**: 편집된 SRT + 이미지 + 오디오 → CapCut이 열 수 있는 드래프트 폴더.

**기본 실행**:
```
/weave-draft 3
```

**예상 응답**:
```
✓ 드래프트 생성 완료
  위치: D:\00work\260417-sceneweaver\workspace\ch03\draft\

CapCut에서 열기:
  1. %LOCALAPPDATA%\CapCut\User Data\Projects\com.lveditor.draft\ 열기
  2. 위 draft/ 폴더 전체를 복사 (폴더명 자유로이 변경 가능)
  3. CapCut 재시작 → 드래프트 목록에 표시됨
```

**자동 설치 옵션**:
```
/weave-draft 3 --install
```
→ CapCut 드래프트 디렉터리로 자동 복사. 기존 동명 드래프트 있으면 덮어쓰기 확인.

**자주 쓰는 후속 프롬프트 예문**:

```
드래프트 다시 빌드하되, 1번 씬 전환효과를 페이드 인 대신
디졸브로 바꿔줘.
```

```
draft_info.json 을 열어서 자막 트랙 위치를 화면 하단에서
중앙 하단으로 (y 오프셋 -300) 바꿔줘.
```

```
씬 3번 이미지를 image2.png 대신 image2_v2.png 로 바꾸고
드래프트를 다시 빌드해줘.
```

---

### 4. `/weave {챕터번호}` — 전체 오케스트레이션

**하는 일**: 위 세 단계를 순차 실행하되 **각 단계마다 확인**한다. 자막 편집 단계에서는 **대기**한다.

**기본 실행**:
```
/weave 3
```

**실행 흐름**:
```
Step 1: /weave-ingest 3
    → 수집 리포트 제시 → 사용자 확인

Step 2: /weave-subtitle 3
    → 자막 생성 + 편집 권장 목록 제시
    → "subtitles/ 에서 수정 후 '계속' 해주세요" 메시지

    [사람이 SRT 편집]

    사용자: "계속" 또는 "수정 끝났어"

Step 3: /weave-draft 3
    → 드래프트 생성 + 설치 안내
```

**중간에 되돌아가기**:

```
"자막 다시 뽑아줘" → Step 2 재실행
"ch03_02 이미지만 다시 받아올게" → Step 1로 돌아가 S2만 재수집
```

---

## 유용한 후속 프롬프트 모음

### 자막 일괄 작업
```
workspace/ch03/subtitles/ 안의 모든 SRT에서
마침표 뒤에 공백이 없는 곳 찾아서 고쳐줘.
```

```
전체 자막을 검토하고 맞춤법 오류만 리포트해줘. 수정은 아직 하지 말고.
```

```
ch03_full.srt 에서 한 큐가 5초를 넘는 것들만 리스트로 보여줘.
```

### 합본(merge) 관련
```
씬별 SRT 7개를 합쳐서 ch03_full.srt 로 만들어줘.
각 씬의 시작 시각은 이전 씬들의 narration_seconds 누적값으로.
큐 번호는 전역으로 1부터 다시 매기고.
```

```
ch03_full.srt 와 개별 씬 SRT 들의 타이밍이 일치하는지 검증해줘.
어긋나는 큐가 있으면 리포트.
```

### 드래프트 검증
```
draft/draft_info.json 을 열어서
1) 총 duration 이 script.json의 total_duration_seconds 와 일치하는지
2) materials/ 안의 파일들이 tracks 에서 모두 참조되고 있는지
3) 참조되지 않는 파일이나 존재하지 않는 파일 참조가 있는지
검증 리포트 만들어줘.
```

### 챕터 전체 일괄 처리 (여러 챕터)
```
ch01 부터 ch07 까지 /weave 를 순차 실행해줘.
각 챕터의 자막 편집 단계는 일단 스킵(자동 추정치 그대로 사용),
최종 드래프트까지 다 만들고 리포트.
```

---

## 문제 해결 프롬프트

### 자산 누락
```
workspace/ch03/images/ 에 있는 파일명과
script.json 의 scenes[].image_filename 을 비교해서
어긋나는 게 있는지 봐줘.
```

### SRT 인코딩 깨짐
```
ch03_full.srt 를 UTF-8 BOM + CRLF 로 다시 저장해줘.
```

### CapCut에서 드래프트 안 열릴 때
```
draft/draft_info.json 의 version 필드를 확인하고,
내 CapCut 버전(메뉴 > 정보에서 확인 후 알려줌)과 호환되는지 판단해줘.
```

---

## 체크리스트 (챕터 시작 전)

- [ ] `D:\00work\260416-scriptforge\output\ch{NN}\ch{NN}_script.json` 있음
- [ ] FlowGenie 이미지 `ch{NN}_{SS}_*.png` 씬 수만큼 있음
- [ ] TTS 내레이션 `ch{NN}_{SS}_narration.mp3` 씬 수만큼 있음
- [ ] CapCut 데스크톱 설치됨
- [ ] Claude Code에 sceneweaver 플러그인 설치됨 (`/plugin list` 로 확인)
