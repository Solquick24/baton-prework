# 바통(Baton) — 진료 동행 노트

가족이 번갈아 동행해도 진료 맥락이 끊기지 않도록 진료 전·중·후 기록을 이어주는 프로젝트다.

현재는 **기획·명세·설계·설정 뼈대** 단계다. 앱·API·AI 기능·테스트·AWS 자원은 아직 구현하지 않았다.

## 문서

- [프로젝트 헌장](.specify/memory/constitution.md)
- [원본 기획안](docs/baton_planning_2026-10-09_04-15-10_KST.md)
- [MVP 명세](specs/001-baton-mvp/spec.md)
- [구현 계획](specs/001-baton-mvp/plan.md)
- [설계 조사](specs/001-baton-mvp/research.md)
- [데이터 모델](specs/001-baton-mvp/data-model.md)
- [API 계약](specs/001-baton-mvp/contracts/api.md)
- [환경 준비·검증 가이드](specs/001-baton-mvp/quickstart.md)
- [명세 품질 검토](specs/001-baton-mvp/checklists/requirements.md)
- [아키텍처](docs/architecture.md) · [결정 기록](docs/decisions.md) · [데모 계획](docs/demo.md)

## 구조

```text
apps/web/             모바일 우선 React/Vite 프론트 자리
apps/api/             TypeScript Lambda·권한·AI·비동기 처리 자리
packages/contracts/   공유 요청·응답·검증 스키마 자리
infra/                AWS SAM·Hosting 설계와 설정 예시
fixtures/             가상 시드·음성·문서·정답 자료 자리
scripts/              환경 점검·시드·평가 스크립트 자리
tests/e2e/            핵심 데모 검증 자리
specs/001-baton-mvp/   명세와 설계 산출물
.specify/             Spec Kit 헌장·설정·템플릿·스크립트
.agents/skills/       팀에서 공유하는 Spec Kit 스킬
```

## 뼈대 준비

Node.js 22.12+ / 22.x, npm 10.x를 사용한다. 현재는 외부 런타임 의존성이 없다.

```bash
npm ci --ignore-scripts --no-audit --no-fund
npm ls --workspaces --depth=0
```

이 명령은 로컬 workspace를 연결할 뿐 앱을 실행하지 않는다.
React·TypeScript·Vite·AWS SDK·테스트 도구와 실제 dev/build/test 명령은 기능 구현 때 추가한다.
설치되지 않은 SAM과 존재하지 않는 Lambda handler를 배포 설정에 선언하지 않았다.

## 다음 구현 단계

`$speckit-tasks`로 명세·계획에 연결된 작업을 만든 뒤 해커톤 당일 구현한다.
새 checkout에서는 다음 값을 지정해 Spec Kit가 main에서도 feature를 찾게 한다.

```bash
export SPECIFY_FEATURE_DIRECTORY=specs/001-baton-mvp
```

`.specify/feature.json`은 checkout별 로컬 포인터여서 공유하지 않는다.
AWS 자격 증명·실제 `.env`·실제 배포 설정은 Git에 포함하지 않는다.
브라우저의 `VITE_*` 변수에는 공개 가능한 식별자만 넣는다.
데모에는 가상 환자·음성·문서만 사용한다.
