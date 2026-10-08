# API workspace

로컬 TypeScript/Fastify API의 자리다. 소스·의존성·DB는 아직 없고 설정 뼈대만 있다.

- handlers: HTTP routes; 요청 검증→인증·관계·행동 권한→기능 모듈→허용 블록 응답.
- modules: members·visits·questions·briefing·summaries·alerts·jobs.
- auth: JWT·매 요청 현재 관계·scope→kind 화이트리스트·위임·원문 검사.
- adapters: SQLite·로컬 files·Bedrock·Transcribe·fixture 연결.
- ai: 생성 입력의 환자·진료과·목적 제한, 세블록 출력·근거·혼입 검증.
- workers: 로컬 비동기 실행·SQLite 작업 상태·재시도·재시작 처리.
- shared: 설정·오류·로그; 원문·토큰을 로그에 남기지 않는다.
- tests: 실제 계약·권한·블록 미조회·AI 입력·공유 보류 검증의 자리.

앱 저장·로그인은로컬, AI만AWS. 원문은full 전용이고 로컬파일은 공개static에 두지 않는다.
POST share 이전 결과를 자동 공유하지 않는다. GET와scope변경은AI를 부르지 않는다.
기존.env.example의Cognito/DynamoDB/S3영구저장 값은 이전 예시이며 Setup에서 로컬값으로 교체한다.
구현목록: [tasks.md](../../specs/001-baton-mvp/tasks.md).
