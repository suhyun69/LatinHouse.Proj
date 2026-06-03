# Data Model: GET /api/lessons 레슨 목록 조회

## 기존 Entity (변경 없음)

`LessonEntity`, `LessonOptionEntity`, `LessonDiscountEntity` — 003-create-lesson 참고.

---

## 신규 DTO 목록

### Application Layer (`application/port/in/`)

#### `GetLessonsAppResponse`

레슨 1개 옵션 = 1개 항목. 목록 반환형: `List<GetLessonsAppResponse>`

| 필드 | 타입 | 설명 |
|------|------|------|
| optionId | Long | 수업 옵션 ID |
| lessonNo | Long | 레슨 ID |
| instructorLo | String | 남성 강사 Profile.id (nullable) |
| instructorLa | String | 여성 강사 Profile.id (nullable) |
| title | String | 레슨 제목 |
| genre | String | 장르 코드 (`S`/`B`) |
| startDate | String | `yyyy-MM-dd` |
| startTime | String | `HH:mm` |
| endDate | String | `yyyy-MM-dd` |
| endTime | String | `HH:mm` |
| region | String | 지역 코드 (`GN`/`HD`) |
| price | BigDecimal | 수강료 (nullable) |
| discountCondition | String | 얼리버드 할인 마감일 `yyyy-MM-dd` (nullable) |
| discountAmount | BigDecimal | 얼리버드 할인 금액 (nullable) |
| status | String | `INACTIVE` / `PENDING` / `IN_PROGRESS` / `DONE` |

#### `GetLessonsAppRequest` (필터)

| 필드 | 타입 | 설명 |
|------|------|------|
| region | Region | 지역 필터 (nullable) |
| instructor | String | 강사 Profile.id 필터 (nullable) |
| genre | Genre | 장르 필터 (nullable) |

---

### Web Adapter Layer (`adapter/in/web/`)

#### `GetLessonsWebResponse`

`GetLessonsAppResponse`와 필드 동일 (String 직렬화). Controller 반환: `ResponseEntity<List<GetLessonsWebResponse>>`

---

## Port 인터페이스

### `LoadLessonsPort` (신규, `application/port/out/`)

```
LoadLessonsPort
└── loadLessons(region: Region, instructor: String, genre: Genre): List<Lesson>
    // null 파라미터는 해당 필터를 적용하지 않음
```

`LessonPersistenceAdapter`가 JPA Specification으로 구현한다.

---

## UseCase 인터페이스

### `GetLessonsUseCase` (신규, `application/port/in/`)

```
GetLessonsUseCase
└── getLessons(appRequest: GetLessonsAppRequest): List<GetLessonsAppResponse>
```

---

## JPA Specification 필터 규칙

| 필터 | Specification 조건 |
|------|--------------------|
| genre | `lessonEntity.genre = :genre` |
| instructor | `lessonEntity.instructorLo = :instructor OR lessonEntity.instructorLa = :instructor` |
| region | `JOIN lessonEntity.options o WHERE o.region = :region` + `DISTINCT` 적용 |

복수 조건은 `Specification.and()`로 결합.

---

## status 계산 로직 (Java)

```java
// GetLessonsAppMapper 내
private static String calcStatus(boolean isActive, LocalDateTime start, LocalDateTime end) {
    if (!isActive) return "INACTIVE";
    LocalDateTime now = LocalDateTime.now();
    if (now.isBefore(start)) return "PENDING";
    if (!now.isAfter(end)) return "IN_PROGRESS";
    return "DONE";
}
```

## discount 계산 로직 (Java)

```java
// GetLessonsAppMapper 내
private static LessonDiscount findEarlybird(List<LessonDiscount> discounts) {
    LocalDate today = LocalDate.now();
    return discounts.stream()
        .filter(d -> d.getType() == DiscountType.EARLYBIRD)
        .filter(d -> LocalDate.parse(d.getCondition()).compareTo(today) >= 0)
        .min(Comparator.comparing(LessonDiscount::getCondition))
        .orElse(null);
}
```
