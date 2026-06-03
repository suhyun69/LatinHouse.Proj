# Contract: POST /api/v1/order

**Date**: 2026-06-03

## Endpoint

`POST /api/v1/order`

인증 불필요 (`ApiSecurityConfig`에 `permitAll` 추가 필요)

---

## Request

**Content-Type**: `application/json`

```json
{
  "lessonNo": 1,
  "lessonOptionNo": 3,
  "profileId": "Ab2Cd3Ef"
}
```

| 필드 | 타입 | 필수 | 검증 | 에러 메시지 |
|------|------|------|------|------------|
| lessonNo | Long | Y | `@NotNull` | "레슨을 선택해 주세요." |
| lessonOptionNo | Long | Y | `@NotNull` | "수업 옵션을 선택해 주세요." |
| profileId | String | Y | `@NotBlank` | "구매자 프로필을 입력해 주세요." |

---

## Response

### 201 Created — 성공

```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 400 Bad Request — 검증 실패

```json
{
  "status": 400,
  "errors": [
    { "field": "lessonNo", "message": "레슨을 선택해 주세요." }
  ]
}
```

### 404 Not Found — 리소스 없음

```json
{
  "status": 404,
  "errors": [
    { "field": null, "message": "레슨을 찾을 수 없습니다" }
  ]
}
```

| 에러 코드 | 조건 |
|-----------|------|
| `LESSON_NOT_FOUND` | 존재하지 않는 lessonNo |
| `LESSON_OPTION_NOT_FOUND` | 존재하지 않는 lessonOptionNo |
| `PROFILE_NOT_FOUND` | 존재하지 않는 profileId |

---

## Test Scenarios

| # | 입력 | 기대 응답 |
|---|------|----------|
| 1 | 유효한 lessonNo + lessonOptionNo + profileId | 201 + UUID orderId |
| 2 | lessonNo = null | 400 + "레슨을 선택해 주세요." |
| 3 | lessonOptionNo = null | 400 + "수업 옵션을 선택해 주세요." |
| 4 | profileId = "" | 400 + "구매자 프로필을 입력해 주세요." |
| 5 | 존재하지 않는 lessonNo | 404 + LESSON_NOT_FOUND |
| 6 | 존재하지 않는 lessonOptionNo | 404 + LESSON_OPTION_NOT_FOUND |
| 7 | 존재하지 않는 profileId | 404 + PROFILE_NOT_FOUND |
