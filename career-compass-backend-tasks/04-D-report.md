# 담당자 D — 분석 리포트 API

먼저 [공통 규칙](./00-common.md)을 읽는다.

## 1. 담당 범위

```text
Branch: feat/report-api
Package: analysis
Tables: analysis_report, analysis_report_company, analysis_report_item
```

담당 API:

| Method | Path | 성공 Status |
|---|---|---:|
| POST | `/api/v1/reports` | 202 |
| GET | `/api/v1/reports` | 200 |
| GET | `/api/v1/reports/{reportId}` | 200 |
| GET | `/api/v1/reports/{reportId}/download` | 200 |

실제 OpenAI·Embedding·Vector 검색·PDF·Background Job은 구현하지 않는다.

의존하는 공용 작업:

- A의 `MockCurrentUserProvider`
- C의 V2 Mock Seed

두 작업을 기다리지 말고 `analysis` Package의 Entity·DTO·Service부터 작성한다. Report Seed 내용은 작성하는 즉시 C에게 전달한다.

## 2. 만들 파일

```text
analysis/controller/ReportController.java
analysis/service/MockReportService.java

analysis/domain/AnalysisReport.java
analysis/domain/AnalysisReportCompany.java
analysis/domain/AnalysisReportItem.java
analysis/domain/ReportType.java
analysis/domain/ReportStatus.java
analysis/domain/ReportItemType.java

analysis/repository/AnalysisReportRepository.java
analysis/repository/AnalysisReportCompanyRepository.java
analysis/repository/AnalysisReportItemRepository.java
analysis/repository/AnalysisSourceQueryRepository.java

analysis/dto/ReportCreateRequest.java
analysis/dto/ReportAcceptedResponse.java
analysis/dto/ReportListItemResponse.java
analysis/dto/ReportListResponse.java
analysis/dto/ReportResponse.java
analysis/dto/ReportErrorResponse.java
analysis/dto/ReportDownloadResponse.java

backend/src/test/java/com/jobfeel/careercompass/analysis/service/MockReportServiceTest.java
backend/src/test/java/com/jobfeel/careercompass/analysis/controller/ReportControllerTest.java
```

결과 내부의 작은 DTO·record는 `analysis/dto/result/`에 둔다. Service Interface와 불필요한 Generic 구조는 만들지 않는다.

## 3. Enum

### `ReportType`

```text
RECOMMEND
FIT_ANALYSIS
```

### `ReportStatus`

```text
PENDING
PROCESSING
COMPLETED
FAILED
```

### `ReportItemType`

```text
REASON
STRENGTHS
GAPS
RECOMMENDED_ACTIONS
PREPARATION_DIRECTION
SCORE_DETAIL
FIT_REASONS
RECOMMENDED_LEARNING
RECOMMENDED_PROJECTS
RESUME_HIGHLIGHTS
```

## 4. Entity Mapping

### `AnalysisReport`

```text
@Table(name = "analysis_report")
reportId: Long, @Id, @GeneratedValue
userId: Long
reportType: ReportType, EnumType.STRING
status: ReportStatus, EnumType.STRING
errorCode: String, nullable
pdfObjectKey: String, nullable
createdAt: OffsetDateTime
completedAt: OffsetDateTime, nullable
```

### `AnalysisReportCompany`

```text
@Table(name = "analysis_report_company")
reportCompanyId: Long, @Id, @GeneratedValue
reportId: Long
companyCode: String
positionId: Long
rank: Integer, nullable
fitScore: BigDecimal, nullable
```

### `AnalysisReportItem`

```text
@Table(name = "analysis_report_item")
itemId: Long, @Id, @GeneratedValue
reportCompanyId: Long
itemType: ReportItemType, EnumType.STRING
content: String
```

복잡한 JPA Association 대신 FK ID를 직접 보관하고 Service에서 조립한다.

## 5. Repository

### `AnalysisReportRepository`

```java
Optional<AnalysisReport> findById(Long reportId)
Optional<AnalysisReport> findByReportIdAndUserId(Long reportId, Long userId)
Page<AnalysisReport> searchByUserIdAndOptionalFilters(...)
```

### `AnalysisReportCompanyRepository`

```java
List<AnalysisReportCompany> findAllByReportIdOrderByRankAsc(Long reportId)
```

### `AnalysisReportItemRepository`

```java
List<AnalysisReportItem> findAllByReportCompanyIdIn(Collection<Long> ids)
```

### `AnalysisSourceQueryRepository`

다른 Package의 Entity를 복사하지 않는다. `JdbcTemplate` 또는 필요한 최소 Query로 다음 사실만 조회한다.

