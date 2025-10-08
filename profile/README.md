# Traffic Tacos 🌮

> 30k RPS 트래픽에도 안정성을 보장하는 Cloud-Native 티켓 예약 플랫폼

## 🎯 프로젝트 개요

Traffic Tacos는 대규모 트래픽 환경에서 안정적인 티켓 예약 서비스를 제공하기 위해 설계된 마이크로서비스 기반 플랫폼입니다. AWS EKS 기반의 컨테이너 오케스트레이션과 이벤트 기반 아키텍처를 통해 초당 30,000건의 요청을 안정적으로 처리할 수 있습니다.

### 핵심 특징

- **🚀 고성능**: 30k RPS 처리 능력, P95 지연시간 < 120ms
- **🔒 Zero Oversell**: DynamoDB 조건부 업데이트를 통한 재고 관리로 좌석 중복 판매 0%
- **⚡ Event-Driven**: EventBridge + SQS 기반 비동기 처리 아키텍처
- **🌐 Cloud-Native**: AWS EKS 기반 컨테이너 오케스트레이션
- **📊 완전한 관측성**: OpenTelemetry + Prometheus + Grafana 통합 모니터링
- **🔐 엔터프라이즈 보안**: JWT 인증, 멱등성 관리, 레이트 리미팅

## 👥 팀 구성원

