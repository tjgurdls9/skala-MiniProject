# 담당자 C — Mock Seed와 기업 API

먼저 [공통 규칙](./00-common.md)을 읽는다.

## 1. 담당 범위

```text
Seed Branch: feat/mock-seed-data
Company Branch: feat/company-search
Package: company
Migration: V2__insert_mock_data.sql
Tables: job_position, tech_stack, company, company_insight, company_insight_tech
```

담당 API:

| Method | Path | 성공 Status |
|---|---|---:|
| GET | `/api/v1/companies` | 200 |
| GET | `/api/v1/companies/{companyCode}` | 200 |

추가 책임:

- 모든 기능이 같이 사용할 Mock Seed 작성
- V1 → V2 Fresh Migration 검증
- 고정 ID 사용 후 Sequence 보정

## 2. Branch 사용

Seed는 다른 담당자가 필요하므로 Company 기능과 Commit을 분리한다.

1. `main`에서 `feat/mock-seed-data`를 만든다.
2. V2 Migration만 Commit·Push한다.
3. 해당 Branch에서 `feat/company-search`를 만든다.
4. Company 코드와 테스트는 `feat/company-search`에 Commit한다.
5. Seed Branch를 Push하면 B·D에게 알린다.

## 3. V2 Mock Seed

파일:

```text
backend/src/main/resources/db/migration/V2__insert_mock_data.sql
```

`V1__create_initial_schema.sql`은 수정하지 않는다.

### Master와 사용자

```text
users
- user_id=1
- email=seeker@example.com
- name=홍길동
- role=JOB_SEEKER
- password_hash=Mock에서 인증에 사용하지 않는 값

job_position
- 10=백엔드 개발자

tech_stack
- 101=Java
- 102=Spring
- 103=PostgreSQL
- 104=Docker
```

`password_hash`에 실제 비밀번호나 API Key를 넣지 않는다. A의 Mock Login은 DB Password를 읽지 않는다.

### Profile — B 요구 데이터

```text
job_seeker_profile
- user_id=1, desired_position_id=10

job_seeker_tech
- (1,101), (1,102), (1,103)

profile_content
- 1001 PROJECT
- 1002 EXPERIENCE
- 1003 STUDY
- 1004 PREFERRED_CULTURE
```

### Company

```text
company
- C001, 예시테크, IT
- C002, 클라우드나인, IT

company_insight
- C001/C002의 기술·문화·채용 관련 Text
- position_id=10

company_insight_tech
- C001 Insight에 Tech 101,102,103,104 연결
```

Embedding은 Null로 둔다. 1536차원 가짜 Vector를 만들지 않는다.

### Report — D 요구 데이터

```text
analysis_report
- 501 RECOMMEND COMPLETED
- 502 FIT_ANALYSIS COMPLETED
- 503 RECOMMEND PROCESSING
- 504 FIT_ANALYSIS FAILED

analysis_report_company
- 601=(501,C001,position 10,rank 1)
- 602=(501,C002,position 10,rank 2)
- 603=(502,C001,position 10,fit_score 75.00)

analysis_report_item
- report_company_id 601~603에 REASON/STRENGTHS/GAPS 등 추가
- FIT_ANALYSIS에 SCORE_DETAIL 추가
```

D가 전달한 Report Item JSON을 그대로 반영하되 JSON 문법을 확인한다.

### Insert와 Sequence 규칙

FK 순서:

```text
Master/User
→ Profile/Insight
→ Analysis Report
→ Analysis Report Company
→ Analysis Report Item
```

고정 `BIGSERIAL` ID를 넣은 뒤 Sequence를 MAX ID로 맞춘다.

```sql
SELECT setval(
    pg_get_serial_sequence('users', 'user_id'),
    (SELECT MAX(user_id) FROM users),
    true
);
```

같은 처리가 필요한 Sequence:

```text
users.user_id
tech_stack.tech_id
job_position.position_id
profile_content.content_id
company_insight.insight_id
analysis_report.report_id
analysis_report_company.report_company_id
analysis_report_item.item_id
```

