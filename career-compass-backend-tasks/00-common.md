# Career Compass Backend 공통 규칙

이 문서는 담당자 A·B·C·D 모두가 먼저 읽는 공통 규칙이다. 담당 기능의 세부 구현은 각자 문서를 따른다.

- [A — 공통 오류와 Mock 인증](./01-A-common-auth.md)
- [B — 사용자 프로필 API](./02-B-profile.md)
- [C — Mock Seed와 기업 API](./03-C-seed-company.md)
- [D — 분석 리포트 API](./04-D-report.md)

## 1. 구현 범위

구현한다.

- Spring MVC Controller
- Request/Response DTO와 Validation
- Service 로직
- Spring Data JPA 또는 필요한 최소 조회 코드
- PostgreSQL 조회·저장
- Flyway Mock Seed
- 트랜잭션과 오류 처리
- Controller/Service 테스트

Mock으로 유지한다.

- 인증 Token: `mock-access-token`
- OpenAI/LLM 응답
- Embedding 생성과 Vector 검색
- PDF 생성과 Storage URL
- Background Worker

구현하지 않는다.

- Frontend 변경
- 실제 JWT·Spring Security
- OpenAI SDK·Spring AI
- Redis·Kafka·Kubernetes·MSA·API Gateway
- 새 UI·테스트 Framework
- 필요 없는 Interface·Generic Wrapper·추상화

## 2. API 담당

| 담당 | Method | API | 기능 |
|---|---|---|---|
| A | POST | `/api/v1/auth/register` | Mock 회원가입 |
| A | POST | `/api/v1/auth/login` | Mock 로그인 |
| B | GET | `/api/v1/me/profile` | 내 프로필 조회 |
| B | PUT | `/api/v1/me/profile` | 내 프로필 전체 저장 |
| C | GET | `/api/v1/companies` | 기업 검색 |
| C | GET | `/api/v1/companies/{companyCode}` | 기업 상세 |
| D | POST | `/api/v1/reports` | Mock 리포트 생성 요청 |
| D | GET | `/api/v1/reports` | 내 리포트 목록 |
| D | GET | `/api/v1/reports/{reportId}` | 리포트 상태·결과 |
| D | GET | `/api/v1/reports/{reportId}/download` | Mock 다운로드 정보 |

## 3. Package와 파일 소유권

```text
backend/src/main/java/com/jobfeel/careercompass/
├── common/      # A만 수정
├── auth/        # A만 수정
├── user/        # B만 수정
├── company/     # C만 수정
└── analysis/    # D만 수정
```

| 담당 | 소유 테이블 |
|---|---|
| A | 없음 — 인증은 Mock |
| B | `users`, `job_seeker_profile`, `job_seeker_tech`, `profile_content` |
| C | `job_position`, `tech_stack`, `company`, `company_insight`, `company_insight_tech` |
| D | `analysis_report`, `analysis_report_company`, `analysis_report_item` |

공용 파일 규칙:

- `V1__create_initial_schema.sql`은 누구도 수정하지 않는다.
- Mock Seed 파일 `V2__insert_mock_data.sql`은 C만 편집한다.
- `build.gradle`, `application.yml`, `docker-compose.yml`, Dockerfile은 이번 API 구현에 변경할 이유가 없다.
- Dependency나 공용 설정 변경이 꼭 필요하면 먼저 팀에 알리고 한 사람만 수정한다.
- 다른 담당자의 Package를 직접 수정하지 않는다.

## 4. 공통 API 계약

- Prefix: `/api/v1`
- JSON 필드명: API 명세서와 동일하게 Camel Case
- 날짜·시간: UTC ISO 8601
- Entity를 Response로 직접 반환하지 않는다.
- Request DTO와 Response DTO를 분리한다.
- 사용자 ID를 Body, Query, Path에서 받지 않는다.

보호 API Header:

```http
Authorization: Bearer mock-access-token
```

Mock Token은 다음 사용자를 의미한다.

```text
userId: 1
role: JOB_SEEKER
```

공통 오류 응답:

```json
{
  "code": "ERROR_CODE",
  "message": "에러 설명"
}
```

| Status | 조건 |
|---:|---|
| 400 | JSON 형식, 필드 또는 Validation 오류 |
| 401 | Token 없음 또는 잘못된 Mock Token |
| 403 | 다른 사용자의 Report 접근 |
| 404 | Profile·Company·Report 없음 |
| 409 | Profile 미완성·중복 요청·Report 미완료 |
| 422 | FIT_ANALYSIS 점수를 계산할 대상 기술 없음 |

