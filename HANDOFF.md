# 인수인계서 — 2026.09.03 세션 종료 시점

새 세션에서 이 프로젝트를 이어받을 때 가장 먼저 읽을 문서입니다. 특히 **"⚠️ 가장 먼저 확인할 것"**은 반드시 먼저 처리하세요 — 안 보고 작업을 시작하면 이미 끝난 일을 처음부터 다시 하게 됩니다.

---

## ⚠️ 가장 먼저 확인할 것 — 되돌아간(reverted) 작업이 백업에 남아 있음

이 작업 디렉토리(`skala-MiniProject/`)에는 코드베이스 두 개가 섞여 있습니다.

1. **`src/`** (지금 라이브 상태) — 어느 시점엔가 **더 오래된 버전으로 되돌아가 있습니다.**
2. **`.restore-backup-20260903-161621/src/`** — 되돌아가기 직전, **더 완성된 버전**이 그대로 남아 있습니다.

`diff -rq src .restore-backup-20260903-161621/src` 결과, 다음 5개 파일이 다릅니다:

| 파일 | 라이브(`src/`) 상태 | 백업 상태 |
|---|---|---|
| `src/components/CompanyLogo.vue` | **없음** | 있음 — 기업 도메인 파비콘을 가져오고 실패하면 브랜드색 이니셜 타일로 대체하는 컴포넌트 |
| `src/views/Companies.vue` | 기업 **6개** | 기업 **21개** (`CompanyLogo` 사용) |
| `src/views/Report.vue` | 추천 실행안 2개 + 이력서 팁 문자열 3줄만 | **추천 프로젝트(P1~P3) 섹션 전체**(할 일·산출물·기간·예상 점수·이력서 문장까지 포함), 이력서 팁이 before/after 4쌍 구조로 확장 |
| `src/style.css` | 인쇄(`@media print`) 규칙에 페이지 잘림 버그 있음 | **PDF 잘림 수정 완료** 버전 (여백 압축, `break-inside` 범위 축소, `--gutter` 인쇄 전용값 등) |
| `src/components/Histogram.vue` | 화면용 스타일만 | 인쇄용 축소 스타일(`@media print`) 추가됨 |
| `src/views/Home.vue` | 카드 그리드 + 형광펜 강조(`.hl`) 남아있음 | 표(`<table>`)로 교체 + 형광펜 강조 제거 (클로드 특유 디자인 tell 제거 작업 반영) |

**결론**: 사용자가 요청했던 "PDF 잘림 수정 / 추천 프로젝트 더미데이터 채우기 / 기업관 21곳 / 기업 로고 가져오기 / 클로드 특유 디자인 제거" 5가지 작업은 **이미 다 끝나 있고 백업 폴더에 존재**합니다. 다만 그 직후 무언가(`/rewind` 등 체크포인트 되돌리기로 추정)로 `src/`가 더 이전 상태로 리셋됐고, 그 도구가 되돌리기 직전 상태를 `.restore-backup-20260903-161621/`에 자동 저장해둔 것으로 보입니다.

**권장 조치** (사용자 확인 후 진행):
```bash
# 백업이 진짜 최신/완성본인지 먼저 dist 빌드로 검증한 뒤
cp -r .restore-backup-20260903-161621/src/* src/
npm run build   # 에러 없는지 확인
```
복구했다면 `DUMMY_DATA.md`도 반드시 다시 뽑아야 합니다 — 그 문서는 **되돌아간 이후의 라이브 상태**를 기준으로 작성되어서 기업 6개짜리 구버전 데이터만 담고 있습니다 (21개 기업, 추천 프로젝트 P1~P3 데이터가 빠져 있음).

> 이 문제는 사용자에게 한 번 설명했고("그게 무슨소리지?" 질문에 답변함), 사용자는 복구 작업 자체는 **"일단 대기"**라고 했습니다. 새 세션에서 먼저 사용자에게 복구 여부를 확인하세요.

---

## 1. 이 디렉토리에 있는 것들

```
skala-MiniProject/
├── src/                                  ← Vue 프론트 프로토타입 (라이브, 아래 2절)
├── index.html, vite.config.js, package.json
├── .restore-backup-20260903-161621/      ← ⚠️ 위 항목 참고. 되돌리기 직전 백업
├── DUMMY_DATA.md                         ← 라이브 src/ 기준 더미 데이터 전체 정리 (아래 4절)
├── HANDOFF.md                            ← 이 문서
├── API_명세서_v2.pdf                      ← 팀 API 명세서 원본 (8페이지)
├── career-compass-backend-tasks/         ← 백엔드 담당자 A/B/C/D 작업 분담 문서 (아래 3절)
├── career-compass/                       ← ⭐ 실제 팀 풀스택 저장소 (자체 git, 아래 3절)
├── node_modules/, dist/, package-lock.json
└── .git/                                 ← 이 최상위 디렉토리 자체의 git (커밋 0개, 아직 아무것도 안 올라감)
```