| 이름 | 이메일 |
|------|--------|
| [황정목](https://www.linkedin.com/in/jeongmok-hwang-51a09720a/) | tyler@makestar.com |
| [김서연](https://www.linkedin.com/in/%EC%84%9C%EC%97%B0-%EA%B9%80-0070b0254/) | maruruyeon@gmail.com |
| [조현민](https://www.linkedin.com/in/hyeonmin-cho-145992b9/) | galaxyeunha0530@gmail.com |
| [정다빈](https://www.linkedin.com/in/manyb2ns/) | manyb2ns@gmail.com |
| [허유정](https://www.linkedin.com/in/yoojunghuh/) | gjenfwo@gmail.com |

> 💡 **이름을 클릭하면 LinkedIn 프로필로 이동합니다**

## 🏗️ 시스템 아키텍처

### 계층별 구조

```
┌─────────────────────────────────────────────────────────────────┐
│                    Layer 0: Frontend Layer                      │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │           Reservation Web (React + Vite)                  │ │
│  │  • 사용자 대기열 인터페이스                                    │ │
│  │  • 실시간 상태 업데이트 (WebSocket)                           │ │
│  │  • CloudFront + S3 정적 호스팅                               │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓ HTTP/REST + WebSocket
┌─────────────────────────────────────────────────────────────────┐
│                   Layer 1: Gateway Layer                        │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Gateway API (Go + Fiber)                     │ │
│  │  • 인증/인가 (JWT OIDC)                                      │ │
│  │  • 대기열 관리 (30k RPS 트래픽 제어)                          │ │
│  │  • 레이트 리미팅 및 멱등성 관리                                │ │
│  │  • BFF (Backend for Frontend)                            │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓ gRPC + REST
┌─────────────────────────────────────────────────────────────────┐
│                 Layer 2: Business Services                      │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐ │
│  │ Reservation API  │  │  Inventory API   │  │ Payment Sim  │ │
│  │  (Kotlin/Spring) │  │    (Go/gRPC)     │  │  (Go/gRPC)   │ │
│  │                  │  │                  │  │              │ │
│  │ • 예약 생성/확정   │  │ • 재고 관리        │  │ • 결제 시뮬  │ │
│  │ • 60초 홀드      │  │ • Zero Oversell  │  │ • 웹훅 처리   │ │
│  │ • DynamoDB      │  │ • 좌석 소유권     │  │ • EventBridge│ │
│  └──────────────────┘  └──────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓ Events (EventBridge + SQS)
┌─────────────────────────────────────────────────────────────────┐
│              Layer 3: Background Processing                     │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │     Reservation Worker (Go + Kubernetes Job)              │ │
│  │  • 예약 만료 처리 (60초 타이머)                               │ │
│  │  • 결제 승인/실패 후속 처리                                   │ │
│  │  • KEDA Auto-scaling (SQS 기반)                            │ │
│  │  • 보상 트랜잭션 처리                                        │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 데이터 플로우

```
사용자 → CloudFront → Gateway API → Reservation API → Inventory API
                                    ↓
                              EventBridge → SQS → Worker
                                    ↓
                              Payment Sim API
```

## 🗂️ 레포지토리 구조

| 레포지토리 | 기술 스택 | 역할 | 포트 |
|----------|---------|-----|------|
| **gateway-api** | Go + Fiber | API Gateway, BFF, 대기열 관리 | 8000 (REST), 8001 (gRPC) |
| **reservation-api** | Kotlin + Spring Boot WebFlux | 예약 라이프사이클 관리, 60초 홀드 | 8010 (REST), 8011 (gRPC) |
| **inventory-api** | Go + gRPC | 재고 관리, Zero Oversell 보장 | 8020 (REST), 8021 (gRPC) |
| **payment-sim-api** | Go + gRPC | 결제 시뮬레이션, 웹훅 처리 | 8030 (REST), 8031 (gRPC) |
| **reservation-worker** | Go/Kotlin | 백그라운드 작업 (만료/결제 처리) | 8040 (REST), 8041 (gRPC) |
| **reservation-web** | React + Vite | 프론트엔드 SPA | 3000 |
| **proto-contracts** | Protocol Buffers | gRPC 서비스 간 계약 정의 | - |
| **infra-iac** | Terraform | AWS 인프라 코드 | - |

## 🛠️ 기술 스택

### 인프라 (AWS)

- **컴퓨팅**: EKS 1.33 (3개 노드 그룹 - ondemand/mix/monitoring)
- **데이터베이스**: DynamoDB (6개 테이블 - reservations, orders, inventory 등)
- **이벤트**: EventBridge (2개 커스텀 버스), SQS (FIFO 큐 + DLQ)
- **캐싱**: ElastiCache Redis (Multi-AZ, 암호화)
- **DNS & SSL**: Route53, ACM (와일드카드 인증서)
- **CDN**: CloudFront + S3 (정적 웹 호스팅)
- **모니터링**: AWS Managed Prometheus, AWS Managed Grafana

### 백엔드

- **API Gateway**: Go 1.22+, Fiber v2
- **Reservation**: Kotlin, Spring Boot 3.x WebFlux, Gradle
- **Inventory**: Go 1.24+, gRPC, Echo
- **Payment Sim**: Go 1.24+, gRPC
- **Worker**: Go/Kotlin, KEDA Auto-scaling

### 프론트엔드

- **Framework**: React 18+, Vite
- **State**: React Context API / Redux Toolkit
- **Real-time**: WebSocket

### 프로토콜 & 통신

- **동기**: REST (HTTP/JSON), gRPC (Protocol Buffers)
- **비동기**: EventBridge, SQS
- **인증**: JWT (OIDC/Cognito)
- **관측성**: OpenTelemetry (Trace/Metric/Log)

### 개발 도구

- **IaC**: Terraform 1.5+
- **컨테이너**: Docker, Kubernetes
- **CI/CD**: GitHub Actions
- **모니터링**: Prometheus, Grafana, Jaeger/Tempo, Loki

## 🔄 핵심 비즈니스 플로우

### 1. 대기열 → 예약 플로우

```
1. 사용자 → POST /api/v1/queue/join
   - 30k RPS 트래픽 유입
   - 대기열 토큰 발급

2. 실시간 위치 업데이트 (WebSocket)
   - GET /api/v1/queue/status
   - 예상 대기시간 제공

3. 입장 허가 → POST /api/v1/queue/enter
   - reservation_token 발급 (TTL: 30초)

4. 예약 생성 → POST /api/v1/reservations
   - 60초 홀드 시작
   - Inventory API 재고 확인 (gRPC)
   - EventBridge Scheduler 등록 (만료 타이머)

5. 결제 처리 → POST /api/v1/payments/intent
   - Payment Sim API 호출
   - 웹훅 대기 (승인/실패)

6-1. 결제 승인 → POST /api/v1/reservations/{id}/confirm
   - Inventory CommitReservation (gRPC)
   - 주문 확정 (orders 테이블)

6-2. 만료 처리 (60초 경과)
   - Worker가 SQS 이벤트 소비
   - Inventory ReleaseHold (gRPC)
   - 상태 → EXPIRED
```

### 2. 백그라운드 처리 (Worker)

```
EventBridge 이벤트 → SQS 큐 → KEDA Scaling → Worker Pod
                                              ↓
                              ┌───────────────┴────────────────┐
                              │                                │
                    reservation.expired              payment.approved/failed
                              ↓                                ↓
                   Inventory ReleaseHold              Inventory Commit/Release
                   Status → EXPIRED                   Status → CONFIRMED/CANCELLED
```

## 🔐 보안 아키텍처

### 엣지 보안

- **AWS WAF/Shield**: DDoS 방어, Bot 차단
- **Rate Limiting**: IP/사용자별 50 RPS (버스트 100)
- **CloudFront**: HTTPS 강제, 지리적 제한

### API 보안

- **JWT 검증**: OIDC/Cognito 기반 인증
- **Idempotency Key**: UUID-v4 (TTL: 5분, Redis)
- **보안 헤더**: CSP, HSTS, X-Frame-Options

### 내부 보안

- **mTLS**: 서비스 간 암호화 통신
- **cert-manager**: TLS 인증서 자동 관리
- **IRSA**: IAM Roles for Service Accounts
- **Network Policy**: Namespace 격리

### 데이터 보안

- **전송 중 암호화**: TLS 1.3
- **저장 시 암호화**: DynamoDB KMS, ElastiCache 암호화
- **Secrets**: AWS Secrets Manager 통합

## 📊 관측성 (Observability)

### 메트릭 수집

```yaml
# Prometheus 메트릭
http_server_requests_total{method, route, status_code}
http_server_requests_duration_seconds{method, route}
backend_call_duration_seconds{service, method}
reservation_status_total{status}  # HOLD/CONFIRMED/EXPIRED
grpc_call_duration_seconds{service, method}
```

### 분산 추적

- **OpenTelemetry**: 전체 서비스 trace 연결
- **W3C Trace Context**: traceparent 헤더 자동 전파
- **샘플링**: 예약 경로 100%, 기타 10%

### 로깅

```json
{
  "ts": "2024-01-01T12:00:00Z",
  "level": "info",
  "service": "gateway-api",
  "trace_id": "abc123...",
  "msg": "reservation_created",
  "reservation_id": "rsv_xyz",
  "latency_ms": 45.2
}
```

### 대시보드

- **AWS Managed Grafana**: 통합 모니터링 대시보드
- **커스텀 대시보드**: 30k RPS 성능 목표 추적
- **알람**: CloudWatch 알람 (읽기/쓰기 스로틀링)

## 🚀 성능 목표

| 지표 | 목표 | 측정 방법 |
|-----|------|---------|
| **시스템 처리량** | 30,000 RPS | Prometheus metrics |
| **P95 지연시간** | < 120ms | OpenTelemetry traces |
| **예약 확정** | < 500ms | API Gateway metrics |
| **에러율** | < 1% | Success rate monitoring |
| **오버셀 방지** | 0% | DynamoDB 조건부 업데이트 |
| **가용성** | 99.9% | Multi-AZ EKS |

## 💰 비용 최적화 (FinOps)

### 컴퓨팅

- **EKS Node Groups**: ondemand + spot 혼합
- **Graviton (ARM64)**: 20% 비용 절감
- **HPA + KEDA**: 자동 스케일링 (이벤트 기반)
- **Karpenter**: 노드 프로비저닝 최적화

### 데이터베이스

- **DynamoDB**: 온디맨드 → 프로비저닝 모드 전환
- **AutoScaling**: 읽기/쓰기 용량 자동 조정
- **TTL**: idempotency 테이블 자동 정리

### 네트워크

- **CloudFront**: 엣지 캐싱으로 origin 요청 감소
- **VPC Endpoints**: NAT Gateway 비용 절감

## 🛠️ 개발 환경 설정

### 전제 조건

- Docker & Docker Compose
- Go 1.22+
- Java 17+ (Kotlin)
- Node.js 18+
- Terraform 1.5+
- kubectl, helm

### 로컬 실행

```bash
# 1. 전체 서비스 시작
./run_local_all.sh start

# 2. AWS 리소스 발견 및 설정
./discover_aws_resources.sh generate

# 3. 연결 테스트
./test_connections.sh test

# 4. 통합 테스트
./test_all_integrated.sh full

# 5. 헬스 체크
./health_check.sh monitor

# 6. 서비스 중지
./run_local_all.sh stop
```

### 개별 서비스 빌드

```bash
# Gateway API (Go)
cd gateway-api
go build -o gateway-api ./cmd/gateway-api

# Reservation API (Kotlin)
cd reservation-api
./gradlew clean build

# Inventory API (Go)
cd inventory-api
make build

# Payment Sim API (Go)
cd payment-sim-api
make build

# Frontend (React)
cd reservation-web
npm install && npm run build
```

### 인프라 배포

```bash
# Terraform 초기화
cd infra-iac
terraform init

# 인프라 배포
terraform plan
terraform apply

# 출력 확인
terraform output
```

## 📋 API 문서

### Gateway API 엔드포인트

#### 대기열 관리

- `POST /api/v1/queue/join` - 대기열 참여 (익명 허용)
- `GET /api/v1/queue/status` - 대기열 상태 조회
- `POST /api/v1/queue/enter` - 입장 요청
- `DELETE /api/v1/queue/leave` - 대기열 이탈

#### 예약 관리

- `GET /api/v1/events/{id}/availability` - 가용성 조회
- `POST /api/v1/reservations` - 예약 생성 (멱등성 필수)
- `GET /api/v1/reservations/{id}` - 예약 조회
- `POST /api/v1/reservations/{id}/confirm` - 예약 확정
- `DELETE /api/v1/reservations/{id}` - 예약 취소

#### 결제 처리

- `POST /api/v1/payments/intent` - 결제 인텐트 생성
- `POST /api/v1/payments/process` - 결제 처리
- `GET /api/v1/payments/{id}/status` - 결제 상태 조회

### gRPC 서비스

- **QueueService**: 대기열 관리, admission control
- **ReservationService**: 예약 CRUD, 60초 홀드
- **InventoryService**: 재고 확인, 좌석 할당/해제
- **PaymentService**: 결제 시뮬레이션, 웹훅

자세한 API 스펙은 [proto-contracts](https://github.com/traffic-tacos/proto-contracts) 참조

## 🧪 테스트

### 통합 테스트

```bash
# 전체 테스트 스위트
./test_all_integrated.sh full

# 빠른 스모크 테스트
./test_all_integrated.sh quick

# 컴포넌트별 테스트
./test_all_integrated.sh backend
./test_all_integrated.sh frontend
./test_all_integrated.sh worker
```

### 성능 테스트

```bash
# Reservation API (JMeter)
cd reservation-api
./gradlew jmeterRun

# Inventory API (ghz)
cd inventory-api
make load-test-ghz

# Payment Sim API
cd payment-sim-api
make perf-test
```

### 프론트엔드 테스트

```bash
cd reservation-web
npm run test          # Jest 유닛 테스트
npm run test:e2e      # Cypress E2E
npm run type-check    # TypeScript 검증
```

## 📚 주요 문서

- [인프라 구성 가이드](infra-iac/README.md)
- [API 계약 문서](proto-contracts/README.md)
- [DynamoDB 스펙](infra-iac/docs/spec/dynamodb-spec.md)
- [EventBridge 스펙](infra-iac/docs/spec/eventbridge-spec.md)
- [개발 수칙](gateway-api/.cursor/rules/rules-must-be-followed.mdc)

## 🤝 기여 가이드

### 브랜치 전략

- `main`: 프로덕션 코드
- `develop`: 개발 브랜치
- `feature/*`: 기능 개발
- `hotfix/*`: 긴급 수정

### 커밋 컨벤션

```
feat: 새로운 기능 추가
fix: 버그 수정
docs: 문서 수정
refactor: 코드 리팩토링
test: 테스트 코드
chore: 빌드/설정 변경
perf: 성능 개선
```

### Pull Request

1. Feature 브랜치 생성
2. 변경사항 구현
3. 테스트 통과 확인
4. PR 생성 및 리뷰 요청
5. Merge 승인 후 배포

## 📞 연락처

- **조직**: Traffic Tacos Team
- **프로젝트 목표**: 30k RPS 안정 처리 티켓 예약 플랫폼
- **라이선스**: Internal Use Only

---

**Traffic Tacos** - 고성능, 안정성, 확장성을 모두 갖춘 Cloud-Native 티켓 예약 플랫폼 🌮