## 5. 공용 코드 받는 방법

각자 자신의 Branch에서 바로 개발한다. 다른 담당자를 기다리지 않는다.

A의 공통 오류·Mock 인증이 Push되거나 C의 Seed가 Push되면, 그 코드가 필요한 담당자는 자신의 Branch에서 가져온다.

```bash
git fetch origin
git merge origin/feat/mock-auth
git merge origin/feat/mock-seed-data
```

이미 자기 구현을 끝냈더라도 PR 전에는 공용 Branch 또는 최신 `main`을 반영하고 다시 테스트한다.

```bash
git fetch origin
git merge origin/main
```

충돌 규칙:

- `common`, `auth` 충돌은 A와 해결한다.
- Migration 충돌은 C와 해결한다.
- 각 기능 Package 충돌은 해당 파일 소유자와 해결한다.
- 충돌 해결을 위해 다른 담당자의 코드를 삭제하지 않는다.
- Force Push를 사용하지 않는다.

## 6. Branch와 Merge

사용 Branch:

```text
A: feat/mock-auth
B: feat/user-profile
C Seed: feat/mock-seed-data
C Company: feat/company-search
D: feat/report-api
통합: test/api-integration
```

C는 Seed를 별도 Branch에 Commit·Push한 다음, 그 Branch에서 `feat/company-search`를 생성한다.

공용 PR인 `feat/mock-auth`, `feat/mock-seed-data`는 이를 사용하는 기능 PR보다 먼저 Main에 Merge한다. 이후 기능 PR은 구현과 Review가 끝난 것부터 Merge한다.

다른 PR이 먼저 Merge되면 남은 담당자는 최신 Main을 자기 Branch에 반영하고 다시 테스트한다.

PR 규칙:

- Main 직접 Push 금지
- 최소 1명 Review
- PR 제목: `type(scope): 내용`
- Squash and Merge
- Merge 후 Branch 삭제

Commit 예시:

```text
feat(be): 프로필 조회 API 구현
feat(db): Mock Seed 데이터 추가
test(api): 기업 API 테스트 추가
fix(be): 리포트 소유권 검증 수정
```

## 7. 공통 테스트 규칙

각 담당자는 다음을 확인한다.

- 정상 요청
- Request Validation 400
- Token 없음 401
- Resource 없음 404
- 기능별 409 또는 422
- JSON 필드명과 Null·생략 규칙
- Log에 Password·API Key가 남지 않음

실행:

```bash
cd backend
./gradlew test
```

현재 CI는 PostgreSQL Service를 띄우지 않으므로 다음처럼 구분한다.

- Controller: `@WebMvcTest`와 Mock Service
- Service: JUnit·Mockito 단위 테스트
- 실제 DB·Flyway·JPA: Docker Compose Smoke Test

## 8. 최종 통합 확인

모든 기능이 Merge되면 A가 `test/api-integration` Branch에서 확인한다.

```bash
docker compose down -v
docker compose up --build
curl http://localhost:8080/actuator/health
cd backend
./gradlew test
```

`docker compose down -v`는 기존 개발 DB 데이터를 삭제하므로 최종 통합 검증에서만 사용한다.

Smoke Test 순서:

1. Login으로 Mock Token 확인
2. Profile GET
3. Profile PUT 후 다시 GET
4. Company List·Filter·Detail
5. RECOMMEND POST 후 Report 501 조회
6. FIT_ANALYSIS POST 후 Report 502 조회
7. Report 503 PROCESSING 조회
8. Report 504 FAILED 조회
9. Report 목록 Filter·Paging
10. Report 502 Download
11. Token 없이 보호 API 호출 시 401

## 9. 공통 완료 조건

- [ ] 담당 Package 외 파일을 임의로 수정하지 않음
- [ ] API Request·Response가 명세와 일치
- [ ] 공통 오류가 `{code, message}` 형태
- [ ] 담당 테스트 통과
- [ ] Docker 환경에서 담당 API 호출 성공
- [ ] 최신 Main 반영 후 재검증
- [ ] Frontend 변경 없음
- [ ] 실제 JWT·OpenAI·PDF·Vector 검색 구현 없음
- [ ] V1 Migration 변경 없음
- [ ] Secret Commit 없음