**Git 구성**: 루트 `.gitignore`가 의존성·빌드 결과·복원 백업·분석 캐시를 제외합니다. `career-compass/`는 `.gitmodules`에 등록된 외부 저장소이며 `git submodule update --init`으로 가져옵니다.

---

## 2. `src/` — Vue 프론트 프로토타입 (Jobpill / 잡필)

### 개요
- Vue 3 + Vite, 라이브러리 최소(라우팅도 직접 짠 해시 라우터), 컴포넌트 7개 + 뷰 8개.
- 실행: `npm run dev` (포트는 vite 기본, 세션 중엔 5199 등으로 띄웠음), 빌드: `npm run build`.
- **백엔드 연동 없음.** `src/api.js`에 REST 클라이언트 함수는 정의돼 있지만 어떤 화면도 실제로 호출하지 않습니다. 모든 화면 데이터는 각 `.vue` 파일 안에 하드코딩된 배열/객체입니다.
- 디자인 방향: Figma 원본 프레임(사용자가 링크로 공유한 파일, node-id 50:4 / Flow 프레임 23:346)의 IA·문구를 최대한 따르되, 그 위에 Apple Design 스킬 기반 리퀴드 글래스·스프링 애니메이션·수필(隨筆) 모티프 로고를 얹은 상태.
- 라우트(`src/App.vue`): `/`, `/login`, `/profile`(진단 STEP 1/2), `/companies`, `/company`, `/analyzing`(연출용 대기 화면), `/report`(진단 STEP 2/2), `/mypage`.

### 더미 데이터로 채워진 인물/기업/리포트
전부 **김서현 / SK Hynix / AI Engineer / Fit Score 87점** 하나의 샘플이 홈 히어로, 진단 리포트, 마이페이지 진단 기록 최신행, 점수 추이 마지막 점에 반복 재사용되고 있습니다. 상세 필드는 `DUMMY_DATA.md` 참고.

### 이번 세션에서 반영된 주요 디자인 결정 (라이브 상태 기준)
- **색상**: dataviz 스킬의 `scripts/validate_palette.js`로 검증한 2계열 팔레트 사용 (`--series-1: #178E5C`, `--series-2: #9B5DE0`, light/dark 모두 CVD 체크 통과).
- **로고**: "Jobpill = Job + 수필(隨筆)" 컨셉. 원고지 칸 + 칸을 넘는 한 획 모티프(`src/components/Logo.vue`). 이전에 시도했던 알약(pill) 모티프는 폐기.
- **리퀴드 글래스**: `style.css`의 `.glass`, `.glass-neon`, `.glass-slate` 세 종류 — 무언가 위에 뜬 카드에만 사용(전체 배경에는 안 씀).
- **애니메이션**: `--spring: cubic-bezier(0.34, 1.56, 0.64, 1)` 하나로 통일. `useReveal.js` composable이 `IntersectionObserver`로 스크롤 리빌·게이지 성장·숫자 카운트업을 한 번에 처리.
- **인쇄/PDF**: `Report.vue`의 "PDF로 저장" 버튼은 `window.print()` 호출. 인쇄 스타일은 `style.css`의 `@media print` 블록 — **단, 라이브 버전에는 페이지 잘림 버그가 있고 수정본은 백업 폴더에 있음(위 0절 참고).**
- **"클로드 특유 디자인" 제거 작업 진행 중**: 사용자가 "클로드 특유의 디자인이 발견되면 즉시 수정하자"고 명시적으로 요청함. 지금까지 제거한 패턴: 헤드라인 한 구절만 형광펜 강조, 동일 카드 4연속 그리드(→ 표로 교체), 중점(·) 4개 이상 이어붙인 메타 문자열(→ `<dl>` 구조로 분해), 임의로 붙인 대문자 eyebrow 라벨. **이 작업도 현재 백업 폴더에만 온전히 반영돼 있고 라이브 Home.vue는 구버전입니다.** 새 세션에서 다른 화면들도 이 기준으로 한 번 더 훑어볼 여지가 있습니다(예: CompanyDetail.vue, Profile.vue 등은 아직 이 관점으로 재검토 안 함).

