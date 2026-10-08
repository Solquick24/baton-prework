# Infrastructure

프론트 Amplify Hosting, 백엔드 AWS SAM을 계획한다. 현재 AWS 자원은 생성하지 않았다.

`samconfig.toml.example`은 구조 예시다. 실제 `template.yaml`은 함수 소스와 배포 권한을
설계한 뒤 추가한다. 존재하지 않는 handler를 선언한 템플릿은 만들지 않는다.

구현할 자원: HTTP API 인증, Cognito 사용자 풀·클라이언트, 기능별 Lambda,
작업 Lambda, DynamoDB, 비공개 S3, 기능별 최소 IAM 권한과 필요한 완료 이벤트 연결.
모델·Guardrails·리전 검증 전에 자동으로 교차 리전 처리를 허용하지 않는다.

`samconfig.toml`과 AWS 자격 증명은 Git에 포함하지 않는다.
