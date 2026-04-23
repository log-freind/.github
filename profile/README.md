# Log Friends

**코드 수정 없이** Spring Boot 앱을 계측하는 Observability 에이전트

ByteBuddy 런타임 계측 → HTTP POST → Console → PostgreSQL

---

## 아키텍처

```
Your Spring Boot App
  + log-friends-sdk (dependency)
        │
        │  HTTP POST /ingest (JSON batch)
        ▼
  log-friends-console  ──────►  PostgreSQL
  (Spring Boot :8082)
        │
        ▼
  GET /api/events/*
```

## 수집 이벤트

| 이벤트 | 설명 |
|---|---|
| `HTTP` | 모든 HTTP 요청/응답 (헤더 포함) |
| `LOG` | Logback 로그 (MDC 포함) |
| `JDBC` | PreparedStatement 실행 쿼리 + 소요시간 |
| `METHOD_TRACE` | `@Service` 메서드 ≥10ms 자동 추적 |
| `LOG_EVENT` | `@LogEvent` 커스텀 이벤트 |

## 빠른 시작

**1. SDK 추가** (`build.gradle`)
```kotlin
repositories {
    maven { url = uri("https://jitpack.io") }
}
dependencies {
    implementation("com.github.log-freind:log-friends-sdk:v1.2.0")
}
```

**2. Console + PostgreSQL 실행**
```bash
docker compose -f docker-compose.infra.yml up -d
docker compose -f docker-compose.platform.yml up -d
```

**3. 앱 실행**
```bash
java -Djdk.attach.allowAttachSelf=true \
     -DLOGFRIENDS_INGEST_URL=http://localhost:8082/ingest \
     -jar your-app.jar
```

## 레포지토리

| 레포 | 설명 |
|---|---|
| [log-friends-sdk](https://github.com/log-freind/log-friends-sdk) | ByteBuddy 계측 에이전트 SDK |
| [log-friends-console](https://github.com/log-freind/log-friends-console) | 이벤트 수신 + 조회 REST API 서버 |
| [log-friends-examples](https://github.com/log-freind/log-friends-examples) | 예제 Spring Boot 앱 |

## 주요 설정

| 환경변수/프로퍼티 | 기본값 | 설명 |
|---|---|---|
| `LOGFRIENDS_INGEST_URL` | `http://localhost:8082/ingest` | Console 엔드포인트 |
| `logfriends.interceptor.*.enabled` | `true` | 인터셉터 개별 on/off |
| `logfriends.trace.threshold.ms` | `10` | METHOD_TRACE 최소 임계값 |
| `logfriends.batch.size` | `100` | 배치 크기 |
