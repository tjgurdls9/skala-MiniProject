# Jobpill 더미 데이터 목록

프런트엔드 작업본(`src/`)에 하드코딩되어 있는 모든 더미 데이터를 화면·컴포넌트별로 빠짐없이 정리했습니다.
DB 설계·시딩 작업자가 참고할 수 있도록 **어떤 화면의 어떤 파일에서 쓰이는지**, **필드 구조**, **원문 텍스트**를 그대로 옮겼습니다.

- 소스 기준: `src/` 전체 (`views/`, `components/`, `data/`, `api.js`, `App.vue`)
- 작성일: 2026.09.03 · 최종 갱신: 2026.09.04 (문구 정돈 · 건당 결제 전환 반영)
- 표기: 코드에 있는 그대로(한글 라벨, 단위 문자열 등)를 보존했고, 필드명은 실제 변수명을 그대로 씁니다.
- `career-compass/frontend/src/`는 이 `src/`의 동일 사본입니다. 한쪽만 고치면 어긋납니다.

---

## 목차

1. [라우트 구조](#1-라우트-구조)
2. [사용자(User) / 세션](#2-사용자user--세션)
3. [기업(Company)](#3-기업company)
4. [기업 상세 — 직무(Job)](#4-기업-상세--직무job)
5. [기업 상세 — 현직자 인터뷰(Insight)](#5-기업-상세--현직자-인터뷰insight)
6. [기업 상세 — 기업 문화 지표(Culture)](#6-기업-상세--기업-문화-지표culture)
7. [진단(Diagnosis) 입력 폼 — 프로필](#7-진단diagnosis-입력폼--프로필)
8. [진단 진행 중 — 분석 스텝](#8-진단-진행-중--분석-스텝)
9. [진단 리포트(Report)](#9-진단-리포트report)
10. [마이페이지 — 진단 기록 / 점수 추이](#10-마이페이지--진단-기록--점수-추이)
11. [마이페이지 — 결제 내역 (건당 결제)](#11-마이페이지--결제-내역-건당-결제)
12. [홈 — 통계 · 카피](#12-홈--통계--카피)
13. [공통 UI 텍스트 (로그인·푸터·GNB)](#13-공통-ui-텍스트-로그인푸터gnb)
14. [API 계약 (연동 시 맞춰야 할 엔드포인트)](#14-api-계약-연동-시-맞춰야-할-엔드포인트)
15. [정리 — 엔티티 후보 요약](#15-정리--엔티티-후보-요약)

---

## 1. 라우트 구조

`src/App.vue` — 해시 라우터. `auth: true`인 화면은 비로그인 시 위에 로그인 안내 팝업(`AuthGate.vue`)이 뜹니다.

| 경로 | 화면 컴포넌트 | 인증 | 설명 |
|---|---|---|---|
| `/` | `Home.vue` | — | 랜딩 |
| `/login` | `Login.vue` | — | 로그인 / 회원가입 (탭 전환) |
| `/profile` | `Profile.vue` | 필요 | 컨설팅 STEP 01/02 — 프로필 입력 + 건당 결제 |
| `/companies` | `Companies.vue` | — | 기업 정보 목록 |
| `/company/:companyCode` | `CompanyDetail.vue` | — | 기업 상세 |
| `/report` | `Reports.vue` | 필요 | 리포트 목록 |
| `/report/:id` | `Report.vue` | 필요 | 리포트 본문 (기본값 `r1`) |
| `/mypage` | `MyPage.vue` | 필요 | 기본 정보 · 진단 기록 · 점수 추이 · 결제 내역 |

- 분석 진행 화면은 **별도 라우트가 아니라** `/profile` 위에 뜨는 팝업입니다(`AnalyzingModal.vue`). 과거 문서의 `/analyzing` 라우트는 없습니다.
- `/company`, `/report`는 이미 id를 받습니다(`#/company/SK001`, `#/report/r1`).

---

## 2. 사용자(User) / 세션

`src/auth.js` — 서버 인증 없이 `localStorage['jobpill.user']`에 `{ name, email }`만 저장하는 로컬 세션입니다.

| 필드 | 값 | 출처 |
|---|---|---|
| 이름 | `김서현` | `MyPage.vue` profile, `reports.js` meta.name |
| 이메일 | `seohyeon.kim@email.com` | `MyPage.vue` profile (로그인 값이 있으면 그쪽 우선) |
| 연락처 | `010-1234-5678` | `MyPage.vue` profile |
| 학교 · 전공 | `한국대학교 산업공학과` | `MyPage.vue` profile |
| 학력 | `4년제 졸업 예정` | `MyPage.vue` profile |
| 희망 직무 | `AI Engineer` | `MyPage.vue` profile |
| 관심 기업 | `SK Hynix` | `MyPage.vue` profile |
| 결제 수단 | `신한카드 ···· 1024` | `MyPage.vue`, `PaymentModal.vue` (두 곳에 중복 하드코딩) |

---

## 3. 기업(Company)

`src/data/companies.js` — **화면에 하드코딩된 배열이 아니라 별도 데이터 모듈입니다.** `companies`(105곳)와 집계 `totals`를 export 합니다.

```
{
  companyCode: string    // 'SK001' — 라우트 키
  companyName: string    // 'SK Hynix'
  domain: string         // 'skhynix.com' — 로고 파비콘 조회용
  role: string           // 대표 노출 직무, "A · B" 형태로 복수 표기
  people: number         // 인터뷰한 현직자 수
  insight: number        // 인사이트(인터뷰 발췌) 건수
  interviewed: string    // 최근 인터뷰 일자 "YYYY.MM.DD"
  freshness: number      // 최신성 % (66~96 분포)
  jobs:    [ ... ]       // 4절
  insiders:[ ... ]       // 5절
  culture: [ ... ]       // 6절
}
```

집계 `totals` (홈·기업 목록 상단 통계가 이 값을 씁니다 — 하드코딩 아님):
- `companies`: 105 · `insiders`: 791 (people 합) · `insights`: 2235 (insight 합)

샘플 (선두 1건):

| companyCode | companyName | domain | role | people | insight | interviewed | freshness |
|---|---|---|---|---|---|---|---|
| SK001 | SK Hynix | skhynix.com | 양산 기술 · AI Engineer | 12 | 34 | 2026.08.21 | 94 |

> 과거 문서의 `sector`·`area`·`fresh`·`match`·`primary` 컬럼은 **없습니다.** 그래서 목록 화면에도 업종·지역 필터 패널을 두지 않았습니다. 정렬만 3종 제공합니다: 최근 인터뷰순 / 현직자 많은 순 / 진단한 기업 먼저.
> 매치 점수는 진단을 실행한 기업에만 있습니다(`reports.js`의 `matchByCompany`). 나머지는 "진단 전"으로 표시합니다.

---

## 4. 기업 상세 — 직무(Job)

`src/data/companies.js`의 `jobs` 배열 → `CompanyDetail.vue`에서 표시.

```
{ name: string, people: number(해당 직무 인터뷰 인원), fit: number(요구 충족 기준 점수), need: string[] }
```

샘플 (SK Hynix):

| name | people | fit | need |
|---|---|---|---|
| 양산 기술 | 5 | 87 | 공정 데이터 분석, Python, 수율 개선 사례 |

`need`가 4절의 "요구 항목"이며, 9절 리포트의 `demands.items`와 같은 개념입니다.

---

## 5. 기업 상세 — 현직자 인터뷰(Insight)

`src/data/companies.js`의 **`insiders`** 배열 (변수명이 `insights`가 아닙니다).

```
{ role: string, years: string, date: string, current: boolean, quote: string, tag: string }
```

- `current: false`면 "현재 재직 아님"으로 구분해 표시합니다.
- `tag` 값 예: `서류 기준`, `기술 스택`, `전형`, `기업 문화`

샘플 (SK Hynix 1건):

| role | years | date | current | tag |
|---|---|---|---|---|
| AI Engineer | 3년차 | 2026.08.21 | true | 서류 기준 |

> `quote`는 현직자의 실제 발화체라 문서 전체의 –습니다체 통일 대상에서 제외합니다(예: "…적어주세요.").

---

## 6. 기업 상세 — 기업 문화 지표(Culture)

`src/data/companies.js`의 `culture` 배열.

```
{ name: string, value: number(0-100), note: string(근거) }
```

항목 4종 고정: `문서 기반 협업` · `실험 · 배포 속도` · `온보딩 체계` · `주니어 성장 지원`

샘플 (SK Hynix):

| name | value | note |
|---|---|---|
| 문서 기반 협업 | 88 | 인터뷰 12건 중 9건에서 언급 |

`note`가 근거(몇 건 중 몇 건)를 항상 함께 제공한다는 점이 이 서비스의 원칙과 이어집니다.

---

## 7. 진단(Diagnosis) 입력폼 — 프로필

`src/views/Profile.vue`

STEP 01(기업·직무·스택) / STEP 02(학력·경력·자격증·프로젝트) 2단계 폼입니다.

### 7-1. 진단 기록 사이드바 — `src/components/ConsultAside.vue`, 배열 `sessions`
```
{ id: string, company: string, role: string, date: string, matched?: 'company'|'role', fold?: boolean }
```

| id | company | role | date | matched | fold |
|---|---|---|---|---|---|
| r1 | SK Hynix | AI Engineer | 2026.09.03 | — | — |
| r2 | SK Hynix | 양산 기술 | 2026.08.08 | role | — |
| r4 | 포스코 퓨쳐엠 | 반도체 · DX | 2026.08.01 | company | — |
| r5 | 카카오페이 | Backend Engineer | 2026.07.22 | — | true |
| r6 | 무신사 | Data Analyst | 2026.07.14 | — | true |

`id`는 9절 리포트와 1:1로 이어집니다(`#/report/:id`). `fold: true` 항목은 "전체 기록 보기"를 눌러야 펼쳐집니다.

### 7-2. 입력 폼 선택지

- 지원 기업(`companies`): **정적 목록이 아니라 `data/companies.js`의 `companyName` 전체**에서 뽑습니다.
- 희망 직무(`roles`): 마찬가지로 `companies.js`의 모든 `jobs[].name`을 중복 제거·정렬해 씁니다.
- 기술 스택(`allStacks`, 중복 선택, 정적): `Python, RAG, FastAPI, PyTorch, Kubernetes, Airflow, SQL, React, TypeScript`
- 학력(`degrees`, 정적): `4년제 졸업 예정, 4년제 졸업, 전문대 졸업, 석사 재학, 석사 졸업`

### 7-3. 폼 초기값 (`form`)
```
{
  company: 'SK Hynix',
  role: 'AI Engineer',
  stacks: ['Python', 'RAG', 'FastAPI'],
  degree: '4년제 졸업 예정',
  careers: [
    { company: 'SK ATX', period: '2025.01 - 2025.12', type: '인턴, 퇴직', role: 'Product Engineer' },
    { company: 'Samsung 전자', period: '2024.10 - 2024.12', type: '인턴, 퇴직', role: '생산관리' }
  ],
  certs: ['정보처리기사', '빅데이터 분석기사', 'OPic(IH)'],
  project: '',
  files: ['~~결과 보고서.pdf', 'xx 사업 기획안.docs']
}
```

### 7-4. 우측 "진단 안내" 카드 (`agent`, STEP에 따라 분기)

| STEP | 제목 | 설명 |
|---|---|---|
| 01 | 직무만 정해도 / 진단을 시작할 수 있습니다. | 기업까지 적으면 공고 데이터를 / 더 정확하게 좁힙니다. |
| 02 | 프로필만 입력해도 / 진단할 수 있습니다. | 기업까지 적으면 공고 데이터를 기준으로 / 더 정확하게 진단합니다. |

체크리스트는 더미가 아니라 폼 값에서 계산됩니다.
STEP 01: 희망 직무 입력 완료 / 기술 스택 N개 선택 · STEP 02: 지원 기업 선택 완료 / 자격증 N개 등록 / 파일 N개 업로드

하단 문장은 선택한 기업의 실제 값을 씁니다: "{기업명}에는 인터뷰에 응해주신 현직자가 {people}명 있습니다. 최근 인터뷰는 {interviewed}이고, 인사이트는 {insight}건입니다."

### 7-5. 결제 (건당) — `src/components/PaymentModal.vue`

STEP 02의 버튼은 결제 전 `결제하기`, 결제 후 `진단하기`로 바뀝니다. 결제창은 실제 PG 연동 없는 목업이며 700ms 뒤 성공 처리합니다.
- 단가 상수 `PRICE = 5900` (₩5,900)
- 결제 수단 표기: `신한카드 ···· 1024` (하드코딩)

---

## 8. 진단 진행 중 — 분석 스텝

`src/components/AnalyzingModal.vue` — 별도 라우트가 아니라 컨설팅 화면 위에 뜨는 팝업입니다. 배열 `steps`는 연출용 타이머(합계 약 8.2초)이며, 종료 시 `#/report/r1`로 이동합니다.

```
{ label: string, note: string, ms: number(연출 지속시간) }
```

| label | note | ms |
|---|---|---|
| 프로필 정리 | 입력한 학력·경력·자격증·프로젝트를 항목으로 나눕니다 | 1400 |
| 현직자 인터뷰 검색 | 해당 기업 현직자 인터뷰를 불러옵니다 | 2200 |
| 항목별 대조 | 요구 수준과 내 수준을 역량 단위로 맞춰봅니다 | 2600 |
| 리포트 작성 | 강점·부족한 역량·보완할 점을 순서대로 정리합니다 | 2000 |

상단 캡션은 고정값이 아니라 폼에서 넘어온 `company · role`입니다.
본문: "현직자 인터뷰와 입력한 프로필을 항목별로 대조하는 중입니다."
하단: "이 창을 닫아도 분석은 계속됩니다. 결과는 마이페이지에 저장됩니다."

> 실제 연동 시 이 화면은 `api.js`의 `waitForReport(reportId)`(2초 폴링)로 대체되는 자리입니다. 현재는 백엔드 상태값 대신 프런트에서 타이머로 흉내만 냅니다.

---

## 9. 진단 리포트(Report)

`src/data/reports.js` (데이터) + `src/views/Report.vue` (본문) + `src/views/Reports.vue` (목록)

**단일 하드코딩이 아니라 리포트 6건이 배열로 있습니다.** `reports` 배열과 조회 헬퍼 `reportById(id)`, 기업명→최고점 매핑 `matchByCompany`를 export 합니다. 화면은 `#/report/:id`로 열고 기본값은 `r1`입니다.

### 9-1. 리포트 목록 (배열 `reports`)

| id | code | companyName | positionName | date | fitScore | grade |
|---|---|---|---|---|---|---|
| r1 | JP-0002 | SK Hynix | AI Engineer | 2026.09.03 | 87 | A |
| r2 | JP-0001 | SK Hynix | 양산 기술 | 2026.08.08 | 75 | B+ |
| r3 | JP-0003 | SK AX | 반도체 · AI Engineer | 2026.08.04 | 82 | A- |
| r4 | JP-0004 | 포스코 퓨쳐엠 | 반도체 · DX | 2026.08.01 | 79 | A- |
| r5 | JP-0005 | 카카오페이 | Backend Engineer | 2026.07.22 | 68 | B |
| r6 | JP-0006 | 무신사 | Data Analyst | 2026.07.14 | 71 | B+ |

### 9-2. 리포트 1건의 구조

```
{
  id: string,
  meta:     { name, companyName, positionName, code, date },
  overall:  { fitScore: number, grade: string },
  headline: string,
  skills:   [{ name, value: 0-100, weak?: boolean }],
  details:  [{ group, name, score, judge, tone: 'good'|'bad'|'' }],
  insight:  { title, lines: string[] },
  demands:  { of: number(인터뷰 인원), items: [{ item, count }] },
  strengths: string[],
  gaps:      string[],
  jobFit:   [{ rank, job, score, note }],
  actions:  [{ no, title, meta(소요시간) }],
  recommendedProjects: [{ no, title, tag, source, why, todo: string[], output, weeks, resume }],
  resumeHighlights:    [{ before, after, why }],
  questions:           [{ q, why }]
}
```

### 9-3. r1 샘플 값

- `overall`: `{ fitScore: 87, grade: 'A' }`
- `skills`: 직무 이해 92 / 정량 성과 54(`weak`) / 협업 경험 78
- `details` 예: `{ group: '직무 이해', name: '도메인 지식', score: 94, judge: '우수', tone: 'good' }`
- `insight`: 제목 "정량 성과가 비어 있습니다" + 근거 2줄
- `demands`: `of: 12`, 최다 항목 "지표 개선 사례 (전후 수치)" 9명 언급
- `jobFit`: AI Engineer 87 / MLOps Engineer 81 / Data Engineer 76
- `actions` 예: `{ no: '01', title: '기존 프로젝트의 전후 지표 복원', meta: '예상 1주' }`
- `resumeHighlights` 예: before/after/why 3필드 (문자열 배열 아님)

### 9-4. 화면 구성 순서

종합 결과 → 역량별 점수 → 세부 역량 → 강점·보완점 → 현직자가 꼽은 요구 항목 → 직무 적합도 → 우선 보완 과제 → 추천 프로젝트 → 이력서 문장 고치기 → 예상 면접 질문

> 지원자 모집단 DB가 없어 **백분위·비교 모수·지원자 등급 분포(히스토그램)는 두지 않습니다.** 대신 현직자 인터뷰로 알 수 있는 요구 항목 빈도와 추천 프로젝트를 넣었습니다. 과거 문서에 있던 `bins`/`Histogram.vue`/`tasks`(주간 체크리스트)는 현재 코드에 없습니다.

## 10. 마이페이지 — 진단 기록 / 점수 추이

`src/views/MyPage.vue` — 좌측 스티키 인덱스 4개 섹션: 기본 정보 / 진단 기록 / 점수 추이 / 결제 내역

### 10-1. 요약 통계 — 배열 `stats` (하드코딩 아님, `reports.js`에서 계산)

```
{ name: string, value: number, unit: string, note?: string }
```

| name | 계산식 | 현재 값 |
|---|---|---|
| 진단 기업 | 리포트의 고유 `companyName` 개수 | 5개 |
| 제안된 프로젝트 | 전체 리포트의 `recommendedProjects` 합계 | 13개 |
| 적합 기업 | `fitScore >= 85`인 리포트의 고유 기업 수 | 1개 |

적합 기업 칸에는 기준 문구가 붙습니다: "85점 이상을 기준으로 합니다" (상수 `FIT_LINE = 85`)

> 이 세 값은 리포트가 늘면 자동으로 따라갑니다. DB 연동 시에도 저장값이 아니라 집계 쿼리로 두는 편이 맞습니다.

### 10-2. 진단 기록 테이블 — 배열 `rows` (하드코딩)
```
{ id: string, date: string, title: string, match: string, delta: string, status: string, live?: boolean }
```

| id | date | title | match | delta | status | live |
|---|---|---|---|---|---|---|
| r1 | 2026.09.03 | SK Hynix AI Engineer 진단 | 87% | +12 | 최신 | true |
| r2 | 2026.08.08 | SK Hynix 양산 기술 진단 | 75% | +7 | 완료 | — |
| r3 | 2026.08.04 | SK AX 반도체 · AI Engineer 진단 | 82% | +3 | 완료 | — |
| r4 | 2026.08.01 | 포스코 퓨쳐엠 반도체 · DX 진단 | 79% | +11 | 완료 | — |
| r5 | 2026.07.22 | 카카오페이 Backend Engineer 진단 | 68% | — | 보관 | — |
| r6 | 2026.07.14 | 무신사 Data Analyst 진단 | 71% | — | 보관 | — |

`id`는 9절 리포트와 이어집니다(행을 누르면 `#/report/:id`).

### 10-3. 점수 추이 그래프 — 배열 `trend` (하드코딩)
```
{ date: string, value: number, last?: boolean }
```

| date | value | last |
|---|---|---|
| 07.14 | 71 | — |
| 07.22 | 68 | — |
| 08.01 | 79 | — |
| 08.04 | 82 | — |
| 08.08 | 75 | — |
| 09.03 | 87 | true |

> `rows`·`trend`는 아직 `reports.js`와 별개로 손으로 적은 값입니다(값 자체는 일치). DB 설계 시 `report_id` 하나로 묶어 세 곳이 어긋나지 않게 해야 합니다.

---

## 11. 마이페이지 — 결제 내역 (건당 결제)

`src/views/MyPage.vue`

**구독(월정액) 모델은 삭제됐습니다.** 진단 1건마다 결제하는 건당 과금이며, 결제 1건 = 리포트 1건입니다.

### 11-1. 단가
- 상수 `PRICE = '₩5,900'` (`PaymentModal.vue`의 `PRICE = 5900`과 같은 값, 현재는 두 파일에 각각 하드코딩)
- 안내 문구: "진단은 건별로 결제됩니다. 1회 진단권 ₩5,900."

### 11-2. 결제 내역 — 배열 `payments`
```
{ id: string, date: string, title: string, amount: string }
```
`rows`(10-2)를 그대로 매핑해 만듭니다 — 진단 1건당 결제 1건이라 항목·날짜가 동일하고, 금액만 일괄 ₩5,900입니다.

표시는 애니메이션·호버 효과 없는 정적 목록이며, 상태(결제완료) 열은 두지 않습니다.

### 11-3. 결제 수단
- `신한카드 ···· 1024` (템플릿 하드코딩)

> DB 설계 시: `payments` 테이블을 `report_id`(FK) · `amount` · `paid_at` · `method` 정도로 두면 됩니다. PG 실연동 시 클라이언트가 보낸 "성공"을 믿으면 안 되고 서버가 승인 API를 별도로 호출해 확정해야 하는데, 현재 프런트 목업에는 그 단계가 없습니다.

---

## 12. 홈 — 통계 · 카피

`src/views/Home.vue`

### 12-1. 기존 서비스 대비 비교 — 배열 `alternatives`
```
{ name: string, limit: string, gap: string }
```

| name | limit | gap |
|---|---|---|
| 채용 공고 | 기업이 직접 쓴 홍보 문구. 실제 업무와 온도차가 크다. | "Python 활용 능력"이 어느 수준인지 알 수 없음 |
| 잡플래닛 · 블라인드 | 만족도 후기 중심이라 내 역량과 대조가 안 된다. | 평점은 있지만 내 기준점이 없음 |
| 취업 커뮤니티 | 정보가 비구조적이고 검색이 되지 않는다. | 같은 질문이 매년 반복됨 |
| AI 챗봇 | 공개된 정보만 안다. | 인터뷰로만 얻는 내부 기준은 모름 |

### 12-2. 진행 흐름 — 배열 `steps`
```
{ no: string, title: string, body: string, cta?: boolean }
```

| no | title | body | cta |
|---|---|---|---|
| 01 | 프로필 입력 | 학력·경력·자격증·프로젝트를 입력합니다. | — |
| 02 | 기준 대조 | 채용 공고와 현직자 인터뷰에 내 프로필을 항목 단위로 대조합니다. | — |
| 03 | 리포트 확인 | 부족한 역량과 보완 과제를 우선순위 순서로 받습니다. | true ("진단 시작하기" → `#/profile`) |

### 12-3. 히어로 유리 카드 — 배열 `questions`

"이 서비스가 답하는 세 가지 질문". (과거 문서의 "TODAY'S DIAGNOSIS 87%" 점수 카드는 현재 없습니다.)

| no | q | a |
|---|---|---|
| 01 | 내 프로필에서 무엇이 부족한가 | 공고 기준과 대조해 부족한 경험을 찾습니다. |
| 02 | 무엇을 실행해야 하는가 | 다음에 할 프로젝트를 제안합니다. |
| 03 | 어떻게 합격권에 가까워지는가 | 다시 진단해 변화를 확인합니다. |

### 12-4. 하단 통계 타일 — 배열 `stats`

**하드코딩이 아니라 `data/companies.js`의 `totals`에서 계산**합니다(`companies.length`, `people` 합, `insight` 합). 카운트업 애니메이션이 `toLocaleString('ko-KR')`로 천단위를 찍습니다.

```
{ cap: string, value: number, suffix: string, note: string }
```

| cap | value | suffix | note |
|---|---|---|---|
| 등록 기업 | totals.companies | 곳 | 현직자 인터뷰를 확보한 기업 |
| 인사이트 | totals.insights | 건 | 인터뷰에서 정리한 요구 항목 |
| 인터뷰 참여 현직자 | totals.insiders | 명 | 등록 기업 현직자 기준 |

+ 고정 타일(데이터 아님, 링크 카드): "새 진단 / 진단 시작하기 / 프로필 입력부터 시작합니다." (`#/profile`)

### 12-5. 카피 (참고용)

- 히어로 킥커: `AI CAREER CONSULTING` (서비스에서 유일하게 영문 대문자를 쓰는 자리)
- 히어로 헤드라인: "현직자가 보는 기준으로 / 당신의 부족한 경험을 찾습니다."
- 리드: "채용 공고와 현직자 인터뷰를 대조해, 다음 프로젝트를 제시합니다."
- 히어로 하단: "진단 결과는 계정에 저장되고, 다시 진단하면 이전 결과와 비교됩니다."
- 문제 정의: "근거가 없으니 전략이 '일단 다'가 됩니다."
- 결론: "위 채널은 정보를 보여줄 뿐, 내 프로필과 대조하지는 않습니다."
- 아웃트로: "읽을 정보는 이미 많습니다. 대조할 기준이 없었을 뿐입니다."
- 운영 원칙: "데이터의 출처와 시점을 함께 제공합니다."

---

## 13. 공통 UI 텍스트 (로그인·푸터·GNB)

### 13-1. `Login.vue`
- 탭 전환형(로그인 / 회원가입) 한 화면
- 제목: "로그인하고 진단 시작하기" / "가입하고 진단 시작하기"
- 안내: "진단 기록과 리포트는 계정에 저장됩니다." / "가입 후 프로필을 입력하면 진단을 시작할 수 있습니다."
- 제출 버튼: "로그인" / "가입하고 진단 시작하기"
- **이동 규칙: 로그인 → `#/` (메인), 회원가입 → `#/profile`**
- 소셜 로그인 버튼: 카카오 / 구글 / 네이버 (연동 없음, UI만)
- 하단 고지: "업로드 파일은 용도 외에 이용되지 않습니다."
- 인증은 서버 없이 `localStorage['jobpill.user']` 로컬 세션만 만듭니다(`src/auth.js`).

### 13-2. `AuthAside.vue` (로그인 화면 좌측 패널)
- 헤드라인: "현직자 기준으로 / 부족한 경험을 확인하세요"
- 서브: "진단 결과는 계정에 저장되고, 다시 진단하면 이전 결과와 비교됩니다."
- 리스트: "✓ 리포트 자동 저장 / ✓ 현직자 인터뷰 교차 검증 / ✓ 업로드 파일 용도 외 미이용"

### 13-3. `TopNav.vue` — GNB 메뉴 (정적 상수 `menu`)
```
{ key: 'profile',   label: '컨설팅',   href: '#/profile' }
{ key: 'companies', label: '기업 정보', href: '#/companies' }
{ key: 'report',    label: '리포트',   href: '#/report' }
```
우측: 마이페이지 · (비로그인) 로그인/회원가입 · (로그인) "{이름} 님" + 로그아웃

### 13-4. `AuthGate.vue` (비로그인 접근 차단 팝업)
- "{메뉴명}은/는 로그인 후 이용할 수 있습니다." — 조사는 받침 유무로 계산합니다(컨설팅**은** / 리포트**는** / 마이페이지**는**).
- 대상 라우트: `/profile`(컨설팅) · `/report`(리포트) · `/mypage`(마이페이지)

### 13-5. `SiteFooter.vue`
- 브랜드 설명: "현직자에게 직접 물어 정리한 기준으로, 내 프로필을 대조합니다."
- 서비스 링크: 컨설팅(`#/profile`), 기업 정보(`#/companies`), 리포트(`#/report`) — GNB와 순서 일치
- 계정 링크: 로그인 · 회원가입(`#/login`), 마이페이지(`#/mypage`)
- 정책 링크: 이용약관, 개인정보 처리방침 (둘 다 `#/`로 연결, 실제 페이지 없음)
- 하단 고지: "인터뷰 자료는 관리자가 검수해 등록하며 인터뷰 시점을 함께 표시합니다. 업로드 파일은 용도 외에 이용되지 않습니다." / "© 2026 Jobpill"

---

## 14. API 계약 (연동 시 맞춰야 할 엔드포인트)

`src/api.js` — 현재 프런트가 기대하는 REST 인터페이스입니다. 위 더미 데이터들은 실제로는 이 엔드포인트들의 응답으로 대체되어야 합니다.

```
POST   /auth/register
POST   /auth/login
GET    /me/profile
PUT    /me/profile
GET    /companies?{query}
GET    /companies/:companyCode
POST   /reports
GET    /reports/:id
GET    /reports?{size, ...query}
GET    /reports/:id/download
```

- `BASE_URL` 기본값: `/api/v1` (환경변수 `VITE_API_BASE_URL`로 override 가능)
- 인증: `localStorage.accessToken` → `Authorization: Bearer` 헤더
- `waitForReport(reportId)`: `/reports/:id`를 2초 간격 폴링, `status`가 `COMPLETED`/`FAILED`가 될 때까지 대기 (→ 8절 Analyzing 화면이 이 흐름을 흉내만 내는 중)

> 현재 어떤 화면도 실제로 `api.js`를 호출하지 않습니다(전부 로컬 하드코딩 배열). 연동 시점에 화면 데이터를 이 엔드포인트 응답으로 바꿔 끼우면 됩니다.

---

## 15. 정리 — 엔티티 후보 요약

더미 데이터를 훑어보면 아래와 같은 테이블/엔티티가 필요해 보입니다 (설계 참고용 제안, 강제 아님):

| 엔티티 | 주요 필드 | 관련 절 |
|---|---|---|
| **User** | name, email, phone, school, degree, target_role, target_company | 2, 10-1 |
| **Company** | company_code, name, domain, role, people, insight_count, interviewed_at | 3 |
| **Job** (Company 하위) | company_id, name, required_keywords[] | 4 |
| **CompanyInsight** (인터뷰 발췌) | company_id, role, years, interviewed_at, is_current, quote, tag | 5 |
| **CompanyCulture** | company_id, name, value, note | 6 |
| **UserProfile** | user_id, target_company, target_role, stacks[], degree, certs[] | 7 |
| **Career** (UserProfile 하위) | profile_id, company, period, type, role | 7-3 |
| **UploadFile** | profile_id, filename, uploaded_at | 7-3 |
| **Report** | user_id, company_id, position_name, code, fit_score, grade, headline, created_at | 9 |
| **ReportSkill** (역량별 점수) | report_id, name, value, is_weak | 9-2 |
| **ReportDetail** (세부 역량) | report_id, group, name, score, judge, tone | 9-2 |
| **ReportDemand** (현직자 요구 항목) | report_id, item, count, interviewee_total | 9-2 |
| **ReportJobFit** (직무 적합도) | report_id, rank, job, score, note | 9-2 |
| **ReportAction** (우선 보완 과제) | report_id, no, title, estimated | 9-2 |
| **ReportProject** (추천 프로젝트) | report_id, no, title, tag, source, why, todo[], output, weeks, resume | 9-2 |
| **ReportResumeRewrite** | report_id, before, after, why | 9-2 |
| **ReportQuestion** (예상 면접 질문) | report_id, question, why | 9-2 |
| **Payment** (건당 결제) | report_id, amount, paid_at, method | 11 |

**과거 문서에 있었으나 현재 코드에 없는 엔티티** — 설계에서 빼야 합니다:
`ReportDistribution`(지원자 히스토그램), `ReportTask`(주간 체크리스트), `Plan`(요금제 마스터). 앞의 둘은 모집단 DB가 없어 의도적으로 제외했고, `Plan`은 구독 모델을 건당 결제로 바꾸면서 삭제했습니다.

### 화면 간 정합성 주의

`reports.js`의 리포트 1건이 여러 화면에서 재사용됩니다 — 마이페이지 진단 기록 행 / 점수 추이 점 / 결제 내역 행 / 리포트 본문 / 기업 카드의 매치 점수. 현재 `rows`·`trend`는 `reports.js`와 별개로 손으로 적혀 있어(값만 일치), 실 데이터 전환 시 **`report_id` 하나로 묶어야** 화면 간 수치가 어긋나지 않습니다.
