# API Contract: GET /api/profiles

## Endpoint

```
GET /api/profiles
```

## Query Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| isInstructor | Boolean | N | 강사 여부 필터. `true` = 강사만, `false` = 비강사만, 생략 시 전체 반환 |

## Response

### 200 OK

```json
[
  {
    "id": "Ab2Cd3Ef",
    "nickname": "홍길동",
    "sex": "M",
    "isInstructor": true
  }
]
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | String | 프로필 ID (8자리 랜덤 문자열) |
| nickname | String | 닉네임 |
| sex | String | 성별. `M` 또는 `F` |
| isInstructor | Boolean | 강사 여부 |

결과가 없을 때: `[]` (빈 배열, 200 OK)

### 400 Bad Request

`isInstructor`에 `true`, `false` 외의 값이 입력된 경우.

```json
{
  "status": 400,
  "errors": [
    {
      "field": "isInstructor",
      "message": "isInstructor는 true 또는 false만 입력 가능합니다."
    }
  ]
}
```

## Notes

- 인증 불필요 (공개 엔드포인트)
- 페이지네이션 미지원 (전체 목록 반환)
- 정렬 기준: DB 기본 순서 (등록 순)
