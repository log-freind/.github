# Log Friends

[English](#english) | [한국어](#korean)

## English

Log Friends is a lightweight event collection platform for Spring Boot applications. It captures runtime events through an SDK, sends them to a Console over HTTP JSON batches, and stores them in PostgreSQL / TimescaleDB for exploration and first-phase analytics.

```text
Spring Boot App + log-friends-sdk
  -> HTTP JSON batch POST /ingest
  -> log-friends-console
  -> PostgreSQL / TimescaleDB
  -> Dashboard / Log Catalog
```

The first-phase goal is intentionally small: no Kafka, NATS, Spark, ClickHouse, or microservice split. Log Friends focuses on fewer operational components so small teams can store Raw Events and inspect business event flows without a heavy observability stack.

### Product Idea

Log Friends is not trying to replace Datadog or New Relic.

The first users are backend engineers, data engineers, and data platform operators who need a simple way to see what eventName flows out of each app.

The main differentiator is the Log Catalog:

```text
LogSpec + Recent Sample + Mismatch + Field Request
```

Data engineers can inspect actual samples, see mismatches, and request fields without guessing what backend services emit. Backend engineers still own the event contract and code changes.

### Architecture

```text
Target Spring Boot App
  + log-friends-sdk
      - ByteBuddy instrumentation
      - bounded in-memory queue
      - JSON batch transport
      - masking before send
        |
        | POST /ingest
        v
log-friends-console
  - Raw Event ingest
  - Agent / Worker management
  - Log Catalog API + static UI
  - scheduler-based statistics
        |
        v
PostgreSQL / TimescaleDB
```

### Event Types

| eventType | Stored in | Description |
|---|---|---|
| `LOG_EVENT` | `custom_events` | Business eventName captured from `@LogEvent` |
| `LOG` | `logs` | Logback logs |
| `HTTP` | `http_events` | HTTP request/response metadata |
| `JDBC` | `jdbc_events` | JDBC execution metadata |
| `METHOD_TRACE` | `method_traces` | Spring `@Service` method duration |

`LOG_EVENT.eventName` is required and must be camelCase. Invalid `LOG_EVENT` values are not sent by the SDK; the target app logs a warning instead.

### Repositories

| Repository | Role |
|---|---|
| [log-friends-sdk](https://github.com/log-freind/log-friends-sdk) | Captures events inside Spring Boot apps and sends them to Console `/ingest` |
| [log-friends-console](https://github.com/log-freind/log-friends-console) | Ingest, storage, Agent management, Log Catalog, statistics |
| [log-friends-examples](https://github.com/log-freind/log-friends-examples) | Example Spring Boot applications |

### SDK Quick Start

```kotlin
dependencies {
    implementation("com.logfriends:log-friends-sdk:1.2.0")
}
```

Required configuration:

```bash
export LOGFRIENDS_WORKER_ID=order-api-local-1
export LOGFRIENDS_INGEST_URL=http://localhost:8082/ingest
```

JVM attach option:

```bash
java -Djdk.attach.allowAttachSelf=true -jar your-app.jar
```

If required settings are missing, the SDK disables capture/transport without failing the target app.

### First-Phase Policy

- SDK transport is HTTP JSON `POST /ingest`.
- `workerId` and `ingestUrl` are required.
- SDK transport failures never fail the target app.
- Queue full or transport failure drops the batch and logs periodic warnings.
- SDK does not auto-register LogSpec.
- Console owns Raw Event storage and statistics generation.
- Console first-phase UI is Spring Boot static HTML with lightweight JavaScript.

---

## Korean

Log Friends는 Spring Boot 앱에서 발생하는 런타임 이벤트를 가볍게 수집하고, 데이터 엔지니어가 Console에서 탐색할 수 있게 만드는 이벤트 수집 플랫폼입니다.

```text
Spring Boot App + log-friends-sdk
  -> HTTP JSON batch POST /ingest
  -> log-friends-console
  -> PostgreSQL / TimescaleDB
  -> Dashboard / Log Catalog
```

1차 목표는 작게 잡습니다. Kafka, NATS, Spark, ClickHouse, MSA 없이 운영 구성 요소를 줄이고, 작은 팀에서도 Raw Event 저장과 최소 분석 흐름을 만들 수 있게 하는 것이 목표입니다.

### 제품 방향

Log Friends는 Datadog이나 New Relic을 대체하려는 대형 Observability 제품이 아닙니다.

1차 사용자는 제한된 리소스 안에서 앱별 eventName 흐름을 확인하고 싶은 백엔드 개발자, 데이터 엔지니어, 빅데이터 담당자입니다.

핵심 차별점은 Log Catalog입니다.

```text
LogSpec + Recent Sample + Mismatch + Field Request
```

데이터 담당자는 실제 샘플과 mismatch를 보고 필요한 field를 요청할 수 있습니다. 백엔드 엔지니어는 eventName 계약과 코드 변경의 소유권을 유지합니다.

### 아키텍처

```text
Target Spring Boot App
  + log-friends-sdk
      - ByteBuddy instrumentation
      - bounded in-memory queue
      - JSON batch transport
      - masking before send
        |
        | POST /ingest
        v
log-friends-console
  - Raw Event ingest
  - Agent / Worker management
  - Log Catalog API + static UI
  - scheduler-based statistics
        |
        v
PostgreSQL / TimescaleDB
```

### Event Types

| eventType | 저장 대상 | 설명 |
|---|---|---|
| `LOG_EVENT` | `custom_events` | `@LogEvent` 기반 비즈니스 eventName |
| `LOG` | `logs` | Logback 로그 |
| `HTTP` | `http_events` | HTTP 요청/응답 메타데이터 |
| `JDBC` | `jdbc_events` | JDBC 실행 메타데이터 |
| `METHOD_TRACE` | `method_traces` | Spring `@Service` 메서드 실행 시간 |

`LOG_EVENT.eventName`은 camelCase 필수입니다. 유효하지 않은 `LOG_EVENT`는 SDK에서 전송하지 않고 대상 앱 로그에 warn을 남깁니다.

### Repositories

| Repository | Role |
|---|---|
| [log-friends-sdk](https://github.com/log-freind/log-friends-sdk) | Spring Boot 앱 내부에서 이벤트를 캡처하고 Console `/ingest`로 전송 |
| [log-friends-console](https://github.com/log-freind/log-friends-console) | 이벤트 수신, 저장, Agent 관리, Log Catalog, 통계 생성 |
| [log-friends-examples](https://github.com/log-freind/log-friends-examples) | SDK/Console 연동 예제 앱 |

### SDK Quick Start

```kotlin
dependencies {
    implementation("com.logfriends:log-friends-sdk:1.2.0")
}
```

필수 설정:

```bash
export LOGFRIENDS_WORKER_ID=order-api-local-1
export LOGFRIENDS_INGEST_URL=http://localhost:8082/ingest
```

JVM attach 옵션:

```bash
java -Djdk.attach.allowAttachSelf=true -jar your-app.jar
```

설정이 없으면 SDK는 앱을 죽이지 않고 캡처/전송을 비활성화합니다.

### 1차 정책

- SDK transport는 HTTP JSON `POST /ingest`입니다.
- `workerId`와 `ingestUrl`은 필수 설정입니다.
- SDK 전송 실패는 대상 앱을 실패시키지 않습니다.
- queue full 또는 전송 실패 시 batch는 drop하며, 주기적으로 warn을 남깁니다.
- SDK는 LogSpec을 자동 등록하지 않습니다.
- Console이 Raw Event 저장과 통계 생성을 담당합니다.
- Console 1차 UI는 Spring Boot static HTML + 가벼운 JavaScript입니다.
