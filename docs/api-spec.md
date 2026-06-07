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
| LESSON_NOT_FOUND | 레슨을 찾을 수 없음 |
| INTERNAL_ERROR | 서버 내부 오류 |

---

## Lesson

### GET /api/lessons
레슨 목록을 조회한다. 레슨은 Option 단위로 펼쳐서 반환한다.

#### Query Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| region | String | N | 지역 필터. `GN`(Gangnam) 또는 `HD`(Hongdae). 생략 시 전체 |
| instructor | String | N | 강사 필터. Profile.id. instructorLo 또는 instructorLa가 일치하는 레슨 반환 |
| genre | String | N | 장르 필터. `S`(Salsa) 또는 `B`(Bachata). 생략 시 전체 |

복수 필터는 AND 조건으로 결합된다.

#### Response
| Status | 설명 |
|--------|------|
| 200 OK | 레슨 옵션 목록 반환. 결과 없으면 빈 배열 반환 |

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

| 필드 | 타입 | 설명 |
|------|------|------|
| optionId | Long | 수업 옵션 ID |
| lessonNo | Long | 레슨 ID |
| instructorLo | String | 남성 강사 Profile.id. 없으면 null |
| instructorLa | String | 여성 강사 Profile.id. 없으면 null |
| title | String | 레슨 제목 |
| genre | String | 장르. `S`(Salsa) 또는 `B`(Bachata) |
| startDate | String | 옵션 시작 날짜. `yyyy-MM-dd` 형식 |
| startTime | String | 옵션 시작 시간. `HH:mm` 형식 |
| endDate | String | 옵션 종료 날짜. `yyyy-MM-dd` 형식 |
| endTime | String | 옵션 종료 시간. `HH:mm` 형식 |
| region | String | 옵션 지역. `GN`(Gangnam) 또는 `HD`(Hongdae) |
| price | BigDecimal | 수강료. 없으면 null |
| discountCondition | String | 적용 얼리버드 할인 마감일 (`yyyy-MM-dd`). 없으면 null |
| discountAmount | BigDecimal | 적용 얼리버드 할인 금액. 없으면 null |
| status | String | 수업 상태. `INACTIVE` / `PENDING` / `IN_PROGRESS` / `DONE` |

**status 계산 규칙**

| 조건 | status |
|------|--------|
| `Lesson.isActive = false` | `INACTIVE` |
| `isActive = true` 이고 현재 시점 < option.startDateTime | `PENDING` |
| `isActive = true` 이고 startDateTime ≤ 현재 시점 ≤ endDateTime | `IN_PROGRESS` |
| `isActive = true` 이고 현재 시점 > option.endDateTime | `DONE` |

**discount 계산 규칙 (얼리버드만 적용)**

1. `Lesson.discounts` 중 `type = "E"` (Earlybird) 인 것만 대상
2. `condition`을 날짜로 파싱하여 현재 날짜 이후인 것만 후보 (`LocalDate.parse(condition) >= LocalDate.now()`)
3. 후보가 없으면 `discountCondition = null`, `discountAmount = null`
4. 후보가 복수이면 `condition` 기준 오름차순 가장 빠른 1건 적용

#### Validation Error — 400 Bad Request

| 파라미터 | 조건 | 에러 메시지 |
|----------|------|------------|
| region | `GN`, `HD` 외의 값 | 지역은 GN 또는 HD만 입력 가능합니다. |
| genre | `S`, `B` 외의 값 | 장르는 S 또는 B만 입력 가능합니다. |

---

### GET /api/lessons/{lessonNo}
레슨 단건을 조회한다.

#### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| lessonNo | Long | Y | 레슨 ID |

#### Request Body
없음

