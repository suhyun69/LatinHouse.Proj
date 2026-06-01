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
| INSTRUCTOR_NOT_FOUND | 강사 프로필을 찾을 수 없음 |
| INSTRUCTOR_NOT_VALID | 강사 조건 미충족 (isInstructor=false 또는 sex 불일치) |
| INTERNAL_ERROR | 서버 내부 오류 |

---

## Lesson

### POST /api/lesson
레슨을 생성한다.

#### Request
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
    {
      "type": "E",
      "condition": "2026-05-25",
      "amount": 10000
    },
    {
      "type": "S",
      "condition": "F",
      "amount": 5000
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

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | Y | 레슨 제목 |
| genre | String | Y | 장르. `S`(Salsa) 또는 `B`(Bachata) |
| instructorLo | String | △ | 남성 강사 Profile.id. instructorLa와 둘 중 하나 이상 필수 |
| instructorLa | String | △ | 여성 강사 Profile.id. instructorLo와 둘 중 하나 이상 필수 |
| options | List | Y | 수업 옵션. 1개 이상 필수 |
| options[].startDate | String | Y | 시작 날짜. `yyyy-MM-dd` 형식 |
| options[].startTime | String | Y | 시작 시간. `HH:mm` 형식 |
| options[].endDate | String | Y | 종료 날짜. `yyyy-MM-dd` 형식 |
| options[].endTime | String | Y | 종료 시간. `HH:mm` 형식. startDate+startTime보다 이후여야 함 |
| options[].region | String | Y | 지역. `GN`(Gangnam) 또는 `HD`(Hongdae) |
| options[].place | String | N | 장소명 |
| options[].placeUrl | String | N | 장소 URL |
| amount | BigDecimal | N | 수강료 |
| discounts | List | N | 할인 목록 |
| discounts[].type | String | Y | `E`(Earlybird) 또는 `S`(Sex) |
| discounts[].condition | String | Y | type=E: `yyyy-MM-dd` 날짜 / type=S: `M` 또는 `F` |
| discounts[].amount | BigDecimal | N | 할인 금액 |
| account | Object | N | 입금 계좌 정보 |
| account.bank | String | N | 은행명 |
| account.account | String | N | 계좌번호 |
| account.name | String | N | 예금주 |
| contacts | List | N | 연락처 목록 |
| contacts[].type | String | Y | `Y`/`K`/`W`/`I`/`L`/`M` |
| contacts[].account | String | N | 연락처 계정 |
| contacts[].name | String | N | 표시명 |
| isActive | Boolean | N | 활성 여부. 기본값 `true` |
| notices | List | N | 공지 목록 |
| notices[].type | String | Y | `L`/`T`/`R`/`N`/`U` |
| notices[].text | String | N | 공지 내용 |

#### Response
| Status | 설명 |
|--------|------|
| 201 Created | 레슨 생성 성공. 생성된 레슨 id 반환 |

```json
{
  "id": 1
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 생성된 레슨 ID |

#### Validation Error — 400 Bad Request (webRequest)

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

#### Business Rule Error — 400 Bad Request (appRequest)

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
| discounts[].type=E이고 condition이 yyyy-MM-dd 형식 아님 | VALIDATION_ERROR | 얼리버드 할인 조건은 yyyy-MM-dd 형식의 날짜여야 합니다. |
| discounts[].type=S이고 condition이 M, F 외의 값 | VALIDATION_ERROR | 성별 할인 조건은 M 또는 F만 입력 가능합니다. |

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

### GET /api/profiles
프로필 목록을 조회한다.

#### Query Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| isInstructor | Boolean | N | 강사 여부 필터. `true` = 강사만, `false` = 비강사만, 생략 시 전체 반환 |

#### Response
| Status | 설명 |
|--------|------|
| 200 OK | 프로필 목록 반환. 결과 없으면 빈 배열 반환 |

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

#### Validation Error — 400 Bad Request

| 파라미터 | 조건 | 에러 메시지 |
|----------|------|------------|
| isInstructor | `true`, `false` 외의 값 | isInstructor는 true 또는 false만 입력 가능합니다. |

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