### 알려진 미스매치 / 다음에 손볼 만한 것
- `/company`, `/report`가 특정 id 없이 항상 SK Hynix 데이터 하나만 보여줌. 실제로는 `#/company/:code`, `#/report/:id`가 되어야 함.
- 홈 히어로 카드(87/92/54/78)와 Report.vue 역량 분포가 같은 수치를 중복 하드코딩 — 나중에 실 데이터 연동 시 하나의 source로 묶어야 함(`DUMMY_DATA.md` 15절 참고).
- `src/api.js`의 엔드포인트 경로는 실제로 **`career-compass/backend`의 컨트롤러 경로와 이미 일치**합니다(아래 3절). 즉 이 프론트를 실 API에 붙이는 작업 자체는 크게 어렵지 않을 것으로 보입니다 — 다만 응답 필드명(camelCase 등)까지 정확히 일치하는지는 DTO 대조가 필요합니다.

---

## 3. `career-compass/` — 실제 팀 풀스택 저장소 ⭐

`skala-MiniProject/src/`와는 **완전히 별개의, 자체 git을 가진 실제 프로젝트**입니다. GitHub 원격 저장소가 이미 있고 팀원들이 PR을 주고받으며 진행 중입니다.

```
career-compass/
├── frontend/     Vue 3 + Vite (아직 개발 초기 — DevelopmentStatusView.vue만 있는 스캐폴딩 수준)
├── backend/      Spring Boot 3.5.16 (Java 21, Gradle) — 모놀리식
├── docker-compose.yml   frontend + backend + PostgreSQL 17(pgvector) 한 번에 기동
└── .env.example
```

- 원격: `https://github.com/Jinhyung01/career-compass.git`
- 실행: `docker compose up` (Node/Java/PostgreSQL 로컬 설치 불필요). 포트 — Frontend `5173`, Backend `8080`, Postgres `5432`.
- **git log를 보면 백엔드 API가 이미 상당히 구현돼 있습니다**: Mock 인증(회원가입/로그인), 프로필 CRUD, 기업 검색/상세, 분석 리포트 Mock API, 기업 검색 상세 API + 테스트까지 머지된 상태 (`git log --oneline` 기준 커밋 20개 이상, PR 7개 머지).
- 패키지 구조(`backend/src/main/java/com/jobfeel/careercompass/`): `auth`, `user`(profile), `company`, `analysis`(report), `common` 5개 도메인 패키지로 나뉘어 있음.

### API 경로 대조 — `src/api.js`와 실제 백엔드 컨트롤러가 이미 일치함

| `skala-MiniProject/src/api.js` | `career-compass` 백엔드 실제 경로 |
|---|---|
| `POST /auth/register` | `AuthController` → `POST /api/v1/auth/register` |
| `POST /auth/login` | `AuthController` → `POST /api/v1/auth/login` |
| `GET /me/profile` | `ProfileController` → `GET /api/v1/me/profile` |
| `PUT /me/profile` | `ProfileController` → `PUT /api/v1/me/profile` |
| `GET /companies` | `CompanyController` → `GET /api/v1/companies` |
| `GET /companies/:companyCode` | `CompanyController` → `GET /api/v1/companies/{companyCode}` |
| `POST /reports` | `ReportController` → `POST /api/v1/reports` |
| `GET /reports` | `ReportController` → `GET /api/v1/reports` |
| `GET /reports/:id` | `ReportController` → `GET /api/v1/reports/{reportId}` |
| `GET /reports/:id/download` | `ReportController` → `GET /api/v1/reports/{reportId}/download` |

`api.js`의 `BASE_URL` 기본값이 `/api/v1`이므로 **경로 프리픽스까지 그대로 맞습니다.** 실 연동 시 `VITE_API_BASE_URL=http://localhost:8080/api/v1`로 맞추고 응답 DTO 필드명만 대조하면 될 가능성이 높습니다(단, 직접 검증은 안 했습니다 — DTO 파일은 `backend/src/main/java/.../*/dto/`에 있음).

### `career-compass-backend-tasks/*.md` — 팀 작업 분담 문서
백엔드를 담당자 A/B/C/D 4명이 나눠 맡는 구조입니다:

| 문서 | 담당 | 브랜치 | 범위 |
|---|---|---|---|
| `00-common.md` | 전원 | — | 공통 규칙(에러 응답 포맷 등), 이 문서부터 읽게 되어 있음 |
| `01-A-common-auth.md` | A | `feat/mock-auth` | 공통 오류 처리 + Mock 인증 |
| `02-B-profile.md` | B | `feat/user-profile` | `users`, `job_seeker_profile`, `job_seeker_tech`, `profile_content` 테이블 + 프로필 API |
| `03-C-seed-company.md` | C | `feat/mock-seed-data`, `feat/company-search` | `job_position`, `tech_stack`, `company`, `company_insight`, `company_insight_tech` 테이블 + Mock Seed + 기업 검색 API |
| `04-D-report.md` | D | `feat/report-api` | `analysis_report`, `analysis_report_company`, `analysis_report_item` 테이블 + 리포트 API |

