# Data Model

## Profile

### Entity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK | 8자리 난수. 대/소문자+숫자 혼용, `i I 1 l 0 o O` 제외 |
| nickname | String | NOT NULL | 중복 허용 |
| sex | String | | `"M"` 또는 `"F"` |
| isInstructor | Boolean | default false | 강사 여부. `PATCH /api/profile/{profileId}/instructor` 호출 시 `true`로 변경됨 |

### Domain

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK | 8자리 난수. 대/소문자+숫자 혼용, `i I 1 l 0 o O` 제외 |
| nickname | String | Not null | 중복 허용 |
| sex | Sex | Enum | `M(Male)` / `F(Female)` |
| isInstructor | Boolean | default false | 강사 여부. `PATCH /api/profile/{profileId}/instructor` 호출 시 `true`로 변경됨 |

### ID 규칙

- 길이: 8자
- 허용 문자 (총 55자):
  - 대문자 24자: `ABCDEFGHJKLMNPQRSTUVWXYZ`
  - 소문자 23자: `abcdefghjkmnpqrstuvwxyz`
  - 숫자 8자: `23456789`

---

## Lesson

### Entity: Lesson

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| title | String | NOT NULL | 레슨 제목 |
| genre | String | NOT NULL | `"S"` (Salsa) 또는 `"B"` (Bachata) |
| instructorLo | String | FK → Profile.id | 남성 강사 ID. `isInstructor=true`, `sex=M`인 프로필만 가능 |
| instructorLa | String | FK → Profile.id | 여성 강사 ID. `isInstructor=true`, `sex=F`인 프로필만 가능 |
| amount | BigDecimal | | 수강료 |
| isActive | Boolean | default true | 활성 여부 |

### Entity: LessonOption

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| startDateTime | LocalDateTime | NOT NULL | 시작 일시 |
| endDateTime | LocalDateTime | NOT NULL | 종료 일시. `startDateTime`보다 이후여야 함 |
| region | String | NOT NULL | `"GN"` (Gangnam) 또는 `"HD"` (Hongdae) |
| place | String | | 장소명 |
| placeUrl | String | | 장소 URL |

### Entity: LessonDiscount

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| type | String | NOT NULL | `"E"` (Earlybird) 또는 `"S"` (Sex) |
| condition | String | NOT NULL | type=E: `yyyy-MM-dd` 형식 날짜 / type=S: `"M"` 또는 `"F"` |
| amount | BigDecimal | | 할인 금액 |

### Entity: LessonAccount

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| bank | String | | 은행명 |
| account | String | | 계좌번호 |
| name | String | | 예금주 |

### Entity: LessonContact

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| type | String | NOT NULL | `"Y"` / `"K"` / `"W"` / `"I"` / `"L"` / `"M"` |
| account | String | | 연락처 계정 |
| name | String | | 표시명 |

### Entity: LessonNotice

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| lessonNo | Long | FK → Lesson.id | |
| type | String | NOT NULL | `"L"` / `"T"` / `"R"` / `"N"` / `"U"` |
| text | String | | 공지 내용 |

---

### Domain: Lesson

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | |
| title | String | |
| genre | Genre | `Salsa(S)` / `Bachata(B)` |
| instructorLo | String | Profile.id (isInstructor=true, sex=M) |
| instructorLa | String | Profile.id (isInstructor=true, sex=F) |
| options | List\<LessonOption\> | 최소 1개 이상 필수 |
| amount | BigDecimal | |
| discounts | List\<LessonDiscount\> | |
| account | LessonAccount | |
| contacts | List\<LessonContact\> | |
| isActive | Boolean | |
| notices | List\<LessonNotice\> | |

### Enum: Genre

| 값 | 코드 | 설명 |
|----|------|------|
| Salsa | S | 살사 |
| Bachata | B | 바차타 |

### Enum: Region

| 값 | 코드 | 설명 |
|----|------|------|
| Gangnam | GN | 강남 |
| Hongdae | HD | 홍대 |

### Enum: DiscountType

| 값 | 코드 | 설명 |
|----|------|------|
| Earlybird | E | 얼리버드. condition = `yyyy-MM-dd` 형식 |
| Sex | S | 성별 할인. condition = `M` 또는 `F` |

### Enum: ContactType

