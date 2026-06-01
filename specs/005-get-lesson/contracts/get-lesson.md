# API Contract: GET /api/lessons/{lessonNo}

> 전체 명세는 `docs/api-spec.md` 참조. 이 파일은 구현 관점의 계약 요약이다.

## Endpoint

```
GET /api/lessons/{lessonNo}
```

## Path Parameter

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| lessonNo | Long | Y | 레슨 ID. Long 변환 실패 시 Spring이 400 자동 처리 |

## Request Body

없음

## Success Response — 200 OK

```json
{
  "id": 1,
  "title": "살사 초급반",
  "genre": "S",
  "instructorLo": "Ab2Cd3Ef",
  "instructorLa": null,
  "options": [
    {
      "id": 1,
      "startDate": "2026-06-01",
      "startTime": "10:00",
      "endDate": "2026-06-01",
      "endTime": "12:00",
      "region": "GN",
      "place": "강남 스튜디오",
      "placeUrl": "https://example.com/map"
    }
  ],
  "amount": 80000,
  "discounts": [
    {
      "id": 1,
      "type": "E",
      "condition": "2026-05-25",
      "amount": 10000
    }
  ],
  "account": {
    "id": 1,
    "bank": "카카오뱅크",
    "account": "3333-01-1234567",
    "name": "홍길동"
  },
  "contacts": [
    {
      "id": 1,
      "type": "K",
      "account": "kakao_id",
      "name": "카카오채널"
    }
  ],
  "isActive": true,
  "notices": [
    {
      "id": 1,
      "type": "N",
      "text": "환불은 수업 3일 전까지 가능합니다."
    }
  ]
}
```

## Error Response — 404 Not Found

```json
{
  "status": 404,
  "errors": [
    {
      "field": "lessonNo",
      "message": "레슨을 찾을 수 없습니다."
    }
  ]
}
```

**에러 코드**: `LESSON_NOT_FOUND`

## Null / Empty Array 처리 규칙

| 필드 | 없을 때 반환값 |
|------|---------------|
| instructorLo | null |
| instructorLa | null |
| amount | null |
| discounts | `[]` |
| account | null |
| contacts | `[]` |
| notices | `[]` |
| options | 최소 1개 보장 (생성 시 필수) |
