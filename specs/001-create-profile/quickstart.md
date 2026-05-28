# Quickstart: Create Profile

## Prerequisites

- Java 25
- Gradle Wrapper (`./gradlew`)

## Run the Application

```bash
cd Latinhouse.Be
./gradlew bootRun
```

## Test the Endpoint

### 정상 생성

```bash
curl -X POST http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -d '{"nickname": "TestUser", "sex": "M"}'
```

기대 응답: `201 Created`
```json
{ "id": "Ab2Cd3Ef" }
```

### nickname 누락

```bash
curl -X POST http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -d '{"sex": "M"}'
```

기대 응답: `400 Bad Request`
```json
{ "status": 400, "errors": [{ "field": "nickname", "message": "닉네임을 입력해 주세요." }] }
```

### sex 누락

```bash
curl -X POST http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -d '{"nickname": "TestUser"}'
```

기대 응답: `400 Bad Request`
```json
{ "status": 400, "errors": [{ "field": "sex", "message": "성별을 입력해 주세요." }] }
```

### 유효하지 않은 sex 값

```bash
curl -X POST http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -d '{"nickname": "TestUser", "sex": "X"}'
```

기대 응답: `400 Bad Request`
```json
{ "status": 400, "errors": [{ "field": "sex", "message": "성별은 M 또는 F만 입력 가능합니다." }] }
```

## Run Tests

```bash
cd Latinhouse.Be
./gradlew test
```

## Swagger UI

애플리케이션 실행 후: http://localhost:8080/swagger-ui/index.html
