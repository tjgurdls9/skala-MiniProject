# 담당자 B — 사용자 프로필 API

먼저 [공통 규칙](./00-common.md)을 읽는다.

## 1. 담당 범위

```text
Branch: feat/user-profile
Package: user
Tables: users, job_seeker_profile, job_seeker_tech, profile_content
```

담당 API:

| Method | Path | 성공 Status |
|---|---|---:|
| GET | `/api/v1/me/profile` | 200 |
| PUT | `/api/v1/me/profile` | 200 |

의존하는 공용 작업:

- A의 `MockCurrentUserProvider`
- C의 `V2__insert_mock_data.sql`

A 또는 C가 Branch를 Push하면 자신의 Branch에 Merge한다. 두 작업을 기다리지 말고 `user` Package 구현부터 시작한다.

## 2. 만들 파일

```text
user/controller/ProfileController.java
user/service/ProfileService.java

user/domain/User.java
user/domain/JobSeekerProfile.java
user/domain/JobSeekerTech.java
user/domain/JobSeekerTechId.java
user/domain/ProfileContent.java

user/repository/UserRepository.java
user/repository/ProfileRepository.java
user/repository/JobSeekerTechRepository.java
user/repository/ProfileContentRepository.java
user/repository/ProfileQueryRepository.java

user/dto/ProfileUpsertRequest.java
user/dto/ProfileContentRequest.java
user/dto/ProfileResponse.java
user/dto/PositionResponse.java
user/dto/TechStackResponse.java
user/dto/ProfileContentResponse.java

backend/src/test/java/com/jobfeel/careercompass/user/service/ProfileServiceTest.java
backend/src/test/java/com/jobfeel/careercompass/user/controller/ProfileControllerTest.java
```

## 3. Entity Mapping

### `User`

```text
@Table(name = "users")
userId: Long, @Id
email: String
passwordHash: String
name: String
role: String
createdAt: OffsetDateTime
```

### `JobSeekerProfile`

```text
@Table(name = "job_seeker_profile")
userId: Long, @Id
desiredPositionId: Long, nullable
```

### `JobSeekerTechId`

`user_id + tech_id` 복합 PK를 표현하는 `@Embeddable`이다. `equals()`와 `hashCode()`를 구현한다.

### `JobSeekerTech`

```text
@Table(name = "job_seeker_tech")
id: JobSeekerTechId, @EmbeddedId
```

### `ProfileContent`

```text
@Table(name = "profile_content")
contentId: Long, @Id, @GeneratedValue
userId: Long
category: String 또는 Enum
content: String
createdAt: OffsetDateTime
```

Mock에서 사용하지 않는 `content_hash`, `embedding`은 Entity에서 생략한다. Vector Java Dependency는 추가하지 않는다.

## 4. Repository

### `UserRepository`

```java
boolean existsById(Long userId)
```

### `ProfileRepository`

```java
Optional<JobSeekerProfile> findByUserId(Long userId)
```

### `JobSeekerTechRepository`

```java
List<JobSeekerTech> findAllByIdUserId(Long userId)
void deleteAllByIdUserId(Long userId)
```

### `ProfileContentRepository`

```java
List<ProfileContent> findAllByUserIdOrderByContentId(Long userId)
void deleteAllByUserId(Long userId)
```

### `ProfileQueryRepository`

`job_position`, `tech_stack`을 `user` Package Entity로 중복 작성하지 않는다. `JdbcTemplate` 또는 Native Projection으로 필요한 값만 조회한다.

```java
boolean existsPosition(long positionId)
Map<Long, String> findTechNames(Collection<Long> techIds)
String findPositionName(long positionId)
```

## 5. 프로필 조회

### Endpoint

```http
GET /api/v1/me/profile
Authorization: Bearer mock-access-token
```

### Service 처리

1. A의 Provider로 `userId=1`을 얻는다.
2. `job_seeker_profile`을 조회한다.
3. 없으면 `404 PROFILE_NOT_FOUND`를 발생시킨다.
4. `job_position`에서 직무 이름을 조회한다.
5. 사용자의 Tech ID를 조회하고 `tech_stack`에서 이름을 가져온다.
6. `profile_content`를 `content_id` 오름차순으로 조회한다.
7. Entity가 아닌 `ProfileResponse`로 조립한다.

Response:

