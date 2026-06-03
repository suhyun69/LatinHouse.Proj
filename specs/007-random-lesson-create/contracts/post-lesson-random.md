# API Contract: POST /api/lesson/random

**Date**: 2026-06-03
**Source of Truth**: [docs/api-spec.md](../../../docs/api-spec.md)

---

## Endpoint

```
POST /api/lesson/random
```

## Request

- **Content-Type**: 없음
- **Request Body**: 없음
- **Query Parameters**: 없음
- **Path Parameters**: 없음

## Response

### 201 Created

```json
{
  "id": 1
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 생성된 레슨 ID |

### 500 Internal Server Error

```json
{
  "status": 500,
  "errors": [
    {
      "field": null,
      "message": "서버 내부 오류가 발생했습니다."
    }
  ]
}
```

| 조건 | 에러 코드 |
|------|-----------|
| 강사 프로필 생성 또는 강사 지정 실패 | `INTERNAL_ERROR` |

---

## 내부 호출 흐름

이 엔드포인트는 아래 기존 API를 내부적으로 순서대로 호출한다.

```
1. GET /api/profiles?isInstructor=true      → 강사 목록 조회
2. (강사 없을 경우) POST /api/profile        → 신규 프로필 생성
3. (강사 없을 경우) PATCH /api/profile/{id}/instructor → 강사 지정
4. POST /api/lesson                          → 수업 생성
```

단, 이 흐름은 HTTP 호출이 아닌 **UseCase 직접 호출** 방식으로 구현된다.
