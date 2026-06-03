# Research: 주문 생성 시 레슨 할인 자동 적용

## 결정사항

### Decision 1: 할인 선별 위치 — Service 계층
- **Decision**: 할인 선별 로직은 `CreateOrderService`에 구현한다.
- **Rationale**: 할인 적용 여부는 도메인 비즈니스 규칙이며, Port/Adapter 계층이 아닌 Application 서비스에 위치해야 Hexagonal Architecture를 준수한다.
- **Alternatives considered**: Domain 객체(Order) 내부에서 처리 → 그러나 Profile 정보를 Domain 계층으로 전달하면 Domain이 외부 의존을 갖게 되어 헌법 위반. Service에서 처리하는 것이 적합하다.

### Decision 2: EARLYBIRD 정렬 기준 — 문자열 정렬
- **Decision**: `LessonDiscount.condition`은 `yyyy-MM-dd` 형식이 보장되므로 문자열 자연 정렬(lexicographic)로 가장 이른 날짜를 선택한다.
- **Rationale**: `yyyy-MM-dd`는 사전순 = 날짜순이 보장되므로 LocalDate 파싱 없이 `String::compareTo`로 min 선택 가능하다. 코드 단순화.
- **Alternatives considered**: LocalDate.parse()로 파싱하여 정렬 → 가능하나 불필요한 파싱 비용, 형식 위반 시 예외 발생 위험.

### Decision 3: Profile.sex null 처리
- **Decision**: `Profile.sex`가 null이면 SEX 할인은 적용하지 않는다.
- **Rationale**: 명세에 명시되지 않은 경우 보수적으로 처리. null condition 비교 시 NPE 방지.
- **Alternatives considered**: null을 특정 성별로 기본값 처리 → 명세 근거 없음, 제외.

### Decision 4: 동일 condition EARLYBIRD 복수 존재 시
- **Decision**: condition이 동일한 EARLYBIRD가 복수 있으면 stream min 처리 시 첫 번째 encountered 요소가 선택된다 (리스트 순서 의존).
- **Rationale**: 명세에 tie-breaking 규칙이 없으므로 별도 정렬 없이 스트림 first 사용.
- **Alternatives considered**: id 오름차순 정렬 후 첫 번째 → 더 예측 가능하나 명세 요건 초과.

### Decision 5: 신규 파일 없음 — 기존 Service만 수정
- **Decision**: 이 피처는 `CreateOrderService.java`와 `CreateOrderServiceTest.java` 수정만 필요하다. 새로운 Port, Entity, Controller, WebRequest/Response는 불필요하다.
- **Rationale**: 할인 선별은 이미 로드된 `Lesson.discounts`와 `Profile.sex`를 사용하는 순수 계산 로직이다. `FindProfilePort`는 이미 존재하며 Profile 전체 객체를 반환한다.
