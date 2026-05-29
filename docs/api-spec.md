# API Spec

## 공통 규칙

### 에러 응답 형식

모든 에러는 동일한 구조로 반환한다.

```json
{
  "status": 400,
  "errors": [
    {
      "field": "필드명",
      "message": "에러 메시지"
    }
  ]
}
```

### HTTP 상태 코드

| 코드 | 의미 | 사용 |
|------|------|------|
| 200 | OK | 조회, 수정, 상태 변경 성공 |
| 201 | Created | 리소스 생성 성공 |
| 204 | No Content | 삭제 성공 |
| 400 | Bad Request | 요청 검증 실패 |
| 404 | Not Found | 존재하지 않는 리소스 |
| 500 | Internal Server Error | 서버 내부 오류 |

### 에러 코드 정의

| 코드 | 의미 |
|------|------|
| VALIDATION_ERROR | 요청 데이터 검증 실패 |
| PROFILE_NOT_FOUND | 프로필을 찾을 수 없음 |
| INTERNAL_ERROR | 서버 내부 오류 |

---

## Profile

### POST /api/profile
프로필을 생성한다.

#### Request
```json
{
  "nickname": "string",
  "sex": "string"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| nickname | String | Y | 닉네임 |
| sex | String | Y | 성별. `M` 또는 `F` |

#### Response
| Status | 설명 |
|--------|------|
| 201 Created | 프로필 생성 성공. 생성된 프로필 id 반환 |

```json
{
  "id": "Ab2Cd3Ef"
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | String | 생성된 프로필 ID (8자리 랜덤 문자열) |

#### Validation Error — 400 Bad Request

| 필드 | 조건 | 에러 메시지 |
|------|------|------------|
| nickname | null 또는 빈 문자열 | 닉네임을 입력해 주세요. |
| sex | null 또는 빈 문자열 | 성별을 입력해 주세요. |
| sex | `M`, `F` 외의 값 | 성별은 M 또는 F만 입력 가능합니다. |

**Error Response Body**
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

---

### PATCH /api/profile/{profileId}/instructor
프로필을 강사로 지정한다. `isInstructor`를 `true`로 변경한다.

#### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| profileId | String | Y | 프로필 ID |

#### Request Body
없음

#### Response
| Status | 설명 |
|--------|------|
| 200 OK | 강사 지정 성공. 대상 프로필 id 반환 |

```json
{
  "id": "Ab2Cd3Ef"
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | String | 강사로 지정된 프로필 ID |

#### Error

| Status | 에러 코드 | 설명 |
|--------|-----------|------|
| 404 Not Found | `PROFILE_NOT_FOUND` | 존재하지 않는 profileId |
