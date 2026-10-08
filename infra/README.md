# Infrastructure — 현재 구현 대상에서 제외

최종 기획안과 사용자 선택은 배포 없는 로컬 Fastify·SQLite·시드 로그인, AI만AWS다.
Lambda·API Gateway·DynamoDB·Cognito·Amplify·SAM 자원을 생성하거나 배포하지 않는다.
기존 samconfig.toml.example은 이전 설계의 예시로 보존하며 현재 실행에 사용하지 않는다.

Bedrock·Transcribe와 STT용 기존 임시S3버킷에 대한 프로필·권한은 환경점검 작업에서 확인한다.
앱의 영구 자료는 로컬DB/files다. 실제 자격증명·DB·업로드는 Git에 넣지 않는다.
계획의 기술예외는 이번 가상 로컬데모에만 적용되고 실제서비스 설계 시 재검토한다.