```json
{
  "userId": 1,
  "desiredPosition": {
    "positionId": 10,
    "positionName": "백엔드 개발자"
  },
  "techStacks": [
    {"techId": 101, "techName": "Java"},
    {"techId": 102, "techName": "Spring"}
  ],
  "contents": [
    {
      "contentId": 1001,
      "category": "PROJECT",
      "content": "Spring Boot로 REST API를 구현했습니다.",
      "createdAt": "2026-09-03T01:10:00Z"
    }
  ]
}
```

`contentHash`, `embedding`, `passwordHash`는 응답에 포함하지 않는다.

## 6. 프로필 전체 저장

### Endpoint

```http
PUT /api/v1/me/profile
Authorization: Bearer mock-access-token
Content-Type: application/json
```

Request:

```json
{
  "desiredPositionId": 10,
  "techIds": [101, 102, 103],
  "contents": [
    {
      "category": "PROJECT",
      "content": "Spring Boot로 REST API를 구현했습니다."
    },
    {
      "category": "PREFERRED_CULTURE",
      "content": "협업과 학습을 중시합니다."
    }
  ]
}
```

Validation:

| 필드 | 규칙 |
|---|---|
| `desiredPositionId` | `@NotNull` |
| `techIds` | `@NotNull`, Null 원소와 중복 금지 |
| `contents` | `@NotNull` |
| `contents[].category` | `@NotBlank`, 허용 Category만 |
| `contents[].content` | `@NotBlank` |

허용 Category:

```text
PROJECT
EXPERIENCE
STUDY
PREFERRED_CULTURE
```

### `ProfileService` 처리

전체 작업을 하나의 `@Transactional` 메서드에서 처리한다.

1. Token에서 User ID를 얻는다.
2. `users`에 User가 있는지 확인한다.
3. Position ID가 존재하는지 확인한다.
4. Tech ID 중복을 확인한다.
5. 요청한 모든 Tech ID가 존재하는지 확인한다.
6. Category 값을 확인한다.
7. `job_seeker_profile`을 Insert 또는 Update한다.
8. 기존 `job_seeker_tech`를 모두 삭제하고 요청 값을 저장한다.
9. 기존 `profile_content`를 모두 삭제하고 요청 값을 저장한다.
10. 저장 결과를 다시 조회해 GET과 동일한 Response를 반환한다.

같은 Category의 Content는 여러 개 저장할 수 있다. Request에서 `contentId`, `userId`, `contentHash`, `embedding`을 받지 않는다.

오류:

| 조건 | Status | Code |
|---|---:|---|
| User 없음 | 404 | `USER_NOT_FOUND` |
| Position 없음 | 400 | `INVALID_POSITION` |
| Tech ID 없음 | 400 | `INVALID_TECH_STACK` |
| Tech ID 중복 | 400 | `DUPLICATE_TECH_STACK` |
| Category 오류 | 400 | `INVALID_PROFILE_CATEGORY` |

## 7. C에게 전달할 Seed

C가 V2 Migration을 편집하므로 B는 SQL 파일을 직접 수정하지 않는다. 다음 데이터가 필요하다고 전달한다.

```text
job_seeker_profile: user_id=1, desired_position_id=10
job_seeker_tech: (1,101), (1,102), (1,103)
profile_content: content_id 1001~1004
categories: PROJECT, EXPERIENCE, STUDY, PREFERRED_CULTURE
```

## 8. 테스트

### `ProfileServiceTest`

- GET 정상 Response 조립
- Profile 없음 → `PROFILE_NOT_FOUND`
- PUT 정상 → 기존 Tech와 Content 전체 교체
- Position 없음 → 400
- Tech 하나라도 없음 → 400
- Tech ID 중복 → 400
- Category 오류 → 400
- 저장 중 예외 → 전체 작업 Rollback 대상

### `ProfileControllerTest`

- Token 없음 → 401
- GET 정상 → 200과 필수 응답 필드
- PUT 정상 → 200
- PUT 필수값 누락 → 400
- PUT 잘못된 Category → 400

## 9. Commit 단위

```text
feat(be): Profile Entity Repository 구현
feat(be): Profile 조회 API 구현
feat(be): Profile 전체 저장 API 구현
test(api): Profile API 테스트 추가
```

## 10. B 완료 조건

- [ ] GET이 Seed Profile을 반환함
- [ ] PUT이 Profile·Tech·Content를 한 Transaction으로 교체함
- [ ] 다른 Package Entity를 중복 생성하지 않음
- [ ] Vector Dependency를 추가하지 않음
- [ ] A Auth와 C Seed를 반영한 상태에서 테스트 통과

