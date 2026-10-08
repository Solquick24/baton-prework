# Research: 바통 MVP 설계 결정

**Date**: 2026-10-09
**Method**: 기획안·헌장·공식 문서의 읽기 전용 조사; 실제 AWS 호출은 하지 않음.

## Node·언어

- **Decision**: 로컬에 설치된 Node 22.23.2를 기준으로 Node 22.12+ / 22.x와 npm 10.x를 사용한다. 구현 언어는 TypeScript.
- **Rationale**: 현재 환경과 프론트·백엔드를 통일하고 환경 교체 없이 뼈대를 공유한다. Vite 요구 조건과 Lambda 지원 런타임을 확인했다.
- **Alternatives considered**: Node 24는 지원 기간 측면에서 다음 후보. 다른 언어는 공유 계약·개발 도구를 별도로 구성해야 한다.
- **Sources**: [Node 릴리스](https://nodejs.org/en/about/previous-releases), [Lambda 런타임](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html), [Vite 가이드](https://vite.dev/guide/).

## Workspaces·설정 뼈대

- **Decision**: npm workspaces 3개(`@baton/web`, `@baton/api`, `@baton/contracts`)와 단일 lockfile. 모두 private.
- **Rationale**: 로컬 계약 패키지 연결을 관리하고 설정 규모를 줄인다. 빈 src에는 빌드 결과 엔트리를 선언하지 않는다.
- **Alternatives considered**: pnpm·별도 저장소·추가 빌드 도구는 현재 규모에서 도입하지 않는다.
- **Sources**: [npm workspaces](https://docs.npmjs.com/cli/using-npm/workspaces/), [package.json](https://docs.npmjs.com/cli/configuring-npm/package-json/).

## 프론트·환경 변수

- **Decision**: React + Vite. 이번에는 generator를 실행하지 않고 폴더·JSON 설정만 둔다.
- **Rationale**: 현재 요청은 기능 코드 없는 구조 생성이다. 브라우저 환경 변수는 공개 식별자만 사용한다.
- **Alternatives considered**: Next.js는 이 MVP의 서버 렌더링 요구가 없어 선택하지 않는다.
- **Sources**: [Vite](https://vite.dev/guide/), [환경 변수](https://vite.dev/guide/env-and-mode), [공개 assets](https://vite.dev/guide/assets.html).

## 백엔드·비동기

- **Decision**: 기능별 TypeScript Lambda를 SAM·esbuild로 묶는다. 작업 ID·상태·결과 조회를 사용한다.
- **Rationale**: 전사·정리의 대기시간을 HTTP 요청과 분리하고 기획안 AWS 구성을 유지한다.
- **Alternatives considered**: 별도 장기 실행 서버·독립 AI 서비스는 초기 범위에서 제외한다.
- **Source**: [SAM TypeScript 빌드](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-build-typescript.html).

## 권한·공유 기본안

- **Decision**: 3등급 허용표, 현행 권한 조회, 검증된 정리 결과 자동 공유. 동행에게는 허용 항목의 안전한 인용만 제공한다.
- **Rationale**: 기획안 확정 결정과 헌장을 우선한다. 전체 원문 링크가 동행 금지 정보를 노출하지 않도록 한다.
- **Alternatives considered**: 2등급은 확정 방향과 충돌; 별도 확인 후 공유는 기존 확정 결정과 달라 기본안에서 제외.
- **Source**: 기획안 8.1·8.2·13·14.3; 명세의 허용표와 Assumptions.

## AI 리전·안전장치

- **Decision**: 가상 데이터 MVP는 서울 리전 내 가능한 모델 호출과 프롬프트·출력 검증을 기본안으로 계획한다. 교차 리전 Guardrails는 사용하지 않는다.
- **Rationale**: 리전 제약을 유지하고 기획안 9.3의 대안 (b)를 선택한다. 이는 안전성 시험 통과를 의미하지 않는다.
- **Alternatives considered**: Guardrails 교차 리전 처리 허용은 별도 정책 변경·실제 검증 후 검토한다.
- **Dependencies**: 실제 모델 ID·호출 권한·한국어 금지 출력 차단을 당일 확인. ID는 환경설정에 두며 기획안의 특정 모델 접근성을 단정하지 않는다.

## 병원 안내

- **Decision**: 공개 가능한 가상 병원 위치·정적 약도부터 사용한다. 외부 지도 SDK 연결은 구현 시 접근성·계정 조건을 확인한 뒤 선택한다.
- **Rationale**: 핵심 인수인계 흐름과 무관한 지도 제공자 결정이 구조 생성을 막지 않는다.
- **Alternatives considered**: 동선 계산·길찾기는 확정 제외 범위다.

## 조사 결과의 한계

외부 문서 확인은 라이브러리 설치·AWS 호출·배포 성공을 의미하지 않는다.
모델·지도 제공자 설정과 실제 처리 요건은 구현·운영 확인 항목이며, 현재 설계에 미해결 NEEDS CLARIFICATION 표시는 없다.