```java
Long findDesiredPositionId(long userId)
boolean existsCompany(String companyCode)
boolean existsTargetTech(String companyCode, long positionId)
String findCompanyName(String companyCode)
String findPositionName(long positionId)
```

## 6. 리포트 생성 요청

### Endpoint

```http
POST /api/v1/reports
Authorization: Bearer mock-access-token
Content-Type: application/json
```

RECOMMEND Request:

```json
{
  "reportType": "RECOMMEND"
}
```

FIT_ANALYSIS Request:

```json
{
  "reportType": "FIT_ANALYSIS",
  "companyCode": "C001"
}
```

Validation:

- `reportType`은 `RECOMMEND` 또는 `FIT_ANALYSIS`만 허용한다.
- RECOMMEND에 `companyCode`가 있으면 400이다.
- FIT_ANALYSIS에는 `companyCode`가 필수다.
- `positionId`는 Request로 받지 않고 Profile에서 조회한다.

### `MockReportService` 처리

1. Token에서 `userId=1`을 얻는다.
2. Profile과 `desired_position_id`를 확인한다.
3. 없으면 `409 PROFILE_INCOMPLETE`를 발생시킨다.
4. FIT_ANALYSIS는 Company 존재를 확인한다.
5. Company가 없으면 `404 COMPANY_NOT_FOUND`를 발생시킨다.
6. Company·Position 대상 기술이 없으면 `422 SCORE_NOT_CALCULABLE`을 발생시킨다.
7. 실제 Report Job이나 OpenAI 요청을 생성하지 않는다.
8. RECOMMEND는 Seed Report `501`, FIT_ANALYSIS는 `502`를 사용한다.
9. `Location: /api/v1/reports/{reportId}` Header를 추가한다.

Response — `202 Accepted`:

```json
{
  "reportId": 501,
  "reportType": "RECOMMEND",
  "status": "PENDING",
  "createdAt": "2026-09-03T02:00:00Z"
}
```

POST 응답은 요청 접수 형태의 PENDING이고, 이어지는 GET은 미리 저장한 Mock 결과를 반환한다.

## 7. 리포트 상태·결과 조회

### Endpoint

```http
GET /api/v1/reports/{reportId}
Authorization: Bearer mock-access-token
```

처리:

1. Token에서 User ID를 얻는다.
2. `findById(reportId)`로 Report 존재를 확인한다.
3. 없으면 `404 REPORT_NOT_FOUND`를 발생시킨다.
4. `report.userId != currentUserId`이면 `403 REPORT_ACCESS_DENIED`를 발생시킨다.
5. Status에 맞게 Response를 조립한다.

처음부터 User ID까지 조건으로 조회하면 404와 403을 구분할 수 없으므로 Detail·Download에서는 사용하지 않는다.

Status별 응답:

| Status | result | error | pdfReady |
|---|---|---|---|
| PENDING | `null` | `null` | `false` |
| PROCESSING | `null` | `null` | `false` |
| FAILED | `null` | `{code,message}` | `false` |
| COMPLETED | 조립된 결과 | `null` | `true` |

Seed 상태:

```text
501 → RECOMMEND COMPLETED
502 → FIT_ANALYSIS COMPLETED
503 → RECOMMEND PROCESSING
504 → FIT_ANALYSIS FAILED
```

### RECOMMEND 결과 조립

- `rank` 오름차순으로 최대 5개를 반환한다.
- 최상위 Response의 `companyCode`, `fitScore`는 생략한다.
- 추천 기업마다 아래 필드를 조립한다.

```text
rank
companyCode
companyName
positionId
positionName
reason
strengths
gaps
recommendedActions
preparationDirection
```

후보가 없으면 `COMPLETED`와 `recommendations=[]`를 반환한다.

### FIT_ANALYSIS 결과 조립

최상위 Response에 `companyCode`, `fitScore`를 포함한다. 결과에는 아래 필드를 조립한다.

```text
companyName
positionId
positionName
scoreDetail
fitReasons
strengths
gaps
recommendedLearning
recommendedProjects
preparationDirection
resumeHighlights
```

FIT_ANALYSIS 결과에는 `rank`를 넣지 않는다.

`analysis_report_item.content`는 JSON 문자열이다. Spring에 포함된 Jackson `ObjectMapper`로 Type별 구조에 맞게 읽는다. 새 JSON Library를 추가하지 않는다.

### FAILED Error 매핑

```text
EMBEDDING_GENERATION_FAILED → 입력 데이터 임베딩 생성에 실패했습니다.
VECTOR_SEARCH_FAILED        → 기업 인사이트 검색에 실패했습니다.
LLM_GENERATION_FAILED       → 분석 결과 생성에 실패했습니다.
INVALID_LLM_RESPONSE        → 분석 결과 형식이 올바르지 않습니다.
PDF_GENERATION_FAILED       → PDF 생성에 실패했습니다.
ANALYSIS_INTERNAL_ERROR     → 분석 처리 중 오류가 발생했습니다.
```

