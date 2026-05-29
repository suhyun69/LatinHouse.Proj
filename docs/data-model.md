# Data Model

## Profile

### Entity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK | 8자리 난수. 대/소문자+숫자 혼용, `i I 1 l 0 o O` 제외 |
| nickname | String | NOT NULL | 중복 허용 |
| sex | String | | `"M"` 또는 `"F"` |
| isInstructor | Boolean | default false | 강사 여부. `PATCH /api/profile/{profileId}/instructor` 호출 시 `true`로 변경됨 |

### Domain

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK | 8자리 난수. 대/소문자+숫자 혼용, `i I 1 l 0 o O` 제외 |
| nickname | String | Not null | 중복 허용 |
| sex | Sex | Enum | `M(Male)` / `F(Female)` |
| isInstructor | Boolean | default false | 강사 여부. `PATCH /api/profile/{profileId}/instructor` 호출 시 `true`로 변경됨 |

### ID 규칙

- 길이: 8자
- 허용 문자 (총 55자):
  - 대문자 24자: `ABCDEFGHJKLMNPQRSTUVWXYZ`
  - 소문자 23자: `abcdefghjkmnpqrstuvwxyz`
  - 숫자 8자: `23456789`

---

## Lesson

### Entity: Lesson

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| title | String | NOT NULL | 레슨 제목 |
| genre | String | NOT NULL | `"S"` (Salsa) 또는 `"B"` (Bachata) |
| instructorLo | String | FK → Profile.id | 남성 강사 ID. `isInstructor=true`, `sex=M`인 프로필만 가능 |
| instructorLa | String | FK → Profile.id | 여성 강사 ID. `isInstructor=true`, `sex=F`인 프로필만 가능 |
| amount | BigDecimal | | 수강료 |
| isActive | Boolean | default true | 활성 여부 |

### Entity: LessonOption

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| startDateTime | LocalDateTime | NOT NULL | 시작 일시 |
| endDateTime | LocalDateTime | NOT NULL | 종료 일시. `startDateTime`보다 이후여야 함 |
| region | String | NOT NULL | `"GN"` (Gangnam) 또는 `"HD"` (Hongdae) |
| place | String | | 장소명 |
| placeUrl | String | | 장소 URL |

### Entity: LessonDiscount

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| type | String | NOT NULL | `"E"` (Earlybird) 또는 `"S"` (Sex) |
| condition | String | NOT NULL | type=E: `yyyy-MM-dd` 형식 날짜 / type=S: `"M"` 또는 `"F"` |
| amount | BigDecimal | | 할인 금액 |

### Entity: LessonAccount

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| bank | String | | 은행명 |
| account | String | | 계좌번호 |
| name | String | | 예금주 |

### Entity: LessonContact

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| type | String | NOT NULL | `"Y"` / `"K"` / `"W"` / `"I"` / `"L"` / `"M"` |
| account | String | | 연락처 계정 |
| name | String | | 표시명 |

### Entity: LessonNotice

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| type | String | NOT NULL | `"L"` / `"T"` / `"R"` / `"N"` / `"U"` |
| text | String | | 공지 내용 |

---

### Domain: Lesson

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | |
| title | String | |
| genre | Genre | `Salsa(S)` / `Bachata(B)` |
| instructorLo | String | Profile.id (isInstructor=true, sex=M) |
| instructorLa | String | Profile.id (isInstructor=true, sex=F) |
| options | List\<LessonOption\> | 최소 1개 이상 필수 |
| amount | BigDecimal | |
| discounts | List\<LessonDiscount\> | |
| account | LessonAccount | |
| contacts | List\<LessonContact\> | |
| isActive | Boolean | |
| notices | List\<LessonNotice\> | |

### Enum: Genre

| 값 | 코드 | 설명 |
|----|------|------|
| Salsa | S | 살사 |
| Bachata | B | 바차타 |

### Enum: Region

| 값 | 코드 | 설명 |
|----|------|------|
| Gangnam | GN | 강남 |
| Hongdae | HD | 홍대 |

### Enum: DiscountType

| 값 | 코드 | 설명 |
|----|------|------|
| Earlybird | E | 얼리버드. condition = `yyyy-MM-dd` 형식 |
| Sex | S | 성별 할인. condition = `M` 또는 `F` |

### Enum: ContactType

| 값 | 코드 | 설명 |
|----|------|------|
| Youtube | Y | 유튜브 |
| Kakaotalk | K | 카카오톡 |
| Web | W | 웹사이트 |
| Instagram | I | 인스타그램 |
| Line | L | 라인 |
| Mobile | M | 전화번호 |

### Enum: NoticeType

| 값 | 코드 | 설명 |
|----|------|------|
| Lesson | L | 레슨 관련 공지 |
| Time | T | 시간 관련 공지 |
| Region | R | 지역 관련 공지 |
| Normal | N | 일반 공지 |
| Urgent | U | 긴급 공지 |
