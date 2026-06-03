# Data Model: 랜덤 수업 생성 (POST /api/lesson/random)

**Date**: 2026-06-03

## 신규 엔티티·도메인

이 기능은 신규 Entity 또는 Domain 객체를 추가하지 않는다.
기존 `Lesson`, `LessonOption`, `LessonDiscount`, `Profile` 엔티티를 그대로 사용한다.

---

## 신규 DTO 클래스

### Application Layer

#### `CreateRandomLessonAppResponse`

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 생성된 레슨 ID |

### Web Adapter Layer

#### `CreateRandomLessonWebResponse`

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 생성된 레슨 ID |

---

## 재사용 DTO (기존)

| 클래스 | 위치 | 재사용 방식 |
|--------|------|------------|
| `CreateLessonAppRequest` | `lesson/application/port/in/` | `CreateRandomLessonAppMapper`가 랜덤 값으로 조립하여 생성 |
| `CreateLessonAppResponse` | `lesson/application/port/in/` | `CreateLessonUseCase` 반환값 → `CreateRandomLessonAppMapper`로 `CreateRandomLessonAppResponse`로 변환 |
| `GetProfilesAppResponse` | `profile/application/port/in/` | 강사 목록 조회 결과. `id`, `sex`, `isInstructor` 사용 |
| `CreateProfileAppRequest` | `profile/application/port/in/` | 신규 강사 프로필 생성 요청 |
| `CreateProfileAppResponse` | `profile/application/port/in/` | 생성된 프로필 ID 수신 |
| `SetInstructorAppRequest` | `profile/application/port/in/` | 강사 지정 요청 |

---

## 랜덤 생성 값 범위

### Genre (Enum: `com.latinhouse.api.lesson.domain.Genre`)

| 코드 | 설명 | 타이틀 후보 |
|------|------|------------|
| S | Salsa | 살사 초급반 / 살사 중급반 / 살사 상급반 |
| B | Bachata | 바차타 초급반 / 바차타 중급반 / 바차타 상급반 |

### Region (Enum: `com.latinhouse.api.lesson.domain.Region`)

| 코드 | 설명 |
|------|------|
| GN | Gangnam |
| HD | Hongdae |

### Amount 후보값

`30000`, `50000`, `80000`, `100000` (BigDecimal)

### DiscountType (Enum: `com.latinhouse.api.lesson.domain.DiscountType`)

| 코드 | condition 형식 |
|------|---------------|
| E | 가장 이른 option.startDate - 7일 (yyyy-MM-dd) |
| S | `M` 또는 `F` |

### Discount Amount 후보값

`5000`, `10000`, `15000` (BigDecimal)
