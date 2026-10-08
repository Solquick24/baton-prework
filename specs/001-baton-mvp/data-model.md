# Data Model: 로컬 SQLite와 등급별 블록

최신 근거는 최종 기획안8장·부록A다. 기존 DynamoDB 키 초안을 SQLite로 옮기는 설계다.
아직 schema.sql·DB파일·자료는 만들지 않았다.

## Entities and constraints

| 엔터티 | 필드·관계 | 제약 |
|---|---|---|
| users | id,name,passwordHash | 시드 계정만, 비밀번호 원문 저장 금지 |
| patients | id,name,leadUserId,delegated,recordingAllowed | delegated 기본 false, 설정은 환자만 |
| members | patientId,userId,role,scope,active | role=patient/lead/guardian; scope=schedule/companion/full; active=false면 접근 거부 |
| visits | id,patientId,dept,hospital,date,companionId,inputVersion,draftVersion,publishedVersion | 공통 메타는 일정만 이상; 입력 버전 양의 정수 |
| visit_blocks | patientId,visitId,version,kind,payload,validationState,mode | kind=schedule/companion/full; 동일 버전·kind 유일; mode=live/fixture |
| questions | id,patientId,visitId,authorId,text | 질문마다 독립 행; 원 질문 관계 보존 |
| observations | id,patientId,visitId,authorId,text,date | 1단계 시드만; 원본·상세는full |
| alerts | id,patientId,visitId,references,differences,status,resolution | status=open/awaiting_confirmation/resolved; 상세는full |
| jobs | id,patientId,visitId,requestedBy,kind,inputVersion,status,attempt,mode,errorCode | status=queued/running/succeeded/failed; 결과는 블록 참조, 원문 직접 응답 금지 |
| share_logs | id,patientId,targetUserId,actorId,action,oldScope,newScope,time,version | action=start/scope_change/stop; 관리자만 조회 |
| uploads | id,patientId,visitId,storagePath,mediaType | 공개static 제외; 원문읽기는full, 업로드와 읽기권한 분리 |
| documents | id,patientId,visitId,uploadId,extractedFields,editedFields | 2단계; 판독·원본·수정 이력full |
| private_notes | id,patientId,text,includeInDoctorView | 2단계; 환자만, 모든 AI 입력 제외 |
| experiences | hospitalId,dept,order,waitBand,tip | 2단계; 공개 자료에 사용자 식별자·진료 내용 없음 |

UUID/동등한 불투명 ID, 시간은 ISO8601로 통일한다. parentId 소속을 서버에서 검사한다.
SQL 바인딩·foreign key·트랜잭션을 사용한다. 마이그레이션과 시드 리셋은 구현 작업이다.

## Block payloads

- **schedule**: nextSchedule[{id,date,time,hospital,needsCheck}]. AI가 원문에서 추출한 경우 참조는full로 연결한다.
- **companion**: medChanges[{id,drug,from,to,caution,needsCheck}], easySummary[{id,text,needsCheck}],
  mergedQuestions[{id,text,fromQuestionIds,addedByAI,needsCheck}], briefing{changes[{id,text,needsCheck}],questions[]}.
- **full**: diagnosis,labResults,doctorExplanation,medReasons,answers,sourceRefs,basisRefs,
  needsCheckDetails,briefing{changeReasons,watch,prep},transcript,audioRef,documentRefs.
- **private_notes**: 블록 바깥. 어떤 AI 파이프라인에도 포함하지 않는다.

같은 값을 블록에 중복하지 않는다. 여러 화면은 같은 저장 필드를 재사용한다.
briefing.questions는 mergedQuestions의ID를 참조하고 중복 문장을 저장하지 않는다.
low-scope 항목은 id·needsCheck만, sourceRefs·basisRefs·quote·문서주소는full에만 둔다.
null은 미확인값이며 needsCheck=true다. full의sourceRefs는 항목ID로 근거를 연결한다.

## Access repository

- 외부 조회: 인증 → 현행members/active/role/위임 → allowed kind 목록 → 필요한 version의 블록만SELECT.
- 원본 진료의 meta를 먼저 읽되 원문full은 포함하지 않는다. payload 전체 읽기 후 키삭제는 쓰지 않는다.
- 내부 생성 입력 repository는 같은 patientId·dept·목적의 허용 원본만 읽는다. 외부 응답 경로와 구분한다.
- Q/OBS/ALERT/문서/작업/로그의 보조 경로도 scope 규칙을 적용한다. 원본 Q문장의 혼입 검사 실패는full 격리·공유보류다.
- 범위 이름·등급은 일반 가족 응답과 JWT에 넣지 않는다. 서버는 매 요청 DB관계를 읽는다.

## State transitions and transactions

- 작업: queued → running → succeeded/failed. 서버 재시작 시 남은running은failed로 전환해 재시도한다.
- 검토본: generating → ready/blocked/failed. 구조·근거·민감혼입 실패는blocked 또는failed.
- 공유: ready + 현행 권한 + inputVersion일치 → publishedVersion 갱신 + start 로그를 같은 트랜잭션으로 저장.
- 미해결 일반 불일치는needsCheck 유지로 공유 가능; 민감혼입blocked는 공유하기로 우회 불가.
- inputVersion증가 시 새draft 생성; 이전publishedVersion보존. 공유확정 전에는 기존 공유본만 가족에게 전달.
- scope변경과scope_change로그를 함께 저장한다. 같은 요청 재전송은 중복 로그·공유를 만들지 않는다.
- 비공개 메모·위임·stop은2단계. stop은active=false와stop로그를 함께 저장한다.

## Files and AWS staging

영구 DB는 로컬data/baton.sqlite, 영구자료는 로컬uploads/를 계획한다. Git·Vitepublic에서 제외한다.
full 원문 다운로드는 환자소속·현재권한 검사 후 스트림으로 제공하고 storagePath는 응답하지 않는다.
Transcribe용 임시S3 object key는 환자·작업별로 구분한다. SDK로 회수한 결과를 로컬에 저장한다.
시드의 full diagnosis/labResults는 실제 정보 대신 진단명·검사수치 자리표시자로 동행과의 차이를 보여준다.
