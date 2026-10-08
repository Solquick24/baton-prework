# API Contract Draft: 바통 MVP

설계 문서이며 아직 경로·DTO·handler를 구현하지 않았다.
공유 런타임 스키마는 `packages/contracts/src/`에, DB 항목은 API 내부에 둔다.

## Common rules

- 모든 요청은 인증된 호출자를 확인하고 환자·구성원·작업·필드 권한을 검사한다.
- 환자 경로 앞에는 `/patients/{patientId}`를 붙인다. `/me/patients`와 공개 병원 안내는 별도다.
- 요청의 patientId와 저장된 visit/document/job/source 소속이 일치해야 한다.
- 오류 형식: `error.code`, `error.message`, `error.requestId`. 원문·비밀 설정은 메시지에 넣지 않는다.
- 인증 실패는 401, 권한 거부는 403, 허용 범위에서 없는 자원은 404,
  잘못된 입력은 400, 입력 버전 충돌은 409, 외부 처리 오류는 502 또는 실패 작업 상태다.
- 알 수 없는 필드·scope·role은 거부한다. 일반 보호자에게 다른 구성원의 등급을 반환하지 않는다.
- ID 기반 자원 접근은 자원 존재나 내용이 권한 밖으로 새지 않도록 검사 순서를 통일한다.

## Endpoints

| Method | Path | 목적·응답 | 주요 권한 |
|---|---|---|---|
| GET | /me/patients | 접근 가능한 환자 목록 | 인증 |
| GET | /home?dept= | 일정·질문·확인 항목·최근 기록 | scope별 필드 제한 |
| GET | /timeline?dept= | 일정·공유 기록 | scope별 필드 제한 |
| POST | /visits/{visitId}/questions | text 입력, 원 질문 저장 | 동행·전체 |
| GET | /visits/{visitId}/questions | 원 질문·통합 결과 | 동행·전체 |
| POST | /visits/{visitId}/questions/merge | inputVersion, 통합 결과 또는 작업 ID | 동행·전체 |
| POST | /visits/{visitId}/briefing | inputVersion, 브리핑 또는 작업 ID | 동행·전체 |
| GET | /visits/{visitId}/briefing | 허용 필드·근거 인용 | 동행·전체 |
| POST | /visits/{visitId}/audio-url | 형식·크기 검사 후 업로드 URL | 동행·전체; recordingAllowed |
| POST | /visits/{visitId}/transcribe | objectRef, inputVersion → 작업 ID | 동행·전체 |
| POST | /visits/{visitId}/notes | text, inputVersion → 저장·새 버전 | 동행·전체 |
| POST | /visits/{visitId}/structure | inputVersion → 작업 ID | 동행·전체 |
| GET | /jobs/{jobId} | status·허용된 결과 참조·실패 코드 | 해당 기능 권한·현행 scope |
| GET | /visits/{visitId} | 일정 또는 허용된 공유 기록 | scope별 필드 제한 |
| GET | /sources/{sourceId} | 허용 인용 또는 원본 접근 URL | 동행: 인용만; 전체: 원본 |
| GET | /alerts | 허용된 확인 항목 | 동행·전체 |
| POST | /alerts/{alertId}/resolve | 수정·재등록·병원 확인 예정 처리 | 동행·전체; 원본 변경 권한 별도 검사 |
| GET | /members | 가족별 scope | 환자·위임된 대표 |
| PUT | /members/{userId}/scope | schedule/accompaniment/full | 환자·위임된 대표 |

별도 `/share` 경로는 기본안에서 두지 않는다. 검증된 정리 저장이 자동 공유 시점이다.
조회 권한과 변경 권한은 독립적으로 검사하며 full인 일반 보호자도 scope를 변경할 수 없다.

## Response projections

- Schedule: visitId, scheduledAt, nextAppointments, hospitalLocation.
- Accompaniment: 일정 + dept, companion, permittedMedicationChanges, precautions,
  easySummary, permittedChanges, questions, briefing, safeEvidenceExcerpts.
- Full: 일정 + 전체 진료 정리, 진단명·수치 등 원문 기록, 허용 원본 참조.
- PrivateNote는 위 DTO 어디에도 자동 포함하지 않는다. 전용 환자 경로에서만 반환한다.
- schedule 홈에는 질문·확인 항목 개수·최근 진료 요약·브리핑 링크를 반환하지 않는다.
- 동행용 변환 결과에도 원문·의료 판단·금지 정보가 포함됐는지 검사한다.

## Evidence-bearing fields

AI 출력 항목은 `value`, `evidenceRefs`, `needsConfirmation`을 공통으로 가진다.
근거가 없으면 value는 null, evidenceRefs는 빈 목록, needsConfirmation은 true다.
질문 통합은 originalQuestionIds·aiAdded를 보존한다.
인용에 허용 밖 정보가 섞이면 인용을 반환하지 않고 확인 상태를 표시한다.
입력·출력의 실제 JSON 스키마는 구현 작업에서 위 규칙에 따라 작성한다.

## Asynchronous work

작업 시작은 202와 jobId·status를 반환한다.
조회는 queued/running/succeeded/failed, inputVersion, 허용 결과 참조 또는 오류 코드를 반환한다.
입력 버전과 작업 종류를 중복 판별에 사용하고 재시도를 별도 attempt로 추적한다.
프론트는 종료 상태에서 폴링을 중단하고 실패 시 재시도 안내를 제공한다.
음성 전사는 시작 Lambda에서 끝까지 기다리지 않고 완료 이벤트·상태 확인 경로로 마무리한다.

## Phase 2 additions

문서 업로드·판독·수정, 비공개 메모, 의사용 화면, 환자 위임·녹음 설정,
방문 경험 입력은 구현 범위 확정 후 별도 계약을 추가한다.
