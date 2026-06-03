# API Contract: PUT /api/lesson/{lessonNo}

**Date**: 2026-06-03
**Reference**: [docs/api-spec.md](../../../docs/api-spec.md)

---

## Endpoint

```
PUT /api/lesson/{lessonNo}
```

## Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| lessonNo | Long | Y | 수정할 레슨 ID |

## Request Body

`Content-Type: application/json`

```json
{
  "title": "살사 중급반",
  "genre": "S",
  "instructorLo": "Ab2Cd3Ef",
  "instructorLa": null,
  "options": [
    {
      "startDate": "2026-07-01",
      "startTime": "19:00",
      "endDate": "2026-07-01",
      "endTime": "21:00",
      "region": "GN",
      "place": "강남 스튜디오",
      "placeUrl": "https://example.com/map"
    }
  ],
  "amount": 80000,
  "discounts": [
    {
      "type": "E",
      "condition": "2026-06-20",
      "amount": 10000
    }
  ],
  "account": {
    "bank": "카카오뱅크",
    "account": "3333-01-1234567",
    "name": "홍길동"
  },
  "contacts": [
    {
      "type": "K",
      "account": "kakao_id",
      "name": "카카오채널"
    }
  ],
  "isActive": true,
  "notices": [
    {
      "type": "N",
      "text": "환불은 수업 3일 전까지 가능합니다."
    }
  ]
}
```

## Response

### 200 OK — 수정 성공

```json
{
  "id": 1
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 수정된 레슨 ID |

### 400 Bad Request — 요청 검증 실패

```json
{
  "status": 400,
  "errors": [
    {
      "field": "title",
      "message": "제목을 입력해 주세요."
    }
  ]
}
```

**Web Layer 검증 오류 (Bean Validation)**

| 필드 | 조건 | 에러 메시지 |
|------|------|------------|
| title | null 또는 빈 문자열 | 제목을 입력해 주세요. |
| genre | null 또는 빈 문자열 | 장르를 입력해 주세요. |
| genre | S, B 외의 값 | 장르는 S 또는 B만 입력 가능합니다. |
| options | null 또는 빈 리스트 | 수업 옵션을 1개 이상 입력해 주세요. |
| options[].startDate | null 또는 빈 문자열 | 시작 날짜를 입력해 주세요. |
| options[].startDate | yyyy-MM-dd 형식이 아닌 경우 | 시작 날짜는 yyyy-MM-dd 형식으로 입력해 주세요. |
| options[].startTime | null 또는 빈 문자열 | 시작 시간을 입력해 주세요. |
| options[].startTime | HH:mm 형식이 아닌 경우 | 시작 시간은 HH:mm 형식으로 입력해 주세요. |
| options[].endDate | null 또는 빈 문자열 | 종료 날짜를 입력해 주세요. |
| options[].endDate | yyyy-MM-dd 형식이 아닌 경우 | 종료 날짜는 yyyy-MM-dd 형식으로 입력해 주세요. |
| options[].endTime | null 또는 빈 문자열 | 종료 시간을 입력해 주세요. |
| options[].endTime | HH:mm 형식이 아닌 경우 | 종료 시간은 HH:mm 형식으로 입력해 주세요. |
| options[].region | null 또는 빈 문자열 | 지역을 입력해 주세요. |
| options[].region | GN, HD 외의 값 | 지역은 GN 또는 HD만 입력 가능합니다. |
| discounts[].type | E, S 외의 값 | 할인 타입은 E 또는 S만 입력 가능합니다. |
| contacts[].type | Y, K, W, I, L, M 외의 값 | 연락처 타입이 올바르지 않습니다. |
| notices[].type | L, T, R, N, U 외의 값 | 공지 타입이 올바르지 않습니다. |

**Application Layer 비즈니스 규칙 오류**

| 조건 | 에러 코드 | 에러 메시지 |
|------|-----------|------------|
| instructorLo, instructorLa 모두 null | VALIDATION_ERROR | 남성 강사 또는 여성 강사 중 하나는 반드시 입력해야 합니다. |
| instructorLo에 해당하는 프로필 없음 | INSTRUCTOR_NOT_FOUND | 존재하지 않는 강사 ID입니다. |
| instructorLo의 isInstructor=false | INSTRUCTOR_NOT_VALID | 강사로 등록되지 않은 프로필입니다. |
| instructorLo의 sex=F | INSTRUCTOR_NOT_VALID | 남성 강사(instructorLo)에는 남성(M) 프로필만 등록 가능합니다. |
| instructorLa에 해당하는 프로필 없음 | INSTRUCTOR_NOT_FOUND | 존재하지 않는 강사 ID입니다. |
| instructorLa의 isInstructor=false | INSTRUCTOR_NOT_VALID | 강사로 등록되지 않은 프로필입니다. |
| instructorLa의 sex=M | INSTRUCTOR_NOT_VALID | 여성 강사(instructorLa)에는 여성(F) 프로필만 등록 가능합니다. |
| options[].startDateTime >= endDateTime | VALIDATION_ERROR | 시작 일시는 종료 일시보다 이전이어야 합니다. |
| discounts[].type=E이고 condition이 yyyy-MM-dd 아님 | VALIDATION_ERROR | 얼리버드 할인 조건은 yyyy-MM-dd 형식의 날짜여야 합니다. |
| discounts[].type=S이고 condition이 M, F 외의 값 | VALIDATION_ERROR | 성별 할인 조건은 M 또는 F만 입력 가능합니다. |

### 404 Not Found — 레슨 미존재

```json
{
  "status": 404,
  "errors": [
    {
      "field": null,
      "message": "존재하지 않는 레슨입니다. lessonNo=999"
    }
  ]
}
```

| Status | 에러 코드 | 설명 |
|--------|-----------|------|
| 404 Not Found | `LESSON_NOT_FOUND` | 존재하지 않는 lessonNo |
