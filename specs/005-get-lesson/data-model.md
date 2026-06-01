# Data Model: 레슨 단건 조회

> 신규 엔티티 없음. 기존 Lesson 도메인 엔티티를 그대로 사용한다.
> 전체 엔티티 정의는 `docs/data-model.md` 참조.

## 조회 대상 엔티티

| 엔티티 | JPA Entity | Domain | 비고 |
|--------|-----------|--------|------|
| Lesson | LessonEntity | Lesson | 루트. id(Long) PK |
| LessonOption | LessonOptionEntity | LessonOption | lesson_id FK. 1개 이상 |
| LessonDiscount | LessonDiscountEntity | LessonDiscount | lesson_id FK. 없으면 빈 배열 |
| LessonAccount | LessonAccountEntity | LessonAccount | account_id FK (OneToOne). 없으면 null |
| LessonContact | LessonContactEntity | LessonContact | lesson_id FK. 없으면 빈 배열 |
| LessonNotice | LessonNoticeEntity | LessonNotice | lesson_id FK. 없으면 빈 배열 |

## 응답 DTO 구조

### GetLessonAppResponse (application/port/in/)

```
GetLessonAppResponse
├── id: Long
├── title: String
├── genre: Genre
├── instructorLo: String (nullable)
├── instructorLa: String (nullable)
├── options: List<OptionResponse>
│   ├── id: Long
│   ├── startDateTime: LocalDateTime
│   ├── endDateTime: LocalDateTime
│   ├── region: Region
│   ├── place: String (nullable)
│   └── placeUrl: String (nullable)
├── amount: BigDecimal (nullable)
├── discounts: List<DiscountResponse>
│   ├── id: Long
│   ├── type: DiscountType
│   ├── condition: String
│   └── amount: BigDecimal (nullable)
├── account: AccountResponse (nullable)
│   ├── id: Long
│   ├── bank: String (nullable)
│   ├── account: String (nullable)
│   └── name: String (nullable)
├── contacts: List<ContactResponse>
│   ├── id: Long
│   ├── type: ContactType
│   ├── account: String (nullable)
│   └── name: String (nullable)
├── isActive: boolean
└── notices: List<NoticeResponse>
    ├── id: Long
    ├── type: NoticeType
    └── text: String (nullable)
```

### GetLessonWebResponse (adapter/in/web/)

```
GetLessonWebResponse
├── id: Long
├── title: String
├── genre: String          ← Genre.getCode()
├── instructorLo: String (nullable)
├── instructorLa: String (nullable)
├── options: List<OptionResponse>
│   ├── id: Long
│   ├── startDate: String  ← LocalDateTime.toLocalDate() → yyyy-MM-dd
│   ├── startTime: String  ← LocalDateTime.toLocalTime() → HH:mm
│   ├── endDate: String
│   ├── endTime: String
│   ├── region: String     ← Region.getCode()
│   ├── place: String (nullable)
│   └── placeUrl: String (nullable)
├── amount: BigDecimal (nullable)
├── discounts: List<DiscountResponse>
│   ├── id: Long
│   ├── type: String       ← DiscountType.getCode()
│   ├── condition: String
│   └── amount: BigDecimal (nullable)
├── account: AccountResponse (nullable)
│   ├── id: Long
│   ├── bank: String (nullable)
│   ├── account: String (nullable)
│   └── name: String (nullable)
├── contacts: List<ContactResponse>
│   ├── id: Long
│   ├── type: String       ← ContactType.getCode()
│   ├── account: String (nullable)
│   └── name: String (nullable)
├── isActive: boolean
└── notices: List<NoticeResponse>
    ├── id: Long
    ├── type: String       ← NoticeType.getCode()
    └── text: String (nullable)
```

## 변환 흐름

```
LessonEntity (JPA)
    → LessonPersistenceMapper.toDomain()    [기존 재사용]
    → Lesson (Domain)
    → GetLessonAppMapper.toAppResponse()    [신규]
    → GetLessonAppResponse
    → GetLessonWebMapper.toWebResponse()    [신규]
    → GetLessonWebResponse (HTTP 응답)
```
