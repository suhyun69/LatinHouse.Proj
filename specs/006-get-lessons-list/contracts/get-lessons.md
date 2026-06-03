# Contract: GET /api/lessons

## Request

```
GET /api/lessons?region={region}&instructor={instructor}&genre={genre}
```

### Query Parameters

| 파라미터 | 타입 | 필수 | 유효값 | 설명 |
|----------|------|------|--------|------|
| region | String | N | `GN`, `HD` | 생략 시 전체 |
| instructor | String | N | Profile.id | instructorLo 또는 instructorLa 일치 |
| genre | String | N | `S`, `B` | 생략 시 전체 |

---

## Response

### 200 OK

```json
[
  {
    "optionId": 1,
    "lessonNo": 10,
    "instructorLo": "Ab2Cd3Ef",
    "instructorLa": null,
    "title": "살사 초급반",
    "genre": "S",
    "startDate": "2026-07-01",
    "startTime": "10:00",
    "endDate": "2026-07-01",
    "endTime": "12:00",
    "region": "GN",
    "price": 80000,
    "discountCondition": "2026-06-20",
    "discountAmount": 10000,
    "status": "PENDING"
  }
]
```

결과 없으면 빈 배열 `[]` 반환.

### 400 Bad Request

```json
{
  "status": 400,
  "errors": [
    {
      "field": "region",
      "message": "지역은 GN 또는 HD만 입력 가능합니다."
    }
  ]
}
```

---

## 비즈니스 규칙 요약

### status

| 조건 | 값 |
|------|----|
| `isActive = false` | `INACTIVE` |
| `isActive = true`, `now < startDateTime` | `PENDING` |
| `isActive = true`, `startDateTime ≤ now ≤ endDateTime` | `IN_PROGRESS` |
| `isActive = true`, `now > endDateTime` | `DONE` |

### discount (EarlyBird만)

```
candidates = discounts
  .filter(type == EARLYBIRD)
  .filter(LocalDate.parse(condition) >= today)
  .sortedBy(condition ASC)
→ candidates.first() 적용, 없으면 null
```

---

## 에러 코드

| Status | 에러 코드 | 조건 |
|--------|-----------|------|
| 400 | `VALIDATION_ERROR` | region이 GN, HD 외의 값 |
| 400 | `VALIDATION_ERROR` | genre가 S, B 외의 값 |
