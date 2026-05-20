# Log Friends

Spring Boot 앱에서 발생하는 런타임 이벤트를 가볍게 수집하고, 데이터 엔지니어가 Console에서 탐색할 수 있게 만드는 로그 수집 플랫폼입니다.

핵심 방향은 단순합니다.

```text
Spring Boot App + log-friends-sdk
  -> HTTP JSON batch POST /ingest
  -> log-friends-console
  -> PostgreSQL / TimescaleDB
  -> Dashboard / Log Catalog
```

Kafka, NATS, Spark, ClickHouse, MSA 없이 1차 목표를 달성합니다. 운영 구성 요소를 줄이고, 작은 팀에서도 원본 이벤트 저장과 최소 분석 흐름을 만들 수 있게 하는 것이 목표입니다.

## Product Idea

Log Friends는 Datadog이나 New Relic을 대체하려는 대형 Observability 제품이 아닙니다.

1차 사용자는 제한된 리소스 안에서 앱별 이벤트 흐름을 확인하고 싶은 백엔드 개발자, 데이터 엔지니어, 빅데이터 담당자입니다.

특히 `LOG_EVENT`는 핵심 비즈니스 eventName을 기준으로 수집합니다. 데이터 담당자는 Log Catalog에서 LogSpec, 최근 샘플, mismatch, Field Request를 보고 백엔드 팀과 필요한 eventName/field를 협업할 수 있습니다.

## Architecture

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

## Event Types

| eventType | 저장 대상 | 설명 |
|---|---|---|
| `LOG_EVENT` | `custom_events` | `@LogEvent` 기반 비즈니스 eventName |
| `LOG` | `logs` | Logback 로그 |
| `HTTP` | `http_events` | HTTP 요청/응답 메타데이터 |
| `JDBC` | `jdbc_events` | JDBC 실행 메타데이터 |
| `METHOD_TRACE` | `method_traces` | Spring `@Service` 메서드 실행 시간 |

`LOG_EVENT.eventName`은 camelCase 필수입니다. 유효하지 않은 `LOG_EVENT`는 SDK에서 전송하지 않고 대상 앱 로그에 warn을 남깁니다.

## Repositories

| Repository | Role |
|---|---|
| [log-friends-sdk](https://github.com/log-freind/log-friends-sdk) | Spring Boot 앱 내부에서 이벤트를 캡처하고 Console `/ingest`로 전송 |
| [log-friends-console](https://github.com/log-freind/log-friends-console) | 이벤트 수신, 저장, Agent 관리, Log Catalog, 통계 생성 |
| [log-friends-examples](https://github.com/log-freind/log-friends-examples) | SDK/Console 연동 예제 앱 |

## SDK Quick Start

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

## Current First-Phase Policy

- SDK transport는 HTTP JSON `POST /ingest`입니다.
- `workerId`와 `ingestUrl`은 필수 설정입니다.
- SDK 전송 실패는 대상 앱을 실패시키지 않습니다.
- queue full 또는 전송 실패 시 batch는 drop하며, 주기적으로 warn을 남깁니다.
- SDK는 LogSpec을 자동 등록하지 않습니다.
- Console이 Raw Event 저장과 통계 생성을 담당합니다.
- Console 1차 UI는 Spring Boot static HTML + 가벼운 JavaScript입니다.

## Log Catalog

Log Catalog는 단순한 이벤트 목록이 아닙니다.

```text
LogSpec + Recent Sample + Mismatch + Field Request
```

기본 축은 `appName`이고, `workerId`는 하위 필터로 봅니다. Field Request는 계약 변경 제안이며, Jira/Linear/GitHub 티켓 시스템을 대체하지 않습니다.
