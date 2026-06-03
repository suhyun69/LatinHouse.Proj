# Research: 랜덤 수업 생성 (POST /api/lesson/random)

**Date**: 2026-06-03

## 결정 사항

### 1. 랜덤 생성 로직 위치

**Decision**: Application Service (`CreateRandomLessonService`)에서 담당

**Rationale**: Domain 레이어에 랜덤 생성 책임을 부여하면 `java.util.Random`이라는 외부 의존이 도메인에 침투한다. Constitution I에 따라 Domain은 외부 의존을 가질 수 없으므로, Application Service 계층에서 랜덤 값을 생성하여 `CreateLessonAppRequest`를 조립한다.

**Alternatives considered**:
- Domain에서 팩토리 메서드로 생성 → Domain에 Random 의존이 생겨 Constitution I 위반
- Controller에서 생성 → Adapter가 비즈니스 로직을 보유하게 되어 Hexagonal Architecture 위반

---

### 2. 기존 UseCase 재사용 방식

**Decision**: `CreateRandomLessonService`가 `CreateLessonUseCase`, `GetProfilesUseCase`, `CreateProfileUseCase`, `SetInstructorUseCase`를 직접 주입받아 호출

**Rationale**: 기존 유스케이스들이 완전한 비즈니스 규칙(강사 검증, 레슨 저장 등)을 이미 캡슐화하고 있다. 중복 구현 없이 조합(Orchestration) 패턴으로 재사용하면 유지보수 비용이 최소화된다.

**Alternatives considered**:
- 새로운 Port(out)를 직접 호출 → 기존 비즈니스 규칙을 중복 구현해야 함
- 별도 도메인 서비스 생성 → 오버엔지니어링. 단순 조합 로직에 불필요한 추상화 계층 추가

---

### 3. 강사가 없을 때 자동 생성 방식

**Decision**: `CreateProfileUseCase` + `SetInstructorUseCase`를 순서대로 호출하여 신규 프로필 생성 후 강사 지정

**Rationale**: 기존 두 유스케이스의 조합으로 완전한 강사 생성 흐름을 재현할 수 있다. `POST /api/profile` → `PATCH /api/profile/{id}/instructor` 흐름과 동일하다.

**Nickname 형식**: `강사_M_<4자리 랜덤 숫자>` / `강사_F_<4자리 랜덤 숫자>`

---

### 4. 오류 처리 전략

**Decision**: 강사 생성/지정 실패 시 `RuntimeException` throw → `GlobalExceptionHandler`가 500으로 처리

**Rationale**: 이 엔드포인트는 외부 입력 검증이 없으므로 400 에러가 발생할 수 없다. 내부 흐름 실패만 존재하며, 이는 500 Internal Server Error로 처리하는 것이 spec 정의에 부합한다.

---

### 5. instructorLo / instructorLa 할당 전략

**Decision**: 남성·여성 강사 모두 항상 할당 (둘 다 null인 경우 없음)

**Rationale**: Spec FR-006에서 "최소 1명 이상" 조건을 만족하기 위해 가장 단순한 구현으로 남성·여성 각각 할당한다. 후보가 없으면 자동 생성하므로 항상 둘 다 할당 가능하다.

---

### 6. Java 난수 생성

**Decision**: `ThreadLocalRandom.current()` 사용

**Rationale**: 멀티스레드 환경에서 `java.util.Random`보다 성능이 우수하고 Spring Boot 서버 환경에 적합하다.
