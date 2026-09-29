<p align="center">
  <img src="https://raw.githubusercontent.com/log-freind/.github/main/profile/assets/log-friends-logo.svg" alt="Log Friends logo" width="480">
</p>

# Log Friends

**서비스 데이터의 의미를 코드에 남기고, 실제 발생한 이벤트와 함께 확인하는 경량 셀프호스팅 플랫폼입니다.**

백엔드 엔지니어는 이벤트 이름과 필드 설명을 코드에 정의합니다.
데이터·ML 엔지니어는 Console에서 설명과 실제 데이터를 함께 살펴보고, 필요한 변경을 더 구체적으로 논의할 수 있습니다.

> Connect event definitions in code with real payloads, so backend and data/ML teams can understand service data together.

## 어떤 문제를 해결하나요?

서비스 데이터를 활용하기 전에는 어떤 데이터가 있는지, 각 필드가 무엇을 뜻하는지,
언제 발생하는지를 확인해야 합니다. 코드와 문서, 로그가 떨어져 있으면 담당자에게 같은 질문을 반복하게 됩니다.

Log Friends는 **코드 설명 → 실제 이벤트 → 계약과의 차이**를 연결해 이 확인 작업을 줄이는 것을 목표로 합니다.
데이터 분석이나 모델 학습 자체를 대신하는 도구는 아닙니다.

## 무엇을 볼 수 있나요?

| 화면 | 확인할 내용 |
|---|---|
| **Log Catalog** | 이벤트 설명, 필드 계약, 코드에서 발견한 힌트, 실제 샘플, 누락·추가 필드 |
| **Raw Events** | 발생한 이벤트를 앱·기간·세션 등으로 조회하고 CSV로 내보내기 |
| **Overview** | 같은 기간의 HTTP 트래픽, 지연, 이벤트 발생 횟수, 오류 |
| **Frontend Tree** | 브라우저 이벤트에 기록된 페이지·컴포넌트 위치 |

Console은 AI 도구가 이벤트 계약과 샘플 등을 조회할 수 있는 **읽기 전용 MCP**도 제공합니다.

## 어떻게 동작하나요?

```text
Spring Boot 서비스 + Kotlin SDK
Node.js / 브라우저 / JavaScript 모바일 앱 + TypeScript SDK
                       │
                       │ HTTP JSON batch
                       ▼
                    Console ── PostgreSQL / TimescaleDB
                       │
                       ├── Console Web
                       └── 읽기 전용 MCP (선택)
```

- **Kotlin SDK**: ByteBuddy로 런타임 정보를 수집하고, `@LogEvent`와 `@LogField`로 이벤트 의미를 남깁니다.
- **TypeScript SDK**: 명시적인 이벤트 호출이나 데코레이터로 데이터를 수집합니다. 브라우저 컴포넌트 위치는 직접 지정합니다.
- **Console**: 이벤트 저장, 계약 관리, 샘플 조회와 필드 비교를 담당합니다.

별도 메시지 브로커 없이 SDK가 Console에 직접 전송하는 구조입니다.

## 처음 사용한다면

**Console → Examples → Console Web** 순서로 실행하면 수집부터 조회까지 확인할 수 있습니다.
각 저장소 README에 준비 사항과 실행 명령이 있습니다.

1. [Console](https://github.com/log-freind/log-friends-console#readme): PostgreSQL/TimescaleDB를 준비하고 백엔드를 실행합니다.
2. [Examples](https://github.com/log-freind/log-friends-examples#readme): 쇼핑몰 예제를 실행하고 상품을 조회합니다.
3. [Console Web](https://github.com/log-freind/log-friends-console-web#readme): Raw Events에서 `catalogProductsListed`를 찾고 Log Catalog에서 설명과 샘플을 확인합니다.

직접 만든 서비스에 연결하려면 아래에서 실행 환경에 맞는 SDK를 선택하세요.
현재 Console은 **HTTP 요청당 최대 50건**을 받으므로 Kotlin SDK와 Examples의
`LOGFRIENDS_BATCH_SIZE`를 **50 이하**로 지정해야 합니다.

## 저장소 안내

| 저장소 | 이런 경우에 사용하세요 |
|---|---|
| [log-friends-kt-sdk](https://github.com/log-freind/log-friends-kt-sdk) | Spring Boot 서비스에 Kotlin/JVM SDK 연결 |
| [log-friends-ts-sdk](https://github.com/log-freind/log-friends-ts-sdk) | Node.js·브라우저·JavaScript 모바일 앱에 SDK 연결 |
| [log-friends-console](https://github.com/log-freind/log-friends-console) | 이벤트 저장과 조회 API, 계약 관리, MCP 실행 |
| [log-friends-console-web](https://github.com/log-freind/log-friends-console-web) | 웹 화면에서 이벤트 확인 |
| [log-friends-examples](https://github.com/log-freind/log-friends-examples) | 쇼핑몰 예제로 전체 흐름 체험 |
| [log-friends-infra](https://github.com/log-freind/log-friends-infra) | NAS/MicroK8s 운영 환경의 배포 설정 확인 (접근 권한 필요) |

## 사용 전에 알아둘 점

- **코드 힌트와 확정 계약은 다릅니다.** SDK가 발견한 정의는 힌트로 보고되며, 확정 계약인 LogSpec은 Console API에서 관리합니다. 현재 mismatch는 필드 누락·추가 비교이며 타입·중첩 구조 전체 검증은 아닙니다.
- **서비스 보호를 우선하는 수집 방식입니다.** 큐 제한과 drop 정책으로 무한 적재를 제한하지만, 데이터 유실이나 OOM 방지를 완전히 보장하지 않습니다. 결제·주문 원장을 대체하지 않습니다.
- **내부망 사용을 전제로 검토하세요.** 현재 애플리케이션 인증·권한 관리가 없으며 CORS나 MCP Host/Origin 검사는 인증을 대신하지 않습니다. 민감정보는 수집 전에 제거해야 합니다.

설치 버전, 설정 기본값, 지원 범위는 각 저장소 README를 기준으로 확인하세요.
