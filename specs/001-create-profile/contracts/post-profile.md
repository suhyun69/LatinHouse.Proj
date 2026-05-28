# Contract: POST /api/profile

**Source**: [docs/api-spec.md](../../../docs/api-spec.md)

## Endpoint

`POST /api/profile`

## Request

**Content-Type**: `application/json`

```json
{
  "nickname": "string",
  "sex": "string"
}
```

| 필드 | 타입 | 필수 | 제약 |
|------|------|------|------|
| nickname | String | Y | Not null, not blank |
| sex | String | Y | `M` 또는 `F` |

## Response

### 201 Created

```json
{
  "id": "Ab2Cd3Ef"
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | String | 생성된 프로필 ID (8자리 랜덤 문자열) |

### 400 Bad Request

```json
{
  "status": 400,
  "errors": [
    {
      "field": "nickname",
      "message": "닉네임을 입력해 주세요."
    }
  ]
}
```

| 필드 | 조건 | 에러 메시지 |
|------|------|-------------|
| nickname | null 또는 빈 문자열 | "닉네임을 입력해 주세요." |
| sex | null 또는 빈 문자열 | "성별을 입력해 주세요." |
| sex | `M`, `F` 외의 값 | "성별은 M 또는 F만 입력 가능합니다." |

**다중 에러**: 여러 필드가 동시에 실패하면 `errors` 배열에 모든 에러를 포함하여 반환한다.

## Implementation Mapping

| Contract 요소 | 구현 클래스 |
|---------------|-------------|
| 요청 수신 | `ProfileController` |
| 요청 DTO | `CreateProfileWebRequest` |
| 입력 검증 | `@Valid` + `@NotBlank`, `@Pattern` |
| 에러 응답 | `GlobalExceptionHandler` + `ErrorResponse` |
| 성공 응답 DTO | `CreateProfileWebResponse` |
