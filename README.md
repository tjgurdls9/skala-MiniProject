# Jobpill — SKALA MiniProject

Vue 3 + Vite로 만든 기업 탐색·프로필 입력·진단 리포트 프론트엔드입니다. 루트 앱과 팀의 전체 서비스 저장소를 구분합니다.

```sh
npm ci
npm run dev
# 배포용 빌드
npm run build
```

| 경로 | 내용 |
| --- | --- |
| `src/`, `public/` | 현재 Jobpill 프론트엔드 |
| `career-compass/` | 팀의 별도 backend/frontend 저장소를 가리키는 Git submodule |
| `career-compass-backend-tasks/` | 백엔드 구현 작업 명세 |
| `BRAND_DIRECTION.md`, `HANDOFF.md` | 디자인 방향과 개발 인계 |
| `DUMMY_*` | 시연용 기업 데이터 |

팀 서비스가 필요하면 `git submodule update --init`을 실행하고 `career-compass/README.md`를 따릅니다. 현재 submodule 커밋은 `feat/frontend-polish`의 저장된 작업 상태입니다. 의존성·빌드 결과·복원 백업·분석 캐시는 Git에서 제외하고 로컬에 보존합니다.
