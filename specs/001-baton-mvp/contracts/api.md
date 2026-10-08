# API Contract Draft: 로컬 서버·등급별 블록·검토 후 공유

아직 경로·스키마·handler는 구현하지 않았다. Fastify 로컬 API와 공유Zod계약을 계획한다.

## Authentication and response rules

- POST /auth/login은 시드 계정의 이메일·비밀번호를 확인하고 서명 JWT를 반환한다.
- 서버는 서명·만료·허용알고리즘·issuer/audience를 검증한다. JWT의userId만 신뢰하고 scope/위임은DB에서 읽는다.
- 환자 API앞에는 /patients/{patientId}, 예외는 /me/patients와 정적 병원 안내.
- error.code/message/requestId 형식; 401인증,403권한,404허용범위의없음,400입력,409버전충돌,502외부처리오류.
- 오류·작업응답에 원문·금지블록·내부경로·비밀설정을 넣지 않는다.
- 모든 외부 응답은 공통assembler를 거친다. 알 수 없는블록·필드는기본거부.
- schedule 응답: meta + schedule. companion: meta + schedule + companion. full: 셋모두.
- meta는id/date/dept/hospital/companion만; 민감내용·다른가족scope·거부블록개수는없다.
- sourceRefs/basisRefs/quote/전체파일/진단/수치/이유/답변/설명/ALERT상세는full전용.

## Routes

| Method | Path | 입력·결과 | 행동 권한 |
|---|---|---|---|
| POST | /auth/login | email/password → accessToken | 시드계정 |
| GET | /me/patients | 접근가능환자목록 | 인증 |
| GET | /home?dept= | meta+허용블록·허용개수 | 현행scope |
| GET | /timeline?dept= | publishedVersion의meta+허용블록 | 현행scope |
| GET | /visits/{vid} | meta+허용블록, 작성자/관리자는검토본선택가능 | 현행scope+검토권한 |
| POST | /visits/{vid}/questions | text → id, 공개용text는혼입검증 | companion/full |
| GET | /visits/{vid}/questions | companion.mergedQuestions, full인경우basisRefs | companion/full |
| POST | /visits/{vid}/questions/merge | inputVersion → 결과블록또는jobId | companion/full |
| POST | /visits/{vid}/briefing | inputVersion → 저장블록또는jobId | companion/full |
| GET | /visits/{vid}/briefing | 허용된briefing블록 | companion/full |
| POST | /visits/{vid}/audio | multipart 가상파일 → uploadId | companion/full+recordingAllowed |
| POST | /visits/{vid}/transcribe | uploadId,inputVersion → 202 jobId | companion/full |
| POST | /visits/{vid}/notes | text,inputVersion → 새버전 | companion/full |
| POST | /visits/{vid}/structure | inputVersion → 202 jobId | companion/full |
| GET | /jobs/{jobId} | status/mode/resultVersion/안전한errorCode | 현재기능권한·scope;원문없음 |
| POST | /visits/{vid}/share | draftVersion,inputVersion,idempotencyKey | 작성자또는환자/위임대표,검증ready |
| GET | /sources/{sourceId} | full근거인용/원본다운로드 | full;소속검사 |
| GET | /alerts | 불일치·확인상세 | full |
| POST | /alerts/{aid}/resolve | edit_note/reupload/confirm_hospital | full;원본변경권한추가검사 |
| GET | /members | 가족·scope | 환자/위임대표 |
| PUT | /members/{uid}/scope | schedule/companion/full | 환자/위임대표 |
| GET | /members/{uid}/share-log | 공유시작·범위변경기록 | 환자/위임대표 |
| GET | /hospitals/{hid} | 가상위치·약도·더미경험 | 정적안내 |

POST /audio-url 대신 직접업로드를사용한다. cloud presigned URL은API계약에없다.
전사완료/정리완료의job결과는raw전사가아니라허용된resultVersion참조다.
share는민감혼입blocked·입력버전불일치에서409로거부하고ready검토본만공개한다.
source/download는작성자여도companion이면403이다.

## Generation and reading

생성요청은정해진모든블록을만들고서버에서검증·저장한다. 반환은호출자의허용블록만.
GET·scope변경은저장된블록선택만하고LLM/STT를호출하지않는다.
새field는full에두고화이트리스트스키마에정의전까지낮은범위로반환하지않는다.
low항목은id/needsCheck, full의sourceRefs[id]로원본을연결한다.
근거가없으면값null·needsCheck=true. 프론트는full블록이있을때만원문버튼을그린다.

## Review and published views

일반가족조회는publishedVersion, 검토는작성자/관리자의draftVersion을사용한다.
검토자가companion이어도full은반환하지않는다. 공유전companion문장을확인할수있다.
보안검증은사용자의공유확인과독립적이며저장검증·현행권한검증을버튼으로우회할수없다.
단순미해결불일치는확인표시를유지하며공유할수있고민감혼입은재생성/수정·재검증전공유불가다.

## Optional phase 2

DELETE /members/{uid} 공유중단, POST /docs 직접업로드, /docs/{did}/extract·PATCH수정,
/private-notes 환자전용, /doctor-view 환자전용, PUT /settings 위임·녹음설정,
/hospitals/{hid}/experiences 익명입력, /flows 같은과흐름을추가한다.
별도easy-summary API는없고저장된companion.easySummary를화면24에서재사용한다.
동의·초대·가입·일정등록은화면만이므로실제동작API를추가하지않는다.