| 값 | 코드 | 설명 |
|----|------|------|
| Youtube | Y | 유튜브 |
| Kakaotalk | K | 카카오톡 |
| Web | W | 웹사이트 |
| Instagram | I | 인스타그램 |
| Line | L | 라인 |
| Mobile | M | 전화번호 |

### Enum: NoticeType

| 값 | 코드 | 설명 |
|----|------|------|
| Lesson | L | 레슨 관련 공지 |
| Time | T | 시간 관련 공지 |
| Region | R | 지역 관련 공지 |
| Normal | N | 일반 공지 |
| Urgent | U | 긴급 공지 |

---

## Order

### Entity: Order

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK, UUID | 주문 ID |
| lessonNo | Long | FK → Lesson.id, NOT NULL | 레슨 ID |
| lessonOptionNo | Long | FK → LessonOption.id, NOT NULL | 수업 옵션 ID |
| buyer | String | FK → Profile.id, NOT NULL | 구매자 프로필 ID |
| price | BigDecimal | NOT NULL | 총 주문 금액. `payment.amount + sum(discounts.amount)` |
| paymentId | Long | FK → Payment.id | 연결된 결제 ID. 결제 완료 후 설정 |
| status | String | NOT NULL | 주문 상태. `OrderStatus` 참조 |

### Entity: OrderDiscount

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| orderId | String | FK → Order.id, NOT NULL | 주문 ID |
| discountType | String | NOT NULL | 할인 유형. `OrderDiscountType` 참조 |
| discountId | Long | NOT NULL | 할인 원본 ID (LessonDiscount.id 또는 쿠폰 ID) |
| amount | BigDecimal | NOT NULL | 할인 금액 |

### Entity: Payment

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| orderId | String | FK → Order.id, NOT NULL | 주문 ID |
| payType | String | NOT NULL | 결제 수단 (카드, 계좌이체 등) |
| amount | BigDecimal | NOT NULL | 실제 결제 금액 |

---

### Domain: Order

| 필드 | 타입 | 설명 |
|------|------|------|
| id | String | UUID |
| lessonNo | Long | |
| lessonOptionNo | Long | |
| buyer | String | Profile.id |
| price | BigDecimal | `payment.amount + sum(discounts.amount)` |
| paymentId | Long | 결제 완료 후 설정 |
| discounts | List\<OrderDiscount\> | 적용된 할인 내역 |
| status | OrderStatus | 주문 상태 |

### Domain: OrderDiscount

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | |
| orderId | String | |
| discountType | OrderDiscountType | |
| discountId | Long | |
| amount | BigDecimal | |

### Domain: Payment

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | |
| orderId | String | |
| payType | String | |
| amount | BigDecimal | |

### Enum: OrderStatus

| 값 | 설명 |
|----|------|
| PAYMENT_PENDING | 결제 대기 중. 주문 생성 직후 초기 상태 |
| PAYMENT_COMPLETED | 결제 완료 |
| APPROVED | 승인 완료 |
| CANCELED | 취소됨 |

### Enum: OrderDiscountType

| 값 | 설명 |
|----|------|
| LESSON | 레슨 할인 (LessonDiscount 기반) |
| COUPON | 쿠폰 할인 |

---

### 할인 적용 흐름 (주문 생성 시)

주문 생성 시 `Lesson.discounts`를 순회하여 아래 규칙에 따라 `Order.discounts`를 구성한다.

| LessonDiscount.type | 적용 조건 | 적용 결과 |
|---------------------|-----------|-----------|
| SEX (`S`) | `LessonDiscount.condition == Profile.sex` | 일치하면 1건 추가, 불일치 시 제외 |
| EARLYBIRD (`E`) | `condition`(yyyy-MM-dd) >= 주문 생성 시점(`now`) 인 항목만 후보. 후보 중 condition이 가장 이른 1건 | 만료된(`condition < now`) 항목 제외. 후보 없으면 미적용 |

적용된 항목은 `OrderDiscount`로 저장되며 다음 값을 가진다.
- `discountType` = `LESSON`
- `discountId` = `LessonDiscount.id`
- `amount` = `LessonDiscount.amount`

---

### DTO: GET /api/orders (주문 목록 조회)

**신규 클래스**

