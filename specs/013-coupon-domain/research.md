# Research: Coupon 도메인 생성

**Feature**: 013-coupon-domain
**Date**: 2026-06-07

## 기술 스택 확정

### Decision: 기존 프로젝트 기술 스택 그대로 사용
- **Rationale**: 신규 도메인이므로 기존 Order/Lesson 도메인과 동일한 기술 스택을 사용한다.
- **Alternatives considered**: 없음 — 신규 도메인은 기존 아키텍처 확장으로 처리.

| 항목 | 값 |
|------|-----|
| Language | Java (Spring Boot 4.x) |
| ORM | Spring Data JPA (`@GeneratedValue(strategy = AUTO)`) |
| ID 전략 | `Long` + DB auto-increment (IDENTITY 전략) |
| Validation | Bean Validation (`@NotNull`, `@Min`) |
| Architecture | Hexagonal (Ports & Adapters) |
| Error Handling | `GlobalExceptionHandler` + `CouponTemplateNotFoundException` 신규 추가 |

---

## 패턴 분석

### Decision: Order 도메인 패턴 동일 적용
- **Rationale**: 프로젝트에 이미 확립된 두 도메인(Order, Lesson)의 패턴이 명확하다. 동일한 패턴을 따라 일관성을 유지한다.
- **Alternatives considered**: 없음.

**참고 패턴 파일**:
- Entity: `OrderEntity.java` — `@NoArgsConstructor(access = AccessLevel.PROTECTED)` + `@Builder`
- Mapper: `OrderPersistenceMapper.java` — private 생성자, static 메서드
- Adapter: `OrderPersistenceAdapter.java` — `@Repository`, `@RequiredArgsConstructor`
- Service: `CreateOrderService.java` — `@Service`, `@Transactional`
- Exception: `LessonNotFoundException.java` + `GlobalExceptionHandler` 핸들러 추가

---

## ID 전략

### Decision: `Long` auto-increment (JPA `@GeneratedValue`)
- **Rationale**: `CouponTemplate`과 `Coupon` 모두 `Long` PK를 사용한다. Order와 달리 UUID가 아닌 순번 ID를 사용한다 (spec 정의 기준).
- **Alternatives considered**: UUID — 사용자 요구사항이 Long이므로 제외.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

---

## 에러 처리 전략

### Decision: `CouponTemplateNotFoundException` 신규 클래스 추가 + GlobalExceptionHandler 등록
- **Rationale**: 기존 `LessonNotFoundException`, `ProfileNotFoundException`과 동일한 패턴으로 404 에러를 처리한다.
- **Alternatives considered**: 없음.

---

## count 검증 전략

### Decision: `@Min(1)` Bean Validation 사용
- **Rationale**: `CreateCouponWebRequest.count`에 `@NotNull @Min(1)`을 적용하면 Bean Validation이 400을 자동 반환한다. 별도 서비스 레이어 검증 불필요.
- **Alternatives considered**: 서비스 레이어 수동 검증 — 중복이므로 제외.

---

## 쿠폰 발행 방식

### Decision: count 루프로 개별 Coupon 생성 후 `saveAll()`
- **Rationale**: 단순 반복 저장. Bulk insert 최적화는 현재 스코프 밖.
- **Alternatives considered**: Native SQL batch insert — 과도한 최적화, 제외.
