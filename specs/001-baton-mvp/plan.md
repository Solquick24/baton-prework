# Implementation Plan: 바통 진료 인수인계 MVP

**Branch**: `main` | **Date**: 2026-10-09 | **Spec**: [spec.md](spec.md)

**Input**: `specs/001-baton-mvp/spec.md`

## Summary

보호자가 바뀌어도 같은 환자·진료과의 맥락을 이어받는 모바일 웹 MVP를 구현한다.
React 웹, TypeScript Lambda, 공유 계약을 npm workspaces로 관리한다.
이번에는 명세·설계·빈 디렉터리·설정만 만들며 앱·API·AI·AWS 자원은 구현하지 않는다.
구현은 해커톤 당일 시작하고 1단계 완주 후 필요한 2단계 기능을 추가한다.

## Technical Context

**Language/Version**: TypeScript 예정, Node.js 22.12+ / 22.x, npm 10.x.
로컬 확인 값은 Node 22.23.2, npm 10.9.8. 도구·라이브러리 버전은 실제 구현 시 lockfile에 고정한다.

**Primary Dependencies**: React·Vite, AWS SDK, esbuild, 검증 스키마 도구 예정.
현재 설치한 런타임 의존성은 없고 workspace 관계만 정의했다.

**Storage**: DynamoDB 환자 단위 항목, 비공개 S3 음성·문서, Cognito 계정.

**Testing**: 권한·AI 입력·비교는 백엔드 검증, 핵심 흐름은 브라우저 E2E;
실제 테스트 도구와 코드는 구현 작업에서 추가한다.

**Target Platform**: 모바일 우선 브라우저, 서울 리전 AWS Lambda `nodejs22.x` 계획.

**Project Type**: 웹앱 + 서버리스 API; 별도 AI 서비스 배포 없음.

**Performance Goals**: 30초 브리핑 읽기, 데모 2회 완주.
긴 작업은 즉시 작업 ID 응답 후 2~3초 간격 조회를 기본안으로 두고 종료 상태에서 중단한다.

**Constraints**: 가상 자료만 사용, 서버 권한 검사, 진료과별 입력, 원문 근거,
자동 공유 전 출력 검증, 비공개 메모 제외, 서울 리전 기본, 당일 약 5시간 구현.

**Scale/Scope**: 환자 1명·계정 3개·내과 이전 두 기록·예정 진료·다른 과 비교 기록의 데모.
추가 사용자 역할·번역·푸시·병원 연동은 이번 범위 밖이다.

## Constitution Check

*GATE: 설계 전·후 점검 결과 모두 PASS (설계 준수, 구현 완료를 의미하지 않음).*

| 원칙 | 설계 근거 | 결과 |
|---|---|---|
| I. 인수인계 | 같은 환자·진료과의 질문·브리핑·다음 기록 흐름 | PASS |
| II. 근거 있는 AI | 출력 검증·원문 접근·빈 값·코드 비교·의료 판단 금지 | PASS |
| III. 환자 통제 | 공통 권한 검사·등급별 투영·비공개 메모·위임 검사 | PASS |
| IV. 접근성·녹음 표시 | 전역 글씨 크기·고대비, 직접 녹음 단계의 허용·상태 표시 | PASS |
| V. 검증 가능한 데모 | 가상 자료·저장 대체 결과 구분·작업 상태·범위 구분 | PASS |

동행 범위·자동 공유의 MVP 기본안은 명세의 Assumptions에 기록했다.
헌장의 실정보 처리 TODO는 실제 사용 전 확인 항목으로 유지한다.
Guardrails 교차 리전은 기본안에서 사용하지 않고 서울 리전 호출·프롬프트·출력 검증을
계획한다. 모델 호출 가능 여부와 한국어 금지 출력 차단은 실제 계정에서 검증해야 한다.

## Project Structure

### Documentation (this feature)

```text
specs/001-baton-mvp/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── api.md
└── checklists/
    └── requirements.md
```

`tasks.md`는 아직 없다. 다음 `$speckit-tasks`에서 명세·계획에 연결된 작업을 생성한다.

### Source Code (repository root)

```text
apps/
├── web/
│   ├── public/hospital/
│   └── src/{app,features,components,lib,styles,mocks}/
└── api/
    ├── src/{handlers,modules,auth,ai,adapters,workers,shared}/
    └── tests/
packages/contracts/src/
infra/
fixtures/{seed,audio,documents,expected}/
scripts/
tests/e2e/
```

**Structure Decision**: 화면은 feature별로, 백엔드는 기능 모듈별로 묶는다.
외부 진입점은 handlers, 오래 걸리는 작업은 workers, AWS 연결은 adapters에 둔다.
AI 파이프라인은 백엔드 내부에 두며 사용자 인증·출력 공개는 API 경로를 통해 수행한다.
공유 contracts에는 외부 요청·응답·검증 스키마만 두고 DB 항목·권한 구현을 넣지 않는다.
빈 디렉터리는 `.gitkeep`으로 Git에서 보존한다.

## Implementation Sequence

| 순서 | 구현 작업 | 완료 기준 |
|---|---|---|
| 1 | 실행 도구·프론트 진입점·백엔드 빌드·계약 스키마 | 실제 앱 실행·타입 검사·빌드 명령이 동작 |
| 2 | 일관된 시드·인증·구성원·필드 정책·원문 접근 | 역할별 직접 호출·작업 조회·파일 검증 통과 |
| 3 | 홈·타임라인·전역 글씨 크기·고대비 | 계정별 조회와 모든 구현 화면 접근성 확인 |
| 4 | 질문·통합·브리핑·허용 근거 표시 | B가 내과 맥락을 이어받고 다른 과·금지 정보 제외 |
| 5 | 가상 음성 업로드·전사·메모·작업 상태 | 실패·재시도·저장 전사 대체 사용을 구분 |
| 6 | 진료 정리·출력 검증·코드 비교·자동 공유 | 완료된 검증 결과만 공유, 확인 항목 유지 |
| 7 | E2E·모의 자료 평가·데모 리허설 | 명세 SC 기준 검증, 실제 평가 수치 기록 |
| 8 | 필요한 2단계 보강 | 1단계 완주 후 사진·직접 녹음·비공개 메모 등 추가 |

프론트는 동일 계약의 모의 응답으로 시작하고 백엔드는 같은 시드·허용표를 사용한다.
API 개발자는 데이터·권한·인프라, AI 담당자는 파이프라인·비교·평가,
프론트 담당자는 화면·접근성, 데모 담당자는 시드·시나리오·리허설을 맡는 분담을 제안한다.

## Complexity Tracking

헌장 위반 예외 없음. 별도 AI 서비스·마이크로서비스·공유 UI 패키지·추가 빌드 오케스트레이터는
초기에 추가하지 않는다. 실패 큐 등 비동기 안정화 자원은 필요한 실행 흐름을 먼저 정의한다.
