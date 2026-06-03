# Research: 레슨 수정 (PUT /api/lesson/{lessonNo})

**Date**: 2026-06-03

## 1. 교체(Replace-All) 전략 — JPA orphanRemoval 활용

**Decision**: 컬렉션 필드(options, discounts, contacts, notices)를 "전체 교체" 방식으로 처리한다.

**Rationale**:
- `LessonEntity`의 모든 OneToMany 관계에 `orphanRemoval = true`가 이미 선언되어 있다.
- JPA `save(entity)` 호출 시, 기존 컬렉션을 clear 후 새 항목을 추가하면 orphanRemoval이 자동으로 기존 행을 DELETE한다.
- 별도의 `DELETE` Port 또는 쿼리가 필요하지 않다.

**Alternatives Considered**:
- Merge 전략 (ID 기반 개별 수정): 복잡도가 높고 이 기능에 불필요하다.
- 직접 DELETE 쿼리: JPA 영속성 컨텍스트 동기화 문제 발생 가능, 사용 안 함.

---

## 2. 기존 Port 재사용

**Decision**: 신규 Port 인터페이스를 추가하지 않는다. `LoadLessonPort`, `SaveLessonPort`, `LoadInstructorPort`를 그대로 사용한다.

**Rationale**:
- `LoadLessonPort.loadLesson(Long)`: 이미 404 예외 처리 포함. 수정 전 존재 확인에 그대로 사용.
- `SaveLessonPort.save(Lesson)`: JPA `save()` = upsert. Lesson 도메인 객체에 `id`를 포함하면 UPDATE로 동작.
- `LoadInstructorPort`: 강사 유효성 검증에 그대로 사용.

**Alternatives Considered**:
- `UpdateLessonPort` 별도 정의: 불필요한 추상화. 기존 `SaveLessonPort`가 upsert를 지원하므로 생략.

---

## 3. 검증 로직 재사용

**Decision**: `UpdateLessonService`는 `CreateLessonService`의 검증 메서드(validateInstructors, validateOptionDateTimes, validateDiscountConditions)와 동일한 로직을 적용한다.

**Rationale**:
- 비즈니스 규칙은 생성과 수정 시 동일하다 (api-spec.md 명세 기준).
- 코드 중복을 최소화하기 위해 공통 유틸 추출보다 동일 패턴 적용을 선택한다 (현재 코드베이스가 서비스별 자기완결성을 유지하는 패턴).

---

## 4. UpdateLessonAppRequest 설계

**Decision**: `CreateLessonAppRequest`의 모든 필드를 그대로 갖고 `lessonNo` 필드를 추가한다. Nested 클래스(`OptionAppReq` 등)는 `CreateLessonAppRequest`의 inner class를 직접 재사용한다.

**Rationale**:
- AppMapper에서 `toDomain()` 호출 시 `id = lessonNo`를 Lesson에 주입해야 JPA가 UPDATE로 처리한다.
- Nested DTO는 동일 구조이므로 `CreateLessonAppRequest.OptionAppReq` 등을 타입으로 그대로 사용한다.

---

## 5. 테스트 전략

**Decision**: 두 계층 테스트를 작성한다:
1. `UpdateLessonControllerTest` (`@WebMvcTest`) — HTTP 레이어 검증 (200/400/404)
2. `UpdateLessonServiceTest` (단위 테스트) — 검증 로직 및 Port 호출 순서 검증

**Rationale**: 기존 `LessonControllerTest`, `CreateLessonServiceTest`와 동일한 패턴을 따른다.