V2가 Main에 Merge된 뒤에는 V2를 수정하지 않는다. 추가 변경은 새 Migration Version을 만든다.

## 4. Company 기능 파일

```text
company/controller/CompanyController.java
company/service/CompanyService.java
company/repository/CompanyRepository.java
company/domain/Company.java
company/dto/CompanyResponse.java
company/dto/CompanySearchResponse.java

backend/src/test/java/com/jobfeel/careercompass/company/service/CompanyServiceTest.java
backend/src/test/java/com/jobfeel/careercompass/company/controller/CompanyControllerTest.java
```

Company 검색·상세에 필요하지 않은 Insight·Tech Entity는 만들지 않는다.

## 5. `Company` Entity와 Repository

Entity:

```text
@Table(name = "company")
companyCode: String, @Id
companyName: String
industry: String, nullable
```

Repository 검색 조건:

```text
query 없음    → 기업명 조건 미적용
industry 없음 → 산업 조건 미적용
query 있음    → lower(companyName) contains lower(query)
industry 있음 → industry exact match
```

하나의 JPQL Query와 `Pageable`로 처리한다.

```java
Page<Company> search(String query, String industry, Pageable pageable)
Optional<Company> findById(String companyCode)
```

## 6. 기업 검색 API

### Endpoint

```http
GET /api/v1/companies?query=예시&industry=IT&page=0&size=20
Authorization: Bearer mock-access-token
```

Query Parameter:

| 이름 | 규칙 |
|---|---|
| `query` | 선택, Blank는 미입력 처리 |
| `industry` | 선택, Blank는 미입력 처리 |
| `page` | 기본 0, 0 이상 |
| `size` | 기본 20, 1~100 |

처리:

1. A의 Provider로 Token을 검증한다.
2. Query와 Industry의 Blank를 Null로 바꾼다.
3. `PageRequest.of(page, size)`를 생성한다.
4. Repository로 검색한다.
5. DTO로 변환한다.

Response:

```json
{
  "items": [
    {
      "companyCode": "C001",
      "companyName": "예시테크",
      "industry": "IT"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

## 7. 기업 상세 API

### Endpoint

```http
GET /api/v1/companies/C001
Authorization: Bearer mock-access-token
```

처리:

1. Token을 검증한다.
2. `companyCode`로 조회한다.
3. 없으면 `404 COMPANY_NOT_FOUND`를 발생시킨다.
4. `companyCode`, `companyName`, `industry`를 반환한다.

DB와 명세에 없는 기업 규모·설명·문화 태그는 추가하지 않는다.

## 8. 테스트

### Seed 검증

- 새 Volume에서 V1과 V2 적용 성공
- 각 Mock 테이블의 행 수 확인
- FK 오류 없음
- Fixed ID 이후 자동 ID Insert 성공
- Vector 컬럼은 Null

### `CompanyServiceTest`

- 조건 없는 전체 검색
- Query 필터
- Industry 필터
- Query와 Industry 동시 필터
- 빈 결과
- Detail 정상
- Detail 없음 → `COMPANY_NOT_FOUND`

### `CompanyControllerTest`

- Token 없음 → 401
- List 정상 → 200과 Paging 필드
- `page=-1` → 400
- `size=0`, `size=101` → 400
- Detail 정상 → 200
- 없는 Company → 404

## 9. Commit 단위

Seed Branch:

```text
feat(db): 공통 Mock Master 데이터 추가
feat(db): Profile Report Mock 데이터 추가
test(db): Flyway Mock Seed 적용 확인
```

Company Branch:

```text
feat(be): Company Entity Repository 구현
feat(be): 기업 검색 상세 API 구현
test(api): 기업 API 테스트 추가
```

## 10. C 완료 조건

- [ ] V2가 새 DB에 정상 적용됨
- [ ] B와 D가 요청한 Seed 포함
- [ ] 고정 ID 뒤 Sequence 보정 완료
- [ ] 기업 검색·상세와 Paging 동작
- [ ] 가짜 Vector 데이터 없음
- [ ] V1 수정 없음
- [ ] Seed와 Company Commit·PR 분리

