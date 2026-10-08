# Quickstart and Validation: 로컬 시연

## 현재 상태

현재는 문서·빈 src·workspace 설정만 있다. 앱·SQLite·로그인·AI·테스트는 미구현이다.
지금 가능한 명령은 workspace 준비·목록 확인뿐이다.

```bash
npm ci --ignore-scripts --no-audit --no-fund
npm ls --workspaces --depth=0
```

## 구현 후 준비와 실행

[tasks.md](tasks.md)의 Setup/Foundation이 실제 실행·시드·빌드·테스트 명령을 추가한다.
아래 명령은 지금 존재하지 않는 미래 명령이며 해당 작업 완료 후에만 실행한다.

1. Node22.x·npm10.x에서 SQLite 바인딩과 Fastify/JWT/SDK 도구 호환성을 확인한다.
2. API 로컬 환경에 JWT_SECRET·SQLITE_PATH·UPLOAD_DIR·AWS_REGION·BEDROCK_MODEL_ID·TRANSCRIBE_STAGING_BUCKET을 설정한다.
3. 웹 환경은 VITE_API_BASE_URL만 사용한다. Cognito 설정·배포 설정은 이번 구성에 필요 없다.
4. 기존 AWS 프로필로 Bedrock·Transcribe·임시 S3 버킷 접근을 확인한다. 실제 계정·토큰·비밀번호를 Git에 넣지 않는다.
5. 원본은 가상 자료만 준비하고 live 호출 불가 시 검증된 fixture provider로 시연한다.

```bash
# 아래는 tasks의 실행 명령 추가·시드 구현 후 사용
npm run seed
npm run dev
npm run typecheck
npm run build
npm run test
npm run test:e2e
```

웹과 API는 localhost에서 실행하고 SQLite·uploads는 공개 static 밖에 둔다.
SAM·Amplify·CloudFormation 배포는 수행하지 않는다. STT의 임시 버킷은 기존 제공 자원만 사용한다.
기존 .env.example은 이전 AWS 구조의 예시라 Setup 작업에서 로컬 구조에 맞게 교체한다.

## 검증 시나리오

### 이어받기 경로

1. B(companion) 로그인 → 내과 브리핑. 변경·질문만 있고 이유·원문·인용·full 키가 없는지 확인한다.
2. 질문 3개 → 통합2개+추가1개. 관계·작성자와 needsCheck를 확인한다.
3. 가상 파일 변환·메모·정리 → ready 검토본. 가족의 timeline에 아직 공개되지 않았는지 확인한다.
4. B가 자기 허용 검토 내용을 확인하고 공유하기 → C/A의 다음 조회에 허용 블록만 표시된다.
5. 환자/A로 전환해 메모·약봉투 불일치와 원문을 확인한다. B에게 19번 상세를 공개하지 않는다.
6. 민감 혼입 샘플은 blocked이고 공유하기로 우회할 수 없는지 확인한다.

### 범위 경로

1. 환자 로그인 → 25-2에서 B를 schedule/companion/full로 변경하고 공유 기록을 확인한다.
2. B의 다음 조회에서 허용 블록과 화면이 바뀌고 금지 블록 키·잠금·다른 가족 범위가 없는지 확인한다.
3. 일반 B의 scope 변경·비구성원 조회·위임이 꺼진 A의 변경을 직접 호출로 거부하는지 확인한다.
4. GET10회·scope3회 동안 AI spy 호출 0회를 확인한다.
5. full에서만 원문·인용·파일을 확인하고 질문·작업·오류·알림 경로도 같은 정책인지 확인한다.

### 공통 품질

가장 큰 글씨·흰 배경 고대비에서 주요 내용·버튼 잘림을 확인한다.
실패·재시작·중복·입력 변경에서 거짓 완료·자동 공유가 없는지 확인한다.
두 경로를2회 연속 완주하고 SC-001~010 결과·실제 평가 분모·검사 전후 누출률을 기록한다.
실제 호출과 저장 대체 결과, 화면만 동의와 기능 완성을 구분한다.