## 8. 내 리포트 목록

### Endpoint

```http
GET /api/v1/reports?reportType=FIT_ANALYSIS&status=COMPLETED&page=0&size=20
Authorization: Bearer mock-access-token
```

Query Parameter:

| 이름 | 규칙 |
|---|---|
| `reportType` | 선택 |
| `status` | 선택 |
| `page` | 기본 0, 0 이상 |
| `size` | 기본 20, 1~100 |

처리:

1. 반드시 현재 `user_id`로 필터한다.
2. Report Type과 Status가 있으면 조건에 추가한다.
3. `created_at DESC`로 정렬한다.
4. `items/page/size/totalElements/totalPages`를 반환한다.
5. RECOMMEND Item은 `companyCode`, `companyName`, `fitScore`를 생략한다.
6. FIT_ANALYSIS Item만 세 필드를 포함한다.

필드를 Null로 보내지 않고 생략하도록 `ReportListItemResponse`에 DTO 단위 `@JsonInclude(JsonInclude.Include.NON_NULL)`을 적용하거나 Type별 DTO를 분리한다. 전역 Jackson 설정은 수정하지 않는다.

## 9. 다운로드 정보

### Endpoint

```http
GET /api/v1/reports/{reportId}/download
Authorization: Bearer mock-access-token
```

처리:

1. Report 존재 여부를 확인한다.
2. 현재 사용자의 Report인지 확인한다.
3. `status != COMPLETED`이면 `409 REPORT_NOT_READY`를 발생시킨다.
4. 실제 PDF 또는 Storage를 호출하지 않는다.
5. 고정 형식의 Mock URL을 반환한다.

Response:

```json
{
  "fileName": "report-502.pdf",
  "downloadUrl": "https://example.invalid/mock/report-502.pdf",
  "expiresAt": "2026-09-03T02:15:00Z"
}
```

- `fileName`: `report-{reportId}.pdf`
- `downloadUrl`: `https://example.invalid/mock/report-{reportId}.pdf`
- `expiresAt`: 현재 UTC + 5분

## 10. C에게 전달할 Seed

D는 V2 Migration을 직접 수정하지 않고 C에게 다음 내용을 전달한다.

```text
Report 501: RECOMMEND, COMPLETED
Report 502: FIT_ANALYSIS, COMPLETED
Report 503: RECOMMEND, PROCESSING
Report 504: FIT_ANALYSIS, FAILED

Report Company 601: report 501, C001, position 10, rank 1
Report Company 602: report 501, C002, position 10, rank 2
Report Company 603: report 502, C001, position 10, fitScore 75.00

Report Item: 601~603 결과를 만들 REASON/STRENGTHS/GAPS 등 JSON
```

## 11. 테스트

### `MockReportServiceTest`

- RECOMMEND POST → Report 501
- FIT_ANALYSIS POST → Report 502
- FIT Company 없음 → 404
- Profile 미완성 → 409
- 대상 기술 없음 → 422
- 501 Recommendation 조립과 Rank 순서
- 502 FIT_ANALYSIS 조립과 `fitScore=75.00`
- 503 PROCESSING Null 규칙
- 504 FAILED Error 규칙
- 목록 사용자 필터·조건 필터·Paging
- Download 정상·미완료

### `ReportControllerTest`

- Token 없음 → 401
- POST Validation 오류 → 400
- POST 정상 → 202와 Location Header
- Detail 정상 → 200
- Report 없음 → 404
- 다른 사용자 Report → 403
- List 정상 → 200과 Paging 필드
- Download 정상 → 200
- 미완료 Download → 409

## 12. Commit 단위

```text
feat(be): Analysis Report Entity Repository 구현
feat(be): Mock Report 생성 상태 API 구현
feat(be): Report 목록 다운로드 API 구현
test(api): Analysis Report API 테스트 추가
```

## 13. D 완료 조건

- [ ] POST가 501 또는 502와 Location을 반환함
- [ ] 501·502·503·504가 Status별 계약에 맞게 응답함
- [ ] Report 404와 소유권 403을 구분함
- [ ] 목록 Filter·Paging·Null 생략 규칙 준수
- [ ] Download는 실제 PDF를 생성하지 않음
- [ ] 실제 OpenAI·Embedding·Vector 검색 없음
- [ ] C에게 Report Seed 전달
- [ ] A Auth와 C Seed 반영 후 테스트 통과

