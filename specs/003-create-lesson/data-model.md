# Data Model: POST /api/lesson 레슨 생성

## JPA Entity

### LessonEntity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| title | String | NOT NULL | |
| genre | String | NOT NULL | `"S"` / `"B"` |
| instructorLo | String | FK → profile(id), nullable | 남성 강사 ID |
| instructorLa | String | FK → profile(id), nullable | 여성 강사 ID |
| amount | BigDecimal | nullable | |
| isActive | Boolean | NOT NULL, default true | |
| options | List\<LessonOptionEntity\> | @OneToMany cascade=ALL | |
| discounts | List\<LessonDiscountEntity\> | @OneToMany cascade=ALL | |
| account | LessonAccountEntity | @OneToOne cascade=ALL, nullable | |
| contacts | List\<LessonContactEntity\> | @OneToMany cascade=ALL | |
| notices | List\<LessonNoticeEntity\> | @OneToMany cascade=ALL | |

### LessonOptionEntity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lesson | LessonEntity | @ManyToOne @JoinColumn(name="lesson_id") | |
| startDateTime | LocalDateTime | NOT NULL | |
| endDateTime | LocalDateTime | NOT NULL | |
| region | String | NOT NULL | `"GN"` / `"HD"` |
| place | String | nullable | |
| placeUrl | String | nullable | |

### LessonDiscountEntity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lesson | LessonEntity | @ManyToOne @JoinColumn(name="lesson_id") | |
| type | String | NOT NULL | `"E"` / `"S"` |
| condition | String | NOT NULL | |
| amount | BigDecimal | nullable | |

### LessonAccountEntity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lesson | LessonEntity | @OneToOne @JoinColumn(name="lesson_id") | |
| bank | String | nullable | |
| account | String | nullable | |
| name | String | nullable | |

### LessonContactEntity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lesson | LessonEntity | @ManyToOne @JoinColumn(name="lesson_id") | |
| type | String | NOT NULL | `"Y"`/`"K"`/`"W"`/`"I"`/`"L"`/`"M"` |
| account | String | nullable | |
| name | String | nullable | |

### LessonNoticeEntity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lesson | LessonEntity | @ManyToOne @JoinColumn(name="lesson_id") | |
| type | String | NOT NULL | `"L"`/`"T"`/`"R"`/`"N"`/`"U"` |
| text | String | nullable | |

---

## Domain 객체

### Lesson

```
Lesson
├── id: Long
├── title: String
├── genre: Genre
├── instructorLo: String          (nullable)
├── instructorLa: String          (nullable)
├── options: List<LessonOption>
├── amount: BigDecimal            (nullable)
├── discounts: List<LessonDiscount>
├── account: LessonAccount        (nullable)
├── contacts: List<LessonContact>
├── isActive: Boolean
└── notices: List<LessonNotice>
```

### LessonOption

```
LessonOption
├── id: Long
├── startDateTime: LocalDateTime
├── endDateTime: LocalDateTime
├── region: Region
├── place: String                 (nullable)
└── placeUrl: String              (nullable)
```

### LessonDiscount

```
LessonDiscount
├── id: Long
├── type: DiscountType
├── condition: String
└── amount: BigDecimal            (nullable)
```

### LessonAccount

```
LessonAccount
├── id: Long
├── bank: String                  (nullable)
├── account: String               (nullable)
└── name: String                  (nullable)
```

### LessonContact

```
LessonContact
├── id: Long
├── type: ContactType
├── account: String               (nullable)
└── name: String                  (nullable)
```

### LessonNotice

```
LessonNotice
├── id: Long
├── type: NoticeType
└── text: String                  (nullable)
```

---

## Enum

| Enum | 값 (코드) |
|------|-----------|
| Genre | Salsa(S), Bachata(B) |
| Region | Gangnam(GN), Hongdae(HD) |
| DiscountType | Earlybird(E), Sex(S) |
| ContactType | Youtube(Y), Kakaotalk(K), Web(W), Instagram(I), Line(L), Mobile(M) |
| NoticeType | Lesson(L), Time(T), Region(R), Normal(N), Urgent(U) |

---

## DTO 목록

### Application Layer (port/in)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `CreateLessonAppRequest` | title, genre: Genre, instructorLo, instructorLa, options: List\<OptionAppReq\>, amount, discounts: List\<DiscountAppReq\>, account: AccountAppReq, contacts: List\<ContactAppReq\>, isActive, notices: List\<NoticeAppReq\> | Service 입력 명령 객체 |
| `CreateLessonAppRequest.OptionAppReq` | startDateTime: LocalDateTime, endDateTime: LocalDateTime, region: Region, place, placeUrl | |
| `CreateLessonAppRequest.DiscountAppReq` | type: DiscountType, condition, amount | |
| `CreateLessonAppRequest.AccountAppReq` | bank, account, name | |
| `CreateLessonAppRequest.ContactAppReq` | type: ContactType, account, name | |
| `CreateLessonAppRequest.NoticeAppReq` | type: NoticeType, text | |
| `CreateLessonAppResponse` | id: Long | Service 결과 반환 객체 |

### Web Adapter Layer (adapter/in/web)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `CreateLessonWebRequest` | title, genre, instructorLo, instructorLa, options: List\<OptionWebReq\>, amount, discounts, account: AccountWebReq, contacts, isActive, notices | HTTP 입력 DTO. Bean Validation 적용 |
| `CreateLessonWebRequest.OptionWebReq` | startDate(String), startTime(String), endDate(String), endTime(String), region, place, placeUrl | |
| `CreateLessonWebRequest.DiscountWebReq` | type, condition, amount | |
| `CreateLessonWebRequest.AccountWebReq` | bank, account, name | |
| `CreateLessonWebRequest.ContactWebReq` | type, account, name | |
| `CreateLessonWebRequest.NoticeWebReq` | type, text | |
| `CreateLessonWebResponse` | id: Long | HTTP 응답 직렬화 객체 |

---

## Port 인터페이스

### SaveLessonPort (port/out)

```
SaveLessonPort
└── save(lesson: Lesson): Lesson
```

### LoadInstructorPort (port/out)

```
LoadInstructorPort
└── findById(profileId: String): Optional<Profile>
```

- `InstructorPersistenceAdapter`가 `ProfileJpaRepository` + `ProfilePersistenceMapper`로 구현
- 반환된 Profile로 `isInstructor`, `sex` 검증은 Service에서 수행

---

## 신규 예외

### LessonValidationException (common/exception)

| 항목 | 내용 |
|------|------|
| 슈퍼클래스 | `RuntimeException` |
| 생성자 파라미터 | `List<ErrorResponse.FieldError>` |
| HTTP 상태 | 400 Bad Request |
| 용도 | appRequest 레이어 비즈니스 검증 실패 시 복수 오류 반환 |

에러 응답 예시:
```json
{
  "status": 400,
  "errors": [
    { "field": "instructorLo", "message": "남성 강사(instructorLo)에는 남성(M) 프로필만 등록 가능합니다." },
    { "field": "options[1].endDateTime", "message": "시작 일시는 종료 일시보다 이전이어야 합니다." }
  ]
}
```