`API_명세서_v2.pdf`(8페이지, 최상위 디렉토리)가 이 API들의 정식 명세서로 보입니다 — 아직 내용을 상세히 읽지는 않았습니다.

**이 저장소가 이번 세션의 "DB 작업 담당자"가 실제로 작업 중인 그 저장소일 가능성이 높습니다.** `DUMMY_DATA.md`를 요청한 사용자 발화("따로 DB를 빼서 관리할 거야 / 그 작업을 하는 사람이 있어")와 정확히 맞아떨어집니다. 위 표의 `job_position`, `tech_stack`, `company`, `company_insight` 같은 실제 테이블명을 참고하면 `DUMMY_DATA.md` 15절의 "엔티티 후보 요약"을 훨씬 더 정확하게 다시 쓸 수 있습니다.

---

## 4. `DUMMY_DATA.md` — 무엇이고, 어떤 한계가 있는지

사용자가 "프런트 작업본에 들어간 더미 데이터를 DB 담당자에게 넘길 수 있게 md로 뽑아달라"고 요청해 만든 문서. `skala-MiniProject/src/` 전체(스크립트 블록 + 템플릿에 직접 박힌 값 전부)를 훑어서 화면 흐름 순서로 15개 섹션에 정리했습니다.

**한계 (새 세션에서 바로 알아야 할 것)**:
1. 위 0절에서 설명한 대로 **되돌아간 이후의 라이브 `src/`를 기준**으로 작성됐습니다. 백업을 복구하면 이 문서도 다시 뽑아야 합니다 — 특히 3절(기업 21개 vs 현재 6개), 9절(추천 프로젝트 P1~P3가 통째로 빠짐)이 크게 바뀝니다.
2. `career-compass-backend-tasks/`의 실제 테이블 스키마를 아직 대조하지 않았습니다. 15절 "엔티티 후보 요약"은 순수히 프런트 더미 데이터만 보고 추측한 것이라, 실제 `company`, `company_insight`, `job_position`, `analysis_report*` 테이블 구조와 필드명이 다를 수 있습니다. **DB 담당자에게 넘기기 전에 `career-compass/backend`의 실제 Entity/Migration 파일과 한 번 대조하는 걸 권장합니다** (`career-compass/backend/src/main/java/.../domain/*.java`, `career-compass/backend/src/main/resources/db/migration/` 등).

---

## 5. 보류 중인 작업 (사용자가 "대기"라고 명시함)

아래는 이미 백업 폴더에 결과물이 있는 것으로 확인됐지만, 사용자가 명시적으로 "일단 대기"라고 한 항목입니다. **사용자가 먼저 지시하기 전까지 손대지 마세요.**

- [ ] PDF 잘림 수정 (→ 백업의 `style.css`에 이미 완료)
- [ ] 리포트에 추천 프로젝트 등 더미데이터 보강 (→ 백업의 `Report.vue`에 이미 완료, P1~P3)
- [ ] 기업관에 기업 15개 추가(총 21개) (→ 백업의 `Companies.vue`에 이미 완료)
- [ ] 기업 로고 가져오기("로고 떼와" = 실제 기업 로고를 파비콘 등에서 가져오라는 뜻으로 해석, 삭제 아님) (→ 백업의 `CompanyLogo.vue`에 이미 완료)

새 세션 시작 시 **가장 먼저** 사용자에게 "백업 폴더에 이미 완성된 버전이 있는데 복구할지, 아니면 이대로 둘지" 확인하세요.

---

## 6. 다음 세션 시작 시 권장 순서

1. 이 문서(`HANDOFF.md`) 전체를 사용자에게 요약 보고 — 특히 0절(되돌아간 작업 발견)과 5절(보류 작업).
2. 백업 복구 여부를 사용자에게 확인받는다.
3. 복구한다면: `cp -r .restore-backup-20260903-161621/src/* src/` → `npm run build`로 검증 → `DUMMY_DATA.md` 재작성 → 정리되면 `.restore-backup-20260903-161621/`은 삭제해도 되는지 사용자에게 확인.
4. `DUMMY_DATA.md`를 넘기기 전에 `career-compass/backend`의 실제 Entity/Migration과 필드명을 한 번 대조해서 정확도를 높인다 (4절 참고).
5. 이후 작업은 사용자 지시에 따름 — 이 시점까지는 아직 다음 방향(실 API 연동 착수인지, 디자인 계속 다듬기인지)이 정해지지 않았습니다.
