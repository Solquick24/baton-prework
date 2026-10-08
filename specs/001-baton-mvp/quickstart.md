# Quickstart and Validation

## Current scaffold

현재는 문서·빈 src·설정만 있다. 앱 실행·빌드·타입 검사·AWS 배포·기능 테스트는 아직 불가능하다.
각 `package.json`에는 존재하지 않는 코드를 실행하는 dev/build/test 명령을 넣지 않았다.
현재 lockfile은 로컬 workspace 관계만 고정한다.

1. Node 22.12+ / 22.x와 npm 10.x를 준비한다(`.nvmrc`는 22).
2. `npm ci --ignore-scripts --no-audit --no-fund`로 로컬 workspace 연결을 준비한다.
3. `npm ls --workspaces --depth=0`에서 web·api·contracts 3개와 로컬 계약 연결을 확인한다.
4. 명세·계획·허용표를 검토한 뒤 `$speckit-tasks`로 구현 작업을 생성한다.

원격에서 checkout을 새로 받으면 기본 main 브랜치에서 downstream Spec Kit 명령이
동작하도록 현재 feature를 지정한다. 로컬 `.specify/feature.json`은 Git에 포함되지 않는다.

```bash
export SPECIFY_FEATURE_DIRECTORY=specs/001-baton-mvp
```

## Implementation prerequisites

- 웹 React/Vite 진입점·TypeScript와 타입 의존성·계약 스키마·실제 빌드·테스트 명령 추가.
- AWS SAM 설치, 계정·서울 리전·Cognito·Bedrock·Transcribe 권한과 실제 모델 호출 확인.
- `infra/template.yaml`과 최소 IAM·비공개 S3·작업 완료 경로 구현 후 배포.
- 공개 웹 식별자와 서버 환경 변수를 각각 `.env.example`에 따라 설정.
- 시드 계정·가상 진료·질문·처방·음성 준비; 실제 비밀번호·토큰은 저장소에 넣지 않음.

## End-to-end acceptance after implementation

1. 보호자 B로 로그인해 내과 브리핑을 연다. 변경·질문·근거를 확인하고 다른 과 기록이 없는지 확인.
2. 질문을 추가·통합해 원 질문을 추적하고 근거 없는 답변이 비어 있는지 확인.
3. 가상 음성·메모로 정리를 시작해 진행·완료 상태를 확인.
4. 메모·약봉투 차이에서 두 근거를 보고 확인 예정으로 처리.
5. 가족의 다음 조회에 검증된 정리가 자동 반영되고 미해결 항목이 유지되는지 확인.
6. 보호자를 schedule로 바꿔 화면·응답·직접 원문 접근에서 금지 정보가 없는지 확인.
7. accompaniment로 바꿔 허용 인용만 보이고 전체 문서·음성 접근이 거부되는지 확인.
8. 일반 보호자의 scope 변경·비구성원 조회·위임 해제 대표의 변경을 거부하는지 확인.
9. 가장 큰 글씨·고대비로 구현 화면을 모두 순회하고 내용·버튼 잘림을 확인.
10. 처리 실패·입력 변경·중복 요청을 재현해 허위 완료·중복 공유가 없는지 확인.

2회 연속 시나리오를 완주하고 [명세](spec.md)의 SC-001~008 결과를 기록한다.
테스트 파일·명령은 구현 단계에서 추가한다. 현재 문서 검토 통과를 기능 테스트 결과로 보고하지 않는다.
