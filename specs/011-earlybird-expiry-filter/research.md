# Research: 주문 생성 시 EARLYBIRD 할인 만료일 필터링

## 결정사항

### Decision 1: 날짜 비교 기준 — LocalDate 단위, 당일 포함
- **Decision**: `LessonDiscount.condition`을 `LocalDate.parse(condition)`으로 파싱하고 `LocalDate.now()`와 비교. `conditionDate.isBefore(LocalDate.now())`인 경우 제외. 즉, `conditionDate >= today`인 경우만 유효 후보로 인정.
- **Rationale**: 명세에 "condition < now인 항목은 제외"로 명시. isBefore()는 strictly less-than이므로 오늘 날짜(`conditionDate.equals(today)`)는 유효로 처리된다. 시각(time) 단위가 아닌 날짜(date) 단위로 처리하는 것이 명세 의도에 부합한다.
- **Alternatives considered**: `!conditionDate.isAfter(today)` 사용 → conditionDate <= today이므로 오늘을 만료로 처리하게 됨. 명세와 다르므로 채택하지 않음.

### Decision 2: 비교 메서드 — LocalDate.parse() 사용
- **Decision**: `condition`은 `yyyy-MM-dd` 형식이 보장되므로 `LocalDate.parse(condition)`을 직접 사용한다.
- **Rationale**: `DateTimeFormatter` 명시 없이도 ISO_LOCAL_DATE 기본 포맷으로 파싱 가능. 코드 단순화.
- **Alternatives considered**: `LocalDate.parse(condition, DateTimeFormatter.ISO_DATE)` → 동일 결과, 불필요한 verbose.

### Decision 3: 변경 범위 — resolveDiscounts() 내 EARLYBIRD 스트림 필터 추가
- **Decision**: `CreateOrderService.resolveDiscounts()` 메서드의 EARLYBIRD 선별 스트림에 `.filter(d -> !LocalDate.parse(d.getCondition()).isBefore(LocalDate.now()))` 조건을 추가한다.
- **Rationale**: 최소 변경으로 기존 로직(SEX 할인, 저장 로직 등)에 영향 없이 EARLYBIRD 필터링만 교체할 수 있다.
- **Alternatives considered**: 별도 메서드로 분리 → 이 규모에서는 과도한 추상화.

### Decision 4: 테스트에서 "now" 제어
- **Decision**: 테스트에서 `LocalDate.now()`를 직접 호출하는 대신, 현재 날짜 대비 미래/과거 날짜를 동적으로 계산하여 fixture를 생성한다 (`LocalDate.now().plusDays(30)` 등). Clock 주입은 이 규모에서 불필요하다.
- **Rationale**: 테스트가 특정 날짜에 의존하지 않아 언제 실행해도 통과한다. Clock 주입은 구현 복잡도가 높다.
- **Alternatives considered**: `@MockBean Clock` 주입 → 헌법에서 과도한 인프라 복잡도는 지양. 날짜 오프셋 방식이 적합.
