# Data Model: 바통 MVP

기술 설계 초안이다. 필드별 응답 허용은 [명세](spec.md)의 표를 따른다.
DB 항목과 브라우저 DTO는 분리하며 DB 항목을 그대로 응답하지 않는다.

## Entities

| 엔터티 | 필드·관계 | 검증 규칙 |
|---|---|---|
| User | id, name; 여러 Patient 관계 | 인증 주체와 id 일치 |
| Patient | id, name, leadUserId, delegated, recordingAllowed | 위임·녹음 설정 변경은 환자만 |
| Member | patientId, userId, role, scope | role: patient/lead/guardian; scope: schedule/accompaniment/full |
| Visit | id, patientId, dept, hospital, scheduledAt, companionId, inputVersion, summaryVersion, status | 환자·진료과 확인; 가족 조회는 shared 상태·현행 권한 검사 |
| Question | id, visitId, authorId, text, mergedFrom, evidence | 질문별 개별 항목; AI 추가 질문은 근거 필요 |
| Evidence | id, patientId, visitId, sourceKind, sourceId, location, excerpt, allowedFields | 원본 소속·인용 존재 검사; 동행에 금지 정보 포함 금지 |
| Document | id, patientId, visitId, objectKey, kind, extractedFields, edits | 2단계 판독; 수정 전 값·근거 보존 |
| Alert | id, patientId, visitId, kind, references, differences, status, resolution | 두 원본 비교; 사용자 처리 내역 보존 |
| Job | id, patientId, visitId, requestedBy, kind, inputVersion, status, resultRef, errorCode | 조회 시 구성원·기능 권한 검사; 중복 입력 키로 중복 처리 제한 |
| PrivateNote | id, patientId, text, includeInDoctorView | 환자만; 가족용 AI 입력 제외; 2단계 |
| DisplaySettings | fontSize, highContrast | normal/large/extra-large; 브라우저 로컬 유지 |

## Storage layout

DynamoDB는 기획안의 단일 테이블 초안을 기본으로 한다.

- `USER#{userId}` / `PROFILE`
- `PATIENT#{patientId}` / `META`
- `PATIENT#{patientId}` / `MEMBER#{userId}`
- `PATIENT#{patientId}` / `VISIT#{visitId}`
- `PATIENT#{patientId}` / `Q#{visitId}#{questionId}`
- `PATIENT#{patientId}` / `DOC#{documentId}`
- `PATIENT#{patientId}` / `ALERT#{alertId}`
- `PATIENT#{patientId}` / `JOB#{jobId}`
- `PATIENT#{patientId}` / `PNOTE#{noteId}` (2단계)

사용자별 환자 목록용 인덱스를 구성원 항목에 둔다.
같은 환자·진료과 입력만 선택하고 비공개 메모는 별도 접근 경로로 제한한다.
S3는 `patients/{patientId}/audio/{visitId}/`와 `patients/{patientId}/docs/{documentId}/`로
소속을 구분하고 DB 참조와 함께 검증한 뒤 한정된 URL을 발급한다.
동행용 인용 조회는 전체 S3 파일 URL을 반환하지 않는다.

## State transitions

- Job: `queued → running → succeeded | failed`; 재시도는 동일 대상의 새 attempt로 추적한다.
- Visit: `draft → processing → shared`; 검증 실패는 `failed`이며 공유하지 않는다.
- 수정: inputVersion 증가 → 이전 summaryVersion은 stale → 재정리 완료 시 갱신.
- Alert: `open → awaiting_confirmation | resolved`; 병원 확인 예정은 해결 완료로 취급하지 않는다.
- Scope: 변경은 다음 서버 조회부터 적용; 화면 캐시를 제거하고 원문 접근도 재검사.

## Sharing and private data

검증된 요약 저장과 shared 상태 변경의 일관성을 보장한다.
기존 공유 기록 재정리 실패 시 마지막 검증본을 보존하고 최신 입력과 다른 상태임을 표시한다.
일정 정보는 요약 공유 여부와 별개로 schedule 권한에서 볼 수 있다.
전사·문서·근거의 원본 접근은 full에만 허용하며, accompaniment는 허용 인용만 조회한다.
AI 호출용 맥락에는 비공개 메모를 넣지 않고 일반 로그에는 원문·파일 내용·토큰을 기록하지 않는다.
