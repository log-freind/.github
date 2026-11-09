# MicroK8s 라우팅 참고 설정

이 디렉터리는 조직 저장소에 남겨 둔 **Ingress 참고 설정**입니다. 전체 배포의 관리 위치는 [log-friends-infra](https://github.com/log-freind/log-friends-infra)이며 접근 권한이 필요할 수 있습니다.

로컬 개발 작업 트리의 `06-ingress.yaml`은 다음 경로를 연결합니다. 이 README 전용 PR에는 manifest 수정이 포함되지 않으므로 GitHub 기본 브랜치의 옛 라우팅과 구분하세요.

```text
/           -> log-friends-console-web:3000
/logfriend  -> log-friends-console:8080
```

과거 `/api`, `/ingest`, `/actuator` 직접 라우팅 안내는 현행 `/logfriend` 컨텍스트와 다릅니다. Console 상태 확인 경로는 기본 컨텍스트에서 `/logfriend/actuator/health`입니다.

## 적용 전에 확인할 것

- 이 파일만으로 DB·앱 키·학습 Job 권한·전체 서비스를 구성하지 않습니다.
- 현재 namespace, 서비스 이름, Console 컨텍스트, Ingress controller를 확인하세요.
- 호스트·DB 연결·이미지 태그는 배포 환경에서 정합니다. 비밀번호·앱 키·레지스트리 토큰을 ConfigMap이나 Git에 넣지 마세요.
- Ingress는 인증·권한 검사를 대신하지 않습니다. HTTPS와 접근 제어를 별도로 검토하세요.

현재 NAS에 그대로 적용하는 빠른 실행 명령은 제공하지 않습니다. 전체 배포 절차와 위험 사항은 Infra 저장소 README를 따르세요.
