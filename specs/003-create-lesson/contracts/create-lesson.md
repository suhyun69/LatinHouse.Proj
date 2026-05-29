# API Contract: POST /api/lesson

**Source of truth**: `docs/api-spec.md`

## Endpoint

| 항목 | 내용 |
|------|------|
| Method | POST |
| Path | `/api/lesson` |
| Content-Type | `application/json` |
| Success Status | 201 Created |

## Request Body

```json
{
  "title": "string",
  "genre": "S",
  "instructorLo": "Ab2Cd3Ef",
  "instructorLa": null,
  "options": [
    {
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
    { "type": "E", "condition": "2026-05-25", "amount": 10000 },
    { "type": "S", "condition": "F", "amount": 5000 }
  ],
  "account": {
    "bank": "카카오뱅크",
    "account": "3333-01-1234567",
    "name": "홍길동"
  },
  "contacts": [
    { "type": "K", "account": "kakao_id", "name": "카카오채널" }
  ],
  "isActive": true,
  "notices": [
    { "type": "N", "text": "환불은 수업 3일 전까지 가능합니다." }
  ]
}
```

## Response Body (201 Created)

```json
{ "id": 1 }
```

## webRequest Validation (Bean Validation — 400)

| 필드 | 조건 | 에러 메시지 |
|------|------|------------|
| title | @NotBlank | 제목을 입력해 주세요. |
| genre | @NotBlank | 장르를 입력해 주세요. |
| genre | @Pattern(`^[SB]$`) | 장르는 S 또는 B만 입력 가능합니다. |
| options | @NotEmpty | 수업 옵션을 1개 이상 입력해 주세요. |
| options[].startDate | @NotBlank | 시작 날짜를 입력해 주세요. |
| options[].startDate | @Pattern(`^\d{4}-\d{2}-\d{2}$`) | 시작 날짜는 yyyy-MM-dd 형식으로 입력해 주세요. |
| options[].startTime | @NotBlank | 시작 시간을 입력해 주세요. |
| options[].startTime | @Pattern(`^\d{2}:\d{2}$`) | 시작 시간은 HH:mm 형식으로 입력해 주세요. |
| options[].endDate | @NotBlank | 종료 날짜를 입력해 주세요. |
| options[].endDate | @Pattern(`^\d{4}-\d{2}-\d{2}$`) | 종료 날짜는 yyyy-MM-dd 형식으로 입력해 주세요. |
| options[].endTime | @NotBlank | 종료 시간을 입력해 주세요. |
| options[].endTime | @Pattern(`^\d{2}:\d{2}$`) | 종료 시간은 HH:mm 형식으로 입력해 주세요. |
| options[].region | @NotBlank | 지역을 입력해 주세요. |
| options[].region | @Pattern(`^(GN\|HD)$`) | 지역은 GN 또는 HD만 입력 가능합니다. |
| discounts[].type | @Pattern(`^[ES]$`) | 할인 타입은 E 또는 S만 입력 가능합니다. |
| contacts[].type | @Pattern(`^[YKWILM]$`) | 연락처 타입이 올바르지 않습니다. |
| notices[].type | @Pattern(`^[LTNU]$` + R) | 공지 타입이 올바르지 않습니다. |

## appRequest Validation (Service — 400)

| 필드 | 에러 코드 | 에러 메시지 |
|------|-----------|------------|
| instructorLo+La 모두 null | VALIDATION_ERROR | 남성 강사 또는 여성 강사 중 하나는 반드시 입력해야 합니다. |
| instructorLo: 프로필 없음 | INSTRUCTOR_NOT_FOUND | 존재하지 않는 강사 ID입니다. |
| instructorLo: isInstructor=false | INSTRUCTOR_NOT_VALID | 강사로 등록되지 않은 프로필입니다. |
| instructorLo: sex=F | INSTRUCTOR_NOT_VALID | 남성 강사(instructorLo)에는 남성(M) 프로필만 등록 가능합니다. |
| instructorLa: 프로필 없음 | INSTRUCTOR_NOT_FOUND | 존재하지 않는 강사 ID입니다. |
| instructorLa: isInstructor=false | INSTRUCTOR_NOT_VALID | 강사로 등록되지 않은 프로필입니다. |
| instructorLa: sex=M | INSTRUCTOR_NOT_VALID | 여성 강사(instructorLa)에는 여성(F) 프로필만 등록 가능합니다. |
| options[i].startDateTime >= endDateTime | VALIDATION_ERROR | 시작 일시는 종료 일시보다 이전이어야 합니다. |
| discounts[i].type=E, condition 형식 오류 | VALIDATION_ERROR | 얼리버드 할인 조건은 yyyy-MM-dd 형식의 날짜여야 합니다. |
| discounts[i].type=S, condition ∉ {M,F} | VALIDATION_ERROR | 성별 할인 조건은 M 또는 F만 입력 가능합니다. |

## Error Response Format

```json
{
  "status": 400,
  "errors": [
    { "field": "필드명", "message": "에러 메시지" }
  ]
}
```
