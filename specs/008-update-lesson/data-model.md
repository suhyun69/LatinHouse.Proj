# Data Model: 레슨 수정 (PUT /api/lesson/{lessonNo})

**Date**: 2026-06-03

## 신규 엔티티·도메인

이 기능은 신규 Entity 또는 Domain 객체를 추가하지 않는다.
기존 `Lesson`, `LessonOption`, `LessonDiscount`, `LessonAccount`, `LessonContact`, `LessonNotice` 도메인 객체와 JPA Entity를 그대로 사용한다.

`LessonEntity`의 모든 컬렉션 관계에 이미 `orphanRemoval = true`가 설정되어 있으므로 전체 교체 전략이 네이티브하게 지원된다.

---

## 신규 DTO 클래스

### Web Adapter Layer (`adapter/in/web/`)

#### `UpdateLessonWebRequest`

`CreateLessonWebRequest`와 동일한 필드·검증 어노테이션 구조. lessonNo는 Path Parameter로 수신하므로 요청 바디에 포함하지 않는다.

| 필드 | 타입 | 검증 | 설명 |
|------|------|------|------|
| title | String | `@NotBlank` | 레슨 제목 |
| genre | String | `@NotBlank`, `@Pattern(^[SB]$)` | `S` 또는 `B` |
| instructorLo | String | - | 남성 강사 Profile.id (null 허용) |
| instructorLa | String | - | 여성 강사 Profile.id (null 허용) |
| options | List\<OptionWebReq\> | `@NotEmpty`, `@Valid` | 수업 옵션 (1개 이상) |
| amount | BigDecimal | - | 수강료 |
| discounts | List\<DiscountWebReq\> | `@Valid` | 할인 목록 |
| account | AccountWebReq | - | 계좌 정보 |
| contacts | List\<ContactWebReq\> | `@Valid` | 연락처 목록 |
| isActive | Boolean | - | 활성 여부 |
| notices | List\<NoticeWebReq\> | `@Valid` | 공지 목록 |

Nested 클래스 (`OptionWebReq`, `DiscountWebReq`, `AccountWebReq`, `ContactWebReq`, `NoticeWebReq`)는 `CreateLessonWebRequest` 내부 클래스와 동일한 구조·검증 어노테이션을 적용한다.

#### `UpdateLessonWebResponse`

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 수정된 레슨 ID |

#### `UpdateLessonWebMapper`

| 메서드 (정적) | 설명 |
|--------------|------|
| `toAppRequest(Long lessonNo, UpdateLessonWebRequest)` → `UpdateLessonAppRequest` | 날짜·시간 String을 LocalDateTime으로 변환. enum 변환 포함 |
| `toWebResponse(UpdateLessonAppResponse)` → `UpdateLessonWebResponse` | id 매핑 |

---

### Application Layer (`application/port/in/`)

#### `UpdateLessonAppRequest`

`CreateLessonAppRequest`와 동일한 필드 + `lessonNo` 추가. Validation 어노테이션 없음.

| 필드 | 타입 | 설명 |
|------|------|------|
| lessonNo | Long | 수정 대상 레슨 ID |
| title | String | 레슨 제목 |
| genre | Genre | Enum (SALSA, BACHATA) |
| instructorLo | String | 남성 강사 Profile.id |
| instructorLa | String | 여성 강사 Profile.id |
| options | List\<CreateLessonAppRequest.OptionAppReq\> | 수업 옵션 (기존 inner class 재사용) |
| amount | BigDecimal | 수강료 |
| discounts | List\<CreateLessonAppRequest.DiscountAppReq\> | 할인 목록 (기존 inner class 재사용) |
| account | CreateLessonAppRequest.AccountAppReq | 계좌 정보 (기존 inner class 재사용) |
| contacts | List\<CreateLessonAppRequest.ContactAppReq\> | 연락처 목록 (기존 inner class 재사용) |
| isActive | Boolean | 활성 여부 |
| notices | List\<CreateLessonAppRequest.NoticeAppReq\> | 공지 목록 (기존 inner class 재사용) |

#### `UpdateLessonAppResponse`

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 수정된 레슨 ID |

#### `UpdateLessonAppMapper`

| 메서드 (정적) | 설명 |
|--------------|------|
| `toDomain(UpdateLessonAppRequest)` → `Lesson` | `id = lessonNo`를 포함한 Lesson 도메인 객체 생성. JPA가 UPDATE로 처리하도록 id 주입 필수 |
| `toAppResponse(Lesson)` → `UpdateLessonAppResponse` | id 매핑 |

---

### Application Layer (`application/port/in/`)

#### `UpdateLessonUseCase`

```java
public interface UpdateLessonUseCase {
    UpdateLessonAppResponse updateLesson(UpdateLessonAppRequest request);
}
```

---

## 재사용 클래스 (기존)

| 클래스 | 위치 | 재사용 방식 |
|--------|------|------------|
| `LoadLessonPort` | `application/port/out/` | 수정 전 레슨 존재 확인. 없으면 `LessonNotFoundException` 발생 |
| `SaveLessonPort` | `application/port/out/` | `save(Lesson)`으로 UPDATE 수행 (id 포함 시 JPA merge) |
| `LoadInstructorPort` | `application/port/out/` | 강사 유효성 검증 |
| `CreateLessonAppRequest.OptionAppReq` | `application/port/in/` | `UpdateLessonAppRequest`의 options 타입으로 직접 재사용 |
| `CreateLessonAppRequest.DiscountAppReq` | `application/port/in/` | 동일 |
| `CreateLessonAppRequest.AccountAppReq` | `application/port/in/` | 동일 |
| `CreateLessonAppRequest.ContactAppReq` | `application/port/in/` | 동일 |
| `CreateLessonAppRequest.NoticeAppReq` | `application/port/in/` | 동일 |
| `LessonValidationException` | `common/exception/` | 검증 실패 시 400 응답 트리거 |
| `LessonNotFoundException` | `common/exception/` | 레슨 미존재 시 404 응답 트리거 |

---

## 수정 흐름 (UpdateLessonService)

```
Controller → UpdateLessonWebMapper.toAppRequest(lessonNo, webReq)
           → UpdateLessonUseCase.updateLesson(appReq)
               1. loadLessonPort.loadLesson(appReq.getLessonNo())  // 404 guard
               2. validateInstructors(appReq, errors)               // 강사 유효성
               3. validateOptionDateTimes(appReq, errors)           // 옵션 시간
               4. validateDiscountConditions(appReq, errors)        // 할인 조건
               5. if (!errors.isEmpty()) throw LessonValidationException
               6. lesson = UpdateLessonAppMapper.toDomain(appReq)   // id 포함
               7. saved = saveLessonPort.save(lesson)               // JPA UPDATE
               8. return UpdateLessonAppMapper.toAppResponse(saved)
           → UpdateLessonWebMapper.toWebResponse(appResponse)
           → ResponseEntity.ok(webResponse)
```
