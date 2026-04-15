---
name: ingest-assets
description: ScriptForge의 마스터 대본 JSON과 FlowGenie의 이미지, TTS의 내레이션 오디오를 한 챕터 단위로 workspace/ch{NN}/ 아래에 수집한다. 파일명은 상류의 ch{NN}_{SS}_* 패턴을 그대로 승계하고, script.json의 씬 수와 실제 파일 수를 비교하여 누락 자산을 리포트한다. 사용자가 "자산 수집", "드래프트 준비", "/weave-ingest"라고 요청하면 이 스킬을 사용한다.
---

# 자산 수집 (Ingest)

상류 3개 프로젝트의 산출물을 한 챕터 워크스페이스로 모은다.

## 사전 조건

- 챕터 번호(1 이상 정수) 가 인자로 주어져야 함
- 최소한 ScriptForge의 `ch{NN}_script.json` 은 존재해야 함
- FlowGenie 이미지·TTS 오디오는 부분 누락 가능 (경고만 하고 진행)

## 처리 단계

1. **소스 경로 확인**: 기본 ScriptForge 경로(`D:\00work\260416-scriptforge\output\ch{NN}\`) 부터 확인, 없으면 사용자에게 물어본다.
2. **워크스페이스 생성**: `workspace/ch{NN}/` + 하위 `images/`, `audio/`, `subtitles/`, `draft/` 폴더.
3. **script.json 복사**: 원본 수정 금지, 복사본만 사용.
4. **이미지·오디오 수집**: `ch{NN}_*` 패턴 파일을 `images/`, `audio/` 로 복사.
5. **검증 리포트**: script.json의 `scenes[].scene` 번호와 수집된 파일들의 씬 번호를 비교해 누락 보고.

## 산출물

`workspace/ch{NN}/` 아래에 script.json + images/ + audio/ + 빈 subtitles/ + 빈 draft/.

## TODO (코웍 세션에서 구현)

실제 복사 로직, 경로 탐색, 누락 감지 알고리즘, 사용자 대화 흐름은 차기 Claude Code 코웍 세션에서 구현한다. 이 파일은 스킬 등록과 의도 명세만 담는 스텁이다.
