# Research: Create Profile

**Branch**: `001-create-profile` | **Date**: 2026-05-28

## 1. Swagger / OpenAPI 의존성

**Decision**: `springdoc-openapi-starter-webmvc-ui` 추가

**Rationale**: Constitution Quality Gate에서 모든 API 엔드포인트의 `@Tag`, `@Operation` Swagger 문서화를 필수로 요구한다. Spring Boot 4.0은 Spring Framework 7 기반이므로 springdoc-openapi 3.x가 필요하다.

**Dependency (build.gradle에 추가)**:
```gradle
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:3.0.0'
```

**Alternatives Considered**:
- springfox: Spring Boot 3+ 미지원, 비활성화된 프로젝트
- 직접 OpenAPI YAML 작성: 코드와 문서 동기화 불가, 유지비용 높음

---

## 2. Profile ID 생성 전략

**Decision**: `SecureRandom` + 커스텀 허용 문자셋 (`ProfileIdGenerator` 유틸리티 클래스)

**허용 문자** (`docs/data-model.md` 기준, 총 55자):
- 대문자 24자: `ABCDEFGHJKLMNPQRSTUVWXYZ` (I 제외)
- 소문자 23자: `abcdefghjkmnpqrstuvwxyz` (i, l 제외)
- 숫자 8자: `23456789` (0, 1 제외)

**Rationale**: 외부 라이브러리 추가 없이 JDK 표준 `SecureRandom`으로 구현 가능. 혼동 문자 제외(`i I 1 l 0 o O`)는 특수 비즈니스 요구사항으로 커스텀 구현이 가장 명확하다.

**Alternatives Considered**:
- Apache Commons Lang `RandomStringUtils`: 추가 의존성 필요, 특정 문자 제외 옵션 없음
- UUID 기반 truncation: 허용 문자 범위 제어 불가

---

## 3. Bean Validation 에러 핸들링

**Decision**: `@RestControllerAdvice` + `MethodArgumentNotValidException` 핸들러 (`GlobalExceptionHandler`)

**Rationale**: Spring MVC에서 `@Valid` 실패 시 `MethodArgumentNotValidException`이 발생한다. 전역 핸들러로 Constitution의 Unified Error Response 형식(`{status, errors[]}`)을 일관 적용한다.

**다중 에러 처리**: `BindingResult.getFieldErrors()`로 모든 필드 에러를 수집하여 한 번에 반환한다.

**에러 응답 예시**:
```json
{
  "status": 400,
  "errors": [
    { "field": "nickname", "message": "닉네임을 입력해 주세요." }
  ]
}
```

---

## 4. Spring Security 설정

**Decision**: `POST /api/profile` 엔드포인트를 인증 없이 허용 (`permitAll`)

**Rationale**: API 명세에 인증 요구사항이 없다. 프로필 생성은 신규 사용자가 인증 없이 수행하는 첫 단계다.

**Alternatives Considered**:
- Spring Security 전체 비활성화: 이후 다른 엔드포인트 보안 추가 시 재작업 필요 → 부분 허용 방식 선택

---

## 5. api-spec.md vs requirements.md 충돌 해소

**Issue**: `docs/requirements.md`는 성공 응답에 "Body 없음"을 기재하나, `docs/api-spec.md`는 `{"id": "..."}` 반환을 명세한다.

**Decision**: `docs/api-spec.md` 우선 적용

**Rationale**: Constitution II (Contract-First API Design)에 따라 `docs/api-spec.md`가 API 계약의 Single Source of Truth다. 성공 시 201 Created + `{"id": "Ab2Cd3Ef"}` 반환.