#### Response
| Status | 설명 |
|--------|------|
| 200 OK | 레슨 단건 반환 |

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

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 레슨 ID |
| title | String | 레슨 제목 |
| genre | String | 장르. `S`(Salsa) 또는 `B`(Bachata) |
| instructorLo | String | 남성 강사 Profile.id. 없으면 null |
| instructorLa | String | 여성 강사 Profile.id. 없으면 null |
| options | List | 수업 옵션 목록 |
| options[].id | Long | 수업 옵션 ID |
| options[].startDate | String | 시작 날짜. `yyyy-MM-dd` 형식 |
| options[].startTime | String | 시작 시간. `HH:mm` 형식 |
| options[].endDate | String | 종료 날짜. `yyyy-MM-dd` 형식 |
| options[].endTime | String | 종료 시간. `HH:mm` 형식 |
| options[].region | String | 지역. `GN`(Gangnam) 또는 `HD`(Hongdae) |
| options[].place | String | 장소명. 없으면 null |
| options[].placeUrl | String | 장소 URL. 없으면 null |
| amount | BigDecimal | 수강료. 없으면 null |
| discounts | List | 할인 목록. 없으면 빈 배열 |
| discounts[].id | Long | 할인 ID |
| discounts[].type | String | `E`(Earlybird) 또는 `S`(Sex) |
| discounts[].condition | String | type=E: `yyyy-MM-dd` 날짜 / type=S: `M` 또는 `F` |
| discounts[].amount | BigDecimal | 할인 금액. 없으면 null |
| account | Object | 입금 계좌 정보. 없으면 null |
| account.id | Long | 계좌 ID |
| account.bank | String | 은행명. 없으면 null |
| account.account | String | 계좌번호. 없으면 null |
| account.name | String | 예금주. 없으면 null |
| contacts | List | 연락처 목록. 없으면 빈 배열 |
| contacts[].id | Long | 연락처 ID |
| contacts[].type | String | `Y`/`K`/`W`/`I`/`L`/`M` |
| contacts[].account | String | 연락처 계정. 없으면 null |
| contacts[].name | String | 표시명. 없으면 null |
| isActive | Boolean | 활성 여부 |
| notices | List | 공지 목록. 없으면 빈 배열 |
| notices[].id | Long | 공지 ID |
| notices[].type | String | `L`/`T`/`R`/`N`/`U` |
| notices[].text | String | 공지 내용. 없으면 null |

#### Error

| Status | 에러 코드 | 설명 |
|--------|-----------|------|
| 404 Not Found | `LESSON_NOT_FOUND` | 존재하지 않는 lessonNo |

---

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

### PUT /api/lesson/{lessonNo}
레슨 전체 데이터를 수정한다. 요청 바디에 포함된 값으로 기존 데이터를 전부 교체(replace)한다.

#### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| lessonNo | Long | Y | 수정할 레슨 ID |

#### Request Body

POST /api/lesson 요청 바디와 동일한 구조.

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

필드 규칙은 POST /api/lesson과 동일.

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | Y | 레슨 제목 |
| genre | String | Y | 장르. `S`(Salsa) 또는 `B`(Bachata) |
| instructorLo | String | △ | 남성 강사 Profile.id. instructorLa와 둘 중 하나 이상 필수 |
| instructorLa | String | △ | 여성 강사 Profile.id. instructorLo와 둘 중 하나 이상 필수 |
| options | List | Y | 수업 옵션. 1개 이상 필수. 기존 옵션 전체 교체 |
| options[].startDate | String | Y | 시작 날짜. `yyyy-MM-dd` 형식 |
| options[].startTime | String | Y | 시작 시간. `HH:mm` 형식 |
| options[].endDate | String | Y | 종료 날짜. `yyyy-MM-dd` 형식 |
| options[].endTime | String | Y | 종료 시간. `HH:mm` 형식. startDate+startTime보다 이후여야 함 |
| options[].region | String | Y | 지역. `GN`(Gangnam) 또는 `HD`(Hongdae) |
| options[].place | String | N | 장소명 |
| options[].placeUrl | String | N | 장소 URL |
| amount | BigDecimal | N | 수강료 |
| discounts | List | N | 할인 목록. 기존 할인 전체 교체 |
| discounts[].type | String | Y | `E`(Earlybird) 또는 `S`(Sex) |
| discounts[].condition | String | Y | type=E: `yyyy-MM-dd` 날짜 / type=S: `M` 또는 `F` |
| discounts[].amount | BigDecimal | N | 할인 금액 |
| account | Object | N | 입금 계좌 정보. null이면 기존 계좌 삭제 |
| account.bank | String | N | 은행명 |
| account.account | String | N | 계좌번호 |
| account.name | String | N | 예금주 |
| contacts | List | N | 연락처 목록. 기존 연락처 전체 교체 |
| contacts[].type | String | Y | `Y`/`K`/`W`/`I`/`L`/`M` |
| contacts[].account | String | N | 연락처 계정 |
| contacts[].name | String | N | 표시명 |
| isActive | Boolean | N | 활성 여부. 기본값 `true` |
| notices | List | N | 공지 목록. 기존 공지 전체 교체 |
| notices[].type | String | Y | `L`/`T`/`R`/`N`/`U` |
| notices[].text | String | N | 공지 내용 |

#### Response

| Status | 설명 |
|--------|------|
| 200 OK | 수정 성공. 수정된 레슨 id 반환 |

