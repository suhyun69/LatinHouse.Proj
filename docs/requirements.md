# Requirements

## 1. 기능 요구사항 (Functional Requirements)

### FR-P-001: 프로필 생성

**설명**: 닉네임과 성별을 입력받아 프로필을 생성한다.

**API 매핑**: `POST /api/profile`

**입력 필드**:
| 필드 | 타입 | 필수 | 제약조건 | 기본값 |
|------|------|------|----------|--------|
| nickname | string | O | - | - |
| sex | string | O | M 또는 F | - |

**처리 규칙**:
- id는 8자리 난수 자동 생성 (대/소문자+숫자, i I 1 l 0 o O 제외)
- isInstructor는 false로 자동 설정

**검증 에러 메시지**:
| 조건 | 메시지 |
|------|--------|
| nickname 누락 | "닉네임을 입력해 주세요." |
| sex 누락 | "성별을 입력해 주세요." |
| sex가 M, F 외의 값 | "성별은 M 또는 F만 입력 가능합니다." |

**성공 응답**: 201 Created (Body 없음)

**실패 응답**: 400 Bad Request + 검증 에러 상세
```json
{
  "status": 400,
  "errors": [{ "field": "필드명", "message": "에러 메시지" }]
}
```

---

### FR-L-001: 레슨 생성

**설명**: 레슨 정보를 입력받아 레슨을 생성한다.

**API 매핑**: `POST /api/lesson`

**입력 필드**:

| 필드 | 타입 | 필수 | 검증 레이어 | 제약조건 |
|------|------|------|-------------|----------|
| title | String | O | webRequest | - |
| genre | String | O | webRequest | `S` 또는 `B` |
| instructorLo | String | △ | appRequest | instructorLa와 둘 중 하나 이상 필수. isInstructor=true, sex=M인 Profile.id |
| instructorLa | String | △ | appRequest | instructorLo와 둘 중 하나 이상 필수. isInstructor=true, sex=F인 Profile.id |
| options | List | O | webRequest | 1개 이상 |
| options[].startDate | String | O | webRequest | `yyyy-MM-dd` 형식 |
| options[].startTime | String | O | webRequest | `HH:mm` 형식 |
| options[].endDate | String | O | webRequest | `yyyy-MM-dd` 형식 |
| options[].endTime | String | O | webRequest | `HH:mm` 형식 |
| options[].region | String | O | webRequest | `GN` 또는 `HD` |
| options[].place | String | X | - | - |
| options[].placeUrl | String | X | - | - |
| amount | BigDecimal | X | - | - |
| discounts | List | X | - | - |
| discounts[].type | String | O | webRequest | `E` 또는 `S` |
| discounts[].condition | String | O | appRequest | type=E: `yyyy-MM-dd` 형식 / type=S: `M` 또는 `F` |
| discounts[].amount | BigDecimal | X | - | - |
| account | Object | X | - | - |
| account.bank | String | X | - | - |
| account.account | String | X | - | - |
| account.name | String | X | - | - |
| contacts | List | X | - | - |
| contacts[].type | String | O | webRequest | `Y` / `K` / `W` / `I` / `L` / `M` |
| contacts[].account | String | X | - | - |
| contacts[].name | String | X | - | - |
| isActive | Boolean | X | - | 기본값 true |
| notices | List | X | - | - |
| notices[].type | String | O | webRequest | `L` / `T` / `R` / `N` / `U` |
| notices[].text | String | X | - | - |

**검증 레이어 구분**:
- **webRequest**: 형식·타입·필수값 등 기본 입력 검증
- **appRequest**: 도메인 규칙 기반 비즈니스 검증

**webRequest 검증 에러 메시지**:

| 필드 | 조건 | 메시지 |
|------|------|--------|
| title | null 또는 빈 문자열 | "제목을 입력해 주세요." |
| genre | null 또는 빈 문자열 | "장르를 입력해 주세요." |
| genre | S, B 외의 값 | "장르는 S 또는 B만 입력 가능합니다." |
| options | null 또는 빈 리스트 | "수업 옵션을 1개 이상 입력해 주세요." |
| options[].startDate | null 또는 빈 문자열 | "시작 날짜를 입력해 주세요." |
| options[].startDate | yyyy-MM-dd 형식이 아닌 경우 | "시작 날짜는 yyyy-MM-dd 형식으로 입력해 주세요." |
| options[].startTime | null 또는 빈 문자열 | "시작 시간을 입력해 주세요." |
| options[].startTime | HH:mm 형식이 아닌 경우 | "시작 시간은 HH:mm 형식으로 입력해 주세요." |
| options[].endDate | null 또는 빈 문자열 | "종료 날짜를 입력해 주세요." |
| options[].endDate | yyyy-MM-dd 형식이 아닌 경우 | "종료 날짜는 yyyy-MM-dd 형식으로 입력해 주세요." |
| options[].endTime | null 또는 빈 문자열 | "종료 시간을 입력해 주세요." |
| options[].endTime | HH:mm 형식이 아닌 경우 | "종료 시간은 HH:mm 형식으로 입력해 주세요." |
| options[].region | null 또는 빈 문자열 | "지역을 입력해 주세요." |
| options[].region | GN, HD 외의 값 | "지역은 GN 또는 HD만 입력 가능합니다." |
| discounts[].type | E, S 외의 값 | "할인 타입은 E 또는 S만 입력 가능합니다." |
| contacts[].type | Y, K, W, I, L, M 외의 값 | "연락처 타입이 올바르지 않습니다." |
| notices[].type | L, T, R, N, U 외의 값 | "공지 타입이 올바르지 않습니다." |

**appRequest 검증 에러 메시지**:

| 조건 | 메시지 |
|------|--------|
| instructorLo, instructorLa 모두 null | "남성 강사 또는 여성 강사 중 하나는 반드시 입력해야 합니다." |
| instructorLo에 해당하는 프로필 없음 | "존재하지 않는 강사 ID입니다." |
| instructorLo의 isInstructor=false | "강사로 등록되지 않은 프로필입니다." |
| instructorLo의 sex=F | "남성 강사(instructorLo)에는 남성(M) 프로필만 등록 가능합니다." |
| instructorLa에 해당하는 프로필 없음 | "존재하지 않는 강사 ID입니다." |
| instructorLa의 isInstructor=false | "강사로 등록되지 않은 프로필입니다." |
| instructorLa의 sex=M | "여성 강사(instructorLa)에는 여성(F) 프로필만 등록 가능합니다." |
| options[].startDateTime >= endDateTime | "시작 일시는 종료 일시보다 이전이어야 합니다." |
| discounts[].type=E이고 condition이 yyyy-MM-dd 형식이 아닌 경우 | "얼리버드 할인 조건은 yyyy-MM-dd 형식의 날짜여야 합니다." |
| discounts[].type=S이고 condition이 M, F 외의 값인 경우 | "성별 할인 조건은 M 또는 F만 입력 가능합니다." |

**성공 응답**: 201 Created
```json
{
  "id": 1
}
```

**실패 응답**: 400 Bad Request + 검증 에러 상세
```json
{
  "status": 400,
  "errors": [{ "field": "필드명", "message": "에러 메시지" }]
}
```

---

### FR-P-002: 강사 지정

**설명**: 프로필의 강사 여부를 `true`로 변경한다.

**API 매핑**: `PATCH /api/profile/{profileId}/instructor`

**입력 필드**:
| 파라미터 | 타입 | 필수 | 제약조건 |
|----------|------|------|----------|
| profileId | String (Path) | O | 존재하는 프로필 ID |

**처리 규칙**:
- `profileId`에 해당하는 프로필의 `isInstructor`를 `true`로 변경한다.
- 이미 `isInstructor`가 `true`인 경우에도 정상 처리(멱등)한다.

**성공 응답**: 200 OK
```json
{
  "id": "Ab2Cd3Ef"
}
```

**실패 응답**:
| 조건 | Status | 에러 코드 |
|------|--------|-----------|
| 존재하지 않는 profileId | 404 Not Found | `PROFILE_NOT_FOUND` |

---

### FR-O-001: 주문 생성

**설명**: 구매자 프로필, 레슨, 수업 옵션을 지정하여 주문을 생성한다.

**API 매핑**: `POST /api/order`

**입력 필드**:
| 필드 | 타입 | 필수 | 제약조건 |
|------|------|------|----------|
| lessonNo | Long | O | 존재하는 Lesson.id |
| lessonOptionNo | Long | O | 존재하는 LessonOption.id |
| profileId | String | O | 존재하는 Profile.id |

**처리 규칙**:
- orderId는 UUID로 자동 생성한다.
- status는 `PAYMENT_PENDING`으로 초기화한다.
- price는 레슨의 수강료(`Lesson.amount`)를 기준으로 설정한다. 실제 결제 금액은 `payment.amount`이며, `price = payment.amount + sum(discounts.amount)` 관계를 만족해야 한다.
- lessonOptionNo는 lessonNo에 속한 옵션이어야 한다.
- `Lesson.discounts`를 조회하여 구매자 조건에 맞는 항목만 `Order.discounts`로 저장한다.
  - **SEX 할인**: `LessonDiscount.condition`이 구매자 `Profile.sex`와 일치하는 경우에만 적용. 불일치 시 제외.
  - **EARLYBIRD 할인**: 주문 생성 시점(`now`) 기준으로 `condition`(yyyy-MM-dd) 날짜가 아직 지나지 않은(`condition >= now`) 항목만 후보로 한다. 후보 중 `condition` 날짜가 가장 이른 1건만 적용. 후보가 없으면 적용하지 않는다.
  - 적용된 각 항목은 `discountType = LESSON`, `discountId = LessonDiscount.id`로 `OrderDiscount`에 저장된다.

**webRequest 검증 에러 메시지**:
| 필드 | 조건 | 메시지 |
|------|------|--------|
| lessonNo | null | "레슨을 선택해 주세요." |
| lessonOptionNo | null | "수업 옵션을 선택해 주세요." |
| profileId | null 또는 빈 문자열 | "구매자 프로필을 입력해 주세요." |

**성공 응답**: 201 Created
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

### FR-O-002: 주문 목록 조회

**설명**: buyer(구매자 Profile ID) 또는 lessonNo(레슨 번호)를 조건으로 주문 목록을 조회한다.

**API 매핑**: `GET /api/orders`

**입력 필드**:
| 필드 | 타입 | 필수 | 제약조건 |
|------|------|------|----------|
| buyer | String | X | 구매자 Profile ID |
| lessonNo | Long | X | 레슨 번호 |

**처리 규칙**:
- 두 파라미터 모두 생략 시 전체 주문 목록을 반환한다.
- 두 파라미터 동시 지정 시 AND 조건으로 필터링한다.
- 조건에 해당하는 주문이 없으면 빈 배열을 반환한다. 에러를 발생시키지 않는다.

**성공 응답**: 200 OK
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

**실패 응답**:
| 조건 | Status | 에러 코드 |
|------|--------|-----------|
| 존재하지 않는 lessonNo | 404 Not Found | `LESSON_NOT_FOUND` |
| 존재하지 않는 lessonOptionNo | 404 Not Found | `LESSON_OPTION_NOT_FOUND` |
| 존재하지 않는 profileId | 404 Not Found | `PROFILE_NOT_FOUND` |

---
