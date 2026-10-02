# 담당자 A — 공통 오류와 Mock 인증

먼저 [공통 규칙](./00-common.md)을 읽는다.

## 1. 담당 범위

```text
Branch: feat/mock-auth
Package: common, auth
DB 변경: 없음
```

담당 API:

| Method | Path | 성공 Status |
|---|---|---:|
| POST | `/api/v1/auth/register` | 201 |
| POST | `/api/v1/auth/login` | 200 |

추가 책임:

- 공통 Error Response
- 전역 Exception Handler
- 보호 API에서 사용할 Mock 사용자 해석
- 모든 기능 Merge 후 통합 Smoke Test

Spring Security, JWT Library, 비밀번호 암호화 Dependency는 추가하지 않는다.

## 2. 만들 파일

```text
common/auth/MockCurrentUserProvider.java
common/error/ErrorResponse.java
common/error/ApiException.java
common/error/GlobalExceptionHandler.java

auth/controller/AuthController.java
auth/service/MockAuthService.java
auth/dto/SignupRequest.java
auth/dto/UserResponse.java
auth/dto/LoginRequest.java
auth/dto/LoginResponse.java

backend/src/test/java/com/jobfeel/careercompass/common/auth/MockCurrentUserProviderTest.java
backend/src/test/java/com/jobfeel/careercompass/auth/controller/AuthControllerTest.java
```

DTO는 Java `record`를 사용해도 된다. Lombok·MapStruct는 추가하지 않는다.

## 3. 공통 오류 구현

### `ErrorResponse`

```java
String code
String message
```

### `ApiException`

다음 값을 생성자에서 받는다.

```java
HttpStatus status
String code
String message
```

### `GlobalExceptionHandler`

`@RestControllerAdvice`로 구현한다.

| Exception | Status | Code |
|---|---:|---|
| `ApiException` | Exception 내부 값 | Exception 내부 값 |
| `MethodArgumentNotValidException` | 400 | `VALIDATION_ERROR` |
| `HttpMessageNotReadableException` | 400 | `INVALID_REQUEST` |
| 예상하지 못한 Exception | 500 | `INTERNAL_SERVER_ERROR` |

예상하지 못한 오류는 서버 Log에 기록하되 Stack Trace를 Response에 넣지 않는다.

## 4. Mock 사용자 해석

### `MockCurrentUserProvider`

```java
long getCurrentUserId(String authorizationHeader)
```

처리 순서:

1. Header가 Null 또는 Blank인지 확인한다.
2. `Bearer ` Prefix를 확인한다.
3. Token이 `mock-access-token`인지 확인한다.
4. 일치하면 `1L`을 반환한다.
5. 불일치하면 `401 AUTHENTICATION_REQUIRED`를 발생시킨다.

B·C·D는 보호 API에서 아래 형태로 사용한다.

```java
@RequestHeader(value = "Authorization", required = false) String authorization
long userId = mockCurrentUserProvider.getCurrentUserId(authorization);
```

Filter·Interceptor·인증 Annotation은 만들지 않는다.

공용 구현이 끝나면 `feat/mock-auth`를 Push하고 B·C·D에게 알린다.

## 5. 회원가입 API

### Endpoint

```http
POST /api/v1/auth/register
Content-Type: application/json
```

### `SignupRequest`

| 필드 | Validation |
|---|---|
| `email` | `@NotBlank`, `@Email`, 최대 100자 |
| `password` | `@NotBlank`, 8~100자 |
| `name` | `@NotBlank`, 최대 50자 |

Request:

```json
{
  "email": "new-seeker@example.com",
  "password": "test1234!",
  "name": "홍길동"
}
```

### `MockAuthService.register()`

1. 입력 Password를 DB 또는 Log에 저장하지 않는다.
2. Mock 사용자 ID `2`를 사용한다.
3. Role은 `JOB_SEEKER`로 고정한다.
4. `createdAt`은 현재 UTC 시각으로 만든다.
5. Request의 Email과 Name을 Response에 사용한다.

Response — `201 Created`:

```json
{
  "userId": 2,
  "email": "new-seeker@example.com",
  "name": "홍길동",
  "role": "JOB_SEEKER",
  "createdAt": "2026-09-03T01:00:00Z"
}
```

회원가입은 Mock이므로 실제 `users` 테이블 Insert와 중복 Email 검사는 구현하지 않는다.

## 6. 로그인 API

### Endpoint

```http
POST /api/v1/auth/login
Content-Type: application/json
```

### `LoginRequest`

| 필드 | Validation |
|---|---|
| `email` | `@NotBlank`, `@Email` |
| `password` | `@NotBlank` |

Mock 로그인 값:

```text
email: seeker@example.com
password: test1234!
```

### `MockAuthService.login()`

1. Email과 Password가 Mock 값과 모두 같은지 확인한다.
2. 다르면 `401 INVALID_CREDENTIALS`를 발생시킨다.
3. 같으면 고정 Token과 Seed 사용자 정보를 반환한다.

Response — `200 OK`:

```json
{
  "accessToken": "mock-access-token",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "user": {
    "userId": 1,
    "name": "홍길동",
    "role": "JOB_SEEKER"
  }
}
```

## 7. 테스트

### `MockCurrentUserProviderTest`

- 정상 Header → `1L`
- Header 없음 → 401
- `Bearer ` Prefix 없음 → 401
- 다른 Token → 401

### `AuthControllerTest`

- 정상 Register → 201과 필수 응답 필드
- Email 형식 오류 → 400 `VALIDATION_ERROR`
- Password 8자 미만 → 400
- 정상 Login → 200과 `mock-access-token`
- 잘못된 Email 또는 Password → 401 `INVALID_CREDENTIALS`
- 잘못된 JSON → 400 `INVALID_REQUEST`

## 8. Commit 단위

```text
feat(be): 공통 에러 응답 구현
feat(be): Mock 사용자 해석 구현
feat(be): Mock 회원가입 로그인 API 구현
test(api): Mock 인증 API 테스트 추가
```

## 9. A 완료 조건

- [ ] 공통 오류가 `{code, message}`로 반환됨
- [ ] 유효한 Mock Token이 사용자 ID 1로 해석됨
- [ ] Register가 Password를 저장·기록하지 않음
- [ ] Login 성공·실패가 명세대로 동작함
- [ ] 테스트 통과
- [ ] `feat/mock-auth` Push 후 팀에 공유