| 클래스 | 레이어 | 설명 |
|--------|--------|------|
| `GetOrdersWebResponse` | Web Adapter (in) | 조회 응답 단건 DTO |
| `GetOrdersWebMapper` | Web Adapter (in) | AppResponse → WebResponse 변환 |
| `GetOrdersAppRequest` | Application Port In | buyer(String), lessonNo(Long) 필드 |
| `GetOrdersAppResponse` | Application Port In | 조회 결과 단건 DTO |
| `GetOrdersUseCase` | Application Port In | 목록 조회 유스케이스 인터페이스 |
| `LoadOrderPort` | Application Port Out | DB 조회 인터페이스 |

**GetOrdersWebResponse / GetOrdersAppResponse 필드**

| 필드 | 타입 | 설명 |
|------|------|------|
| orderId | String | 주문 ID (UUID) |
| lessonNo | Long | 레슨 번호 |
| lessonOptionNo | Long | 레슨 옵션 번호 |
| price | BigDecimal | 주문 금액 |
| status | String | 주문 상태 (`OrderStatus` 값) |
| discounts | List\<OrderDiscountInfo\> | 적용된 할인 목록 |

**OrderDiscountInfo (중첩 DTO)**

| 필드 | 타입 | 설명 |
|------|------|------|
| discountType | String | 할인 유형 (`OrderDiscountType` 값) |
| discountId | Long | `LessonDiscount.id` |
| amount | BigDecimal | 할인 금액 |

---

## Coupon

### Domain: CouponTemplate

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | auto-increment PK |
| title | String | 쿠폰 템플릿 이름 |
| type | CouponTemplateType | 쿠폰 유형 |
| target | Long | 적용 대상 ID (type=LESSON이면 lessonNo) |
| amount | BigDecimal | 할인 금액 |

### Domain: Coupon

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | auto-increment PK |
| templateId | Long | CouponTemplate.id |
| owner | String | 쿠폰 소유자 Profile.id (발행 직후 null) |
| status | CouponStatus | 쿠폰 상태. 기본값: `AVAILABLE` |

### Entity: coupon_templates

| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| title | VARCHAR | NOT NULL | |
| type | VARCHAR | NOT NULL | `CouponTemplateType` 값 |
| target | BIGINT | NOT NULL | |
| amount | DECIMAL | NOT NULL | |

### Entity: coupons

| 컬럼 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | |
| template_id | BIGINT | NOT NULL | FK → coupon_templates.id |
| owner | VARCHAR | NULL | |
| status | VARCHAR | NOT NULL | 기본값 `AVAILABLE` |

### Enum: CouponTemplateType

| 값 | 설명 |
|----|------|
| LESSON | 특정 레슨에 적용되는 쿠폰 |

### Enum: CouponStatus

| 값 | 설명 |
|----|------|
| AVAILABLE | 사용 가능 상태 (초기값) |
| USED | 사용 완료 상태 |

---

### DTO: POST /api/coupon/template (쿠폰 템플릿 생성)

**신규 클래스**

| 클래스 | 레이어 | 설명 |
|--------|--------|------|
| `CreateCouponTemplateWebRequest` | Web Adapter (in) | 생성 요청 DTO |
| `CreateCouponTemplateWebResponse` | Web Adapter (in) | 생성 응답 DTO (`couponTemplateId: String`) |
| `CouponWebMapper` | Web Adapter (in) | Web ↔ App 변환 |
| `CreateCouponTemplateAppRequest` | Application Port In | title, type, target, amount |
| `CreateCouponTemplateAppResponse` | Application Port In | couponTemplateId: Long |
| `CreateCouponTemplateUseCase` | Application Port In | 템플릿 생성 유스케이스 인터페이스 |
| `SaveCouponTemplatePort` | Application Port Out | DB 저장 인터페이스 |

### DTO: POST /api/coupon (쿠폰 일괄 발행)

**신규 클래스**

| 클래스 | 레이어 | 설명 |
|--------|--------|------|
| `CreateCouponWebRequest` | Web Adapter (in) | 발행 요청 DTO |
| `CreateCouponAppRequest` | Application Port In | templateId: Long, count: Integer |
| `CreateCouponUseCase` | Application Port In | 쿠폰 발행 유스케이스 인터페이스 |
| `LoadCouponTemplatePort` | Application Port Out | 템플릿 조회 인터페이스 |
| `SaveCouponPort` | Application Port Out | 쿠폰 저장 인터페이스 |