```json
{
  "id": 1
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 수정된 레슨 ID |

#### Validation Error — 400 Bad Request (webRequest)

POST /api/lesson과 동일한 검증 규칙 적용.

#### Business Rule Error — 400 Bad Request (appRequest)

POST /api/lesson과 동일한 비즈니스 규칙 적용.

#### Error

| Status | 에러 코드 | 설명 |
|--------|-----------|------|
| 404 Not Found | `LESSON_NOT_FOUND` | 존재하지 않는 lessonNo |

---

### POST /api/lesson/random
`POST /api/lesson` 실행에 필요한 파라미터를 랜덤으로 생성하여 수업을 생성한다.

강사는 기존 프로필 중 `isInstructor=true` 조건으로 조회하여 랜덤 할당한다.  
조건을 만족하는 프로필이 없는 경우 신규 프로필을 생성한 뒤 강사로 지정하여 사용한다.

#### Request Body
없음

#### Response
| Status | 설명 |
|--------|------|
| 201 Created | 수업 생성 성공. 생성된 레슨 id 반환 |

```json
{
  "id": 1
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 생성된 레슨 ID |

#### 랜덤 생성 규칙

**강사 할당 규칙**

1. `GET /api/profiles?isInstructor=true` 로 강사 프로필 목록 조회
2. 남성(`sex=M`) 강사와 여성(`sex=F`) 강사를 각각 분리
3. 남성/여성 강사 각각에 대해 랜덤 선택 (없으면 신규 생성 후 강사 지정)
4. 두 강사 중 하나만 랜덤으로 선택하거나 둘 다 할당 가능 (단, 최소 1명 이상 할당)

**신규 프로필 생성 규칙 (강사 없을 때)**

| 항목 | 규칙 |
|------|------|
| nickname | `강사_M_<랜덤 4자리 숫자>` (남성) 또는 `강사_F_<랜덤 4자리 숫자>` (여성) |
| sex | 필요한 성별 (`M` 또는 `F`) |

생성 후 `PATCH /api/profile/{profileId}/instructor` 호출하여 강사 지정.

**레슨 필드 랜덤 생성 규칙**

| 필드 | 규칙 |
|------|------|
| title | `<genre명> <레벨>반` 형식. genre=S → `살사`, genre=B → `바차타`. 레벨은 `초급` / `중급` / `상급` 중 랜덤 |
| genre | `S` 또는 `B` 중 랜덤 |
| instructorLo | 위 강사 할당 규칙에 따라 결정된 남성 강사 ID 또는 null |
| instructorLa | 위 강사 할당 규칙에 따라 결정된 여성 강사 ID 또는 null |
| options | 1~3개 랜덤 생성 |
| options[].startDate | 오늘로부터 7~60일 이내 랜덤 날짜 (`yyyy-MM-dd`) |
| options[].startTime | `10:00` / `14:00` / `19:00` / `20:00` 중 랜덤 |
| options[].endDate | startDate와 동일 |
| options[].endTime | startTime + 2시간 |
| options[].region | `GN` 또는 `HD` 중 랜덤 |
| options[].place | null |
| options[].placeUrl | null |
| amount | `30000` / `50000` / `80000` / `100000` 중 랜덤 |
| discounts | 0~2개 랜덤 생성 |
| discounts[].type | `E` 또는 `S` 중 랜덤 |
| discounts[].condition | type=E: options 중 가장 이른 startDate 기준 7일 전 날짜 / type=S: `M` 또는 `F` 중 랜덤 |
| discounts[].amount | `5000` / `10000` / `15000` 중 랜덤 |
| account | null |
| contacts | null |
| isActive | `true` 고정 |
| notices | null |

#### Error

| Status | 에러 코드 | 설명 |
|--------|-----------|------|
| 500 Internal Server Error | `INTERNAL_ERROR` | 프로필 생성 또는 강사 지정 실패 |

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

---

## Order

### POST /api/order
주문을 생성한다. 구매자 프로필, 레슨, 수업 옵션을 지정하면 주문 레코드가 생성된다. 레슨에 할인이 있을 경우 구매자 조건에 맞는 항목을 자동으로 적용한다.

#### Request Body

```json
{
  "lessonNo": 1,
  "lessonOptionNo": 3,
  "profileId": "Ab2Cd3Ef"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| lessonNo | Long | Y | 레슨 ID |
| lessonOptionNo | Long | Y | 수업 옵션 ID |
| profileId | String | Y | 구매자 프로필 ID |

#### Response

| Status | 설명 |
|--------|------|
| 201 Created | 주문 생성 성공. 생성된 주문 ID 반환 |

```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000"
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| orderId | String | 생성된 주문 ID (UUID) |

#### Validation Error — 400 Bad Request

| 필드 | 조건 | 에러 메시지 |
|------|------|------------|
| lessonNo | null | "레슨을 선택해 주세요." |
| lessonOptionNo | null | "수업 옵션을 선택해 주세요." |
| profileId | null 또는 빈 문자열 | "구매자 프로필을 입력해 주세요." |

#### 할인 적용 규칙

주문 생성 시 `Lesson.discounts` 목록을 조회하여 아래 조건을 충족하는 항목만 `Order.discounts`에 기록한다.

| 할인 유형 | 적용 조건 |
|-----------|-----------|
| SEX | `LessonDiscount.condition`이 구매자 `Profile.sex`와 일치하는 경우에만 적용 |
| EARLYBIRD | 주문 생성 시점(`now`) 기준으로 `condition`(yyyy-MM-dd) 날짜가 **아직 지나지 않은** 항목 중 가장 이른 1건만 적용. `condition < now`인 항목은 제외 |
| COUPON | `Coupon.owner = profileId`이고 `Coupon.status = AVAILABLE`인 쿠폰 중, `CouponTemplate.type = LESSON`이고 `CouponTemplate.target = lessonNo`인 쿠폰을 모두 적용. `discountId = Coupon.id`, `amount = CouponTemplate.amount` |

적용된 각 할인 항목은 `OrderDiscount`로 저장되며, LESSON 유형은 `discountType = LESSON`, `discountId = LessonDiscount.id`로, COUPON 유형은 `discountType = COUPON`, `discountId = Coupon.id`로 설정된다.

#### Error

| Status | 에러 코드 | 설명 |
|--------|-----------|------|
| 404 Not Found | `LESSON_NOT_FOUND` | 존재하지 않는 lessonNo |
| 404 Not Found | `LESSON_OPTION_NOT_FOUND` | 존재하지 않는 lessonOptionNo |
| 404 Not Found | `PROFILE_NOT_FOUND` | 존재하지 않는 profileId |

---

### GET /api/orders

주문 목록을 조회한다. buyer 또는 lessonNo로 필터링할 수 있으며, 두 조건은 AND로 결합된다.

**Query Parameters** (모두 선택사항)

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| buyer | String | N | 구매자 Profile ID로 필터링 |
| lessonNo | Long | N | 특정 레슨의 주문만 필터링 |

**Response — 200 OK**

```json
[
  {
    "orderId": "550e8400-e29b-41d4-a716-446655440000",
    "lessonNo": 1,
    "lessonOptionNo": 3,
    "price": 80000,
    "status": "PAYMENT_PENDING",
    "discounts": [
      {
        "discountType": "LESSON",
        "discountId": 10,
        "amount": 5000
      }
    ]
  }
]
```

파라미터 조건에 맞는 주문이 없으면 빈 배열 `[]` 반환. 에러 없음 (유효하지 않은 buyer/lessonNo는 결과 없음으로 처리).

---

## Coupon API

### POST /api/coupon/template

쿠폰 템플릿을 생성한다.

**Request Body**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | O | 쿠폰 템플릿 이름 |
| type | String | O | 쿠폰 유형. `CouponTemplateType` 값 (`LESSON`) |
| target | Long | O | 적용 대상 ID (type=LESSON이면 lessonNo) |
| amount | BigDecimal | O | 할인 금액 |

**Response — 201 Created**

```json
{
  "couponTemplateId": "1"
}
```

**실패 응답**

| 조건 | Status | 에러 코드 |
|------|--------|-----------|
| 필수 필드 누락 또는 null | 400 Bad Request | - |

---

### POST /api/coupon

쿠폰 템플릿을 기반으로 쿠폰을 count 개수만큼 일괄 발행한다.

**Request Body**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| templateId | Long | O | 발행 기준이 될 쿠폰 템플릿 ID |
| count | Integer | O | 발행할 쿠폰 수량 (1 이상) |

**Response — 201 Created** (Body 없음)

**실패 응답**

| 조건 | Status | 에러 코드 |
|------|--------|-----------|
| 존재하지 않는 templateId | 404 Not Found | `COUPON_TEMPLATE_NOT_FOUND` |
| count < 1 | 400 Bad Request | - |

---

### PATCH /api/coupon/{profileId}

쿠폰의 소유자(owner)를 지정된 프로필로 배정한다.

**Path Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| profileId | String | O | 쿠폰을 배정받을 프로필 ID |

**Request Body**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| couponId | Long | O | 배정할 쿠폰 ID |

**Response — 200 OK**

```json
{
  "couponId": 1
}
```

**실패 응답**

| 조건 | Status | 에러 코드 |
|------|--------|-----------|
| 존재하지 않는 profileId | 404 Not Found | `PROFILE_NOT_FOUND` |
| 존재하지 않는 couponId | 404 Not Found | `COUPON_NOT_FOUND` |
