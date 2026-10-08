# API workspace

TypeScript Lambda 백엔드의 자리다. 함수·비즈니스 로직·AWS SDK 의존성은 아직 없다.

- `handlers`: 요청 검증 → 인증·권한 → 기능 모듈 → 응답의 얇은 진입점.
- `modules`: 구성원·진료·질문·브리핑·정리·불일치·작업의 기능 로직.
- `auth`: 환자별 구성원·작업 권한·필드 허용·원문 접근 공통 검사.
- `ai/prompts`: 진단·처방·수치 해석 금지와 근거 요구를 명시한 프롬프트.
- `ai/pipelines`: 환자·진료과·공유 목적별 입력 구성과 출력 검증.
- `ai/safety`: 근거 누락·금지 출력·권한 밖 정보 확인.
- `adapters`: DynamoDB·S3·Cognito·Bedrock·Transcribe 연결.
- `workers`: 긴 작업의 실행·상태·결과·실패·재시도.
- `shared`: 설정·오류·로그; 원문·음성을 로그에 그대로 남기지 않는다.
- `tests`: 권한 우회·원문 접근·AI 입력 제한·불일치 비교의 실제 테스트 자리.

API별 인증·권한 검사를 수행하고 S3 URL과 작업 결과에도 동일 정책을 적용한다.
구현 착수 시 TypeScript·Node 타입·필요 AWS SDK·esbuild와 실제 빌드 명령을 추가한다.
명세: [spec.md](../../specs/001-baton-mvp/spec.md).
