# Traffic Tacos 🌮

> 30k RPS 트래픽에도 안정성을 보장하는 Cloud-Native 티켓 예약 플랫폼

<div align="center">

![Build Status](https://img.shields.io/badge/build-passing-brightgreen?style=flat-square)
![Go Version](https://img.shields.io/badge/Go-1.22+-00ADD8?style=flat-square&logo=go&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-2.1+-7F52FF?style=flat-square&logo=kotlin&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.33-326CE5?style=flat-square&logo=kubernetes&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-1.8+-7B42BC?style=flat-square&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EKS-FF9900?style=flat-square&logo=amazon-aws&logoColor=white)
![License](https://img.shields.io/badge/License-Internal-blue?style=flat-square)

</div>

## 🔗 Quick Links

<div align="center">

| 📖 [아키텍처](#️-시스템-아키텍처) | 🗂️ [레포지토리](#️-레포지토리-구조) | 🚀 [시작하기](#-빠른-시작) | 📊 [성능](#-성능-목표) | 👥 [팀](#-팀-구성원) | 🤝 [기여](#-기여-가이드) |

</div>

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

| 이름 | 이메일 | 직무 |
|------|--------|-----|
| [황정목](https://www.linkedin.com/in/jeongmok-hwang-51a09720a/) | xnslqjtmghaf@gmail.com | DevOps Engineer |
| [김서연](https://www.linkedin.com/in/%EC%84%9C%EC%97%B0-%EA%B9%80-0070b0254/) | maruruyeon@gmail.com | Cloud Engineer |
| [조현민](https://www.linkedin.com/in/hyeonmin-cho-145992b9/) | galaxyeunha0530@gmail.com | Software Engineer |
| [정다빈](https://www.linkedin.com/in/manyb2ns/) | manyb2ns@gmail.com | Solution Architect |
| [허유정](https://www.linkedin.com/in/yoojunghuh/) | gjenfwo@gmail.com | DevOps Engineer |

> 💡 **이름을 클릭하면 LinkedIn 프로필로 이동합니다**

## 🏗️ 시스템 아키텍처
<img width="1756" height="1094" alt="image" src="https://github.com/user-attachments/assets/5deb4016-af3b-4306-b6fa-55a84fd9d276" />

### 계층별 구조

```
┌─────────────────────────────────────────────────────────────────┐
│                    Layer 0: Frontend Layer                      │
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │           Reservation Web (React + Vite)                   │ │
│  │  • 사용자 대기열 인터페이스                                      │ │
│  │  • 실시간 상태 업데이트 (WebSocket)                             │ │
│  │  • CloudFront + S3 정적 호스팅                                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓ HTTP/REST + WebSocket
┌─────────────────────────────────────────────────────────────────┐
│                   Layer 1: Gateway Layer                        │
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              Gateway API (Go + Fiber)                      │ │
│  │  • 인증,인가 (JWT OIDC)                                      │ │
│  │  • 대기열 관리 (30k RPS 트래픽 제어)                             │ │
│  │  • 레이트 리미팅 및 멱등성 관리                                   │ │
│  │  • BFF (Backend for Frontend)                              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓ gRPC + REST
┌─────────────────────────────────────────────────────────────────┐
│                 Layer 2: Business Services                      │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────┐ │
│  │ Reservation API  │  │  Inventory API   │  │ Payment Sim    │ │
│  │  (Kotlin/Spring) │  │    (Go/gRPC)     │  │  (Go/gRPC)     │ │
│  │                  │  │                  │  │                │ │
│  │ • 예약 생성/확정    │  │ • 재고 관리         │  │ • 결제 시뮬      │ │
│  │ • 60초 홀드        │  │ • Zero Oversell  │  │ • 웹훅 처리      │ │
│  │ • DynamoDB       │  │ • 좌석 소유권       │  │ • EventBridge  │ │
│  └──────────────────┘  └──────────────────┘  └────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓ Events (EventBridge + SQS)
┌─────────────────────────────────────────────────────────────────┐
│              Layer 3: Background Processing                     │
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │     Reservation Worker (Go + Kubernetes Job)               │ │
│  │  • 예약 만료 처리 (60초 타이머)                                 │ │
│  │  • 결제 승인/실패 후속 처리                                     │ │
│  │  • KEDA Auto-scaling (SQS 기반)                             │ │
│  │  • 보상 트랜잭션 처리                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
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
| **[gateway-api](https://github.com/traffic-tacos/gateway-api)** | Go + Fiber | API Gateway, BFF, 대기열 관리 | 8000 (REST), 8001 (gRPC) |
| **[reservation-api](https://github.com/traffic-tacos/reservation-api)** | Kotlin + Spring Boot WebFlux | 예약 라이프사이클 관리, 60초 홀드 | 8010 (REST), 8011 (gRPC) |
| **[inventory-api](https://github.com/traffic-tacos/inventory-api)** | Go + gRPC | 재고 관리, Zero Oversell 보장 | 8020 (REST), 8021 (gRPC) |
| **[payment-sim-api](https://github.com/traffic-tacos/payment-sim-api)** | Go + gRPC | 결제 시뮬레이션, 웹훅 처리 | 8030 (REST), 8031 (gRPC) |
| **[reservation-worker](https://github.com/traffic-tacos/reservation-worker)** | Go/Kotlin | 백그라운드 작업 (만료/결제 처리) | 8040 (REST), 8041 (gRPC) |
| **[reservation-web](https://github.com/traffic-tacos/reservation-web)** | React + Vite | 프론트엔드 SPA | 3000 |
| **[proto-contracts](https://github.com/traffic-tacos/proto-contracts)** | Protocol Buffers | gRPC 서비스 간 계약 정의 | - |
| **[traffic-tacos-infra-iac](https://github.com/traffic-tacos/traffic-tacos-infra-iac)** | Terraform | AWS 인프라 코드 | - |
| **[deployment-repo](https://github.com/traffic-tacos/deployment-repo)** | Kubernetes + Helm + ArgoCD | GitOps 배포 매니페스트 | - |

## 🛠️ 기술 스택

<div align="center">

### 인프라 (AWS)
![AWS EKS](https://img.shields.io/badge/AWS_EKS-1.33-FF9900?style=for-the-badge&logo=amazon-eks&logoColor=white)
![DynamoDB](https://img.shields.io/badge/DynamoDB-6_Tables-4053D6?style=for-the-badge&logo=amazon-dynamodb&logoColor=white)
![EventBridge](https://img.shields.io/badge/EventBridge-2_Buses-FF4F8B?style=for-the-badge&logo=amazon-aws&logoColor=white)
![ElastiCache](https://img.shields.io/badge/ElastiCache-Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![CloudFront](https://img.shields.io/badge/CloudFront-CDN-8C4FFF?style=for-the-badge&logo=amazon-cloudfront&logoColor=white)

### 백엔드
![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?style=for-the-badge&logo=go&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-2.1+-7F52FF?style=for-the-badge&logo=kotlin&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white)
![gRPC](https://img.shields.io/badge/gRPC-Protocol-244c5a?style=for-the-badge&logo=google&logoColor=white)

### 프론트엔드
![React](https://img.shields.io/badge/React-18+-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-5.0-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0-3178C6?style=for-the-badge&logo=typescript&logoColor=white)

### DevOps & 모니터링
![Terraform](https://img.shields.io/badge/Terraform-1.8+-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Dashboard-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-Traces-000000?style=for-the-badge&logo=opentelemetry&logoColor=white)

</div>

### 상세 스택

<details>
<summary><b>📦 인프라 (AWS)</b></summary>

- **컴퓨팅**: EKS 1.33 (3개 노드 그룹 - ondemand/mix/monitoring)
- **데이터베이스**: DynamoDB (6개 테이블 - reservations, orders, inventory 등)
- **이벤트**: EventBridge (2개 커스텀 버스), SQS (FIFO 큐 + DLQ)
- **캐싱**: ElastiCache Redis (Multi-AZ, 암호화)
- **DNS & SSL**: Route53, ACM (와일드카드 인증서)
- **CDN**: CloudFront + S3 (정적 웹 호스팅)
- **모니터링**: AWS Managed Prometheus, AWS Managed Grafana

</details>

<details>
<summary><b>⚙️ 백엔드</b></summary>

- **API Gateway**: Go 1.22+, Fiber v2
- **Reservation**: Kotlin, Spring Boot 3.x WebFlux, Gradle
- **Inventory**: Go 1.24+, gRPC, Echo
- **Payment Sim**: Go 1.24+, gRPC
- **Worker**: Go/Kotlin, KEDA Auto-scaling

</details>

<details>
<summary><b>🎨 프론트엔드</b></summary>

- **Framework**: React 18+, Vite
- **State**: React Context API / Redux Toolkit
- **Real-time**: WebSocket

</details>

<details>
<summary><b>🔗 프로토콜 & 통신</b></summary>

- **동기**: REST (HTTP/JSON), gRPC (Protocol Buffers)
- **비동기**: EventBridge, SQS
- **인증**: JWT (OIDC/Cognito)
- **관측성**: OpenTelemetry (Trace/Metric/Log)

</details>

<details>
<summary><b>🛠️ 개발 도구</b></summary>

- **IaC**: Terraform 1.8+
- **컨테이너**: Docker, Kubernetes
- **CI/CD**: GitHub Actions, ArgoCD
- **모니터링**: Prometheus(Metric), Grafana(AMG), CloudWatch(Log), X-Ray(Trace)
- **부하테스트**: k6

</details>

## 🔄 핵심 비즈니스 플로우

### 1. 대기열 → 예약 플로우

```
1. 사용자 → POST /api/v1/queue/join
   - 30k RPS 트래픽 유입
   - 대기열 토큰 발급

2. 실시간 위치 업데이트 (WebSocket)
   - GET /api/v1/queue/status
   - 매 2초 예상 대기시간 제공

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

## 🚀 빠른 시작

5분 만에 Traffic Tacos 플랫폼을 로컬에서 실행해보세요!

```bash
# 1. 레포지토리 클론
git clone https://github.com/traffic-tacos/gateway-api.git
cd gateway-api

# 2. 의존성 설치 및 환경 설정
./setup.sh

# 3. 전체 플랫폼 실행
./run_local_all.sh start

# 4. 브라우저에서 확인
open http://localhost:3000
```

### 필수 요구사항
- ✅ Docker & Docker Compose
- ✅ Go 1.22+ / Java 17+ / Node.js 18+
- ✅ AWS CLI (로컬 테스트용)

## 🛠️ 개발 환경 설정

### 전제 조건

- Docker & Docker Compose
- Go 1.22+
- Java 17+ (Kotlin)
- Node.js 18+
- Terraform 1.8+
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
cd traffic-tacos-infra-iac
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

- [인프라 구성 가이드](https://github.com/traffic-tacos/traffic-tacos-infra-iac/README.md)
- [API 계약 문서](https://github.com/traffic-tacos/proto-contracts/README.md)
- [DynamoDB 스펙](https://github.com/traffic-tacos/traffic-tacos-infra-iac/docs/spec/dynamodb-spec.md)
- [EventBridge 스펙](https://github.com/traffic-tacos/traffic-tacos-infra-iac/docs/spec/eventbridge-spec.md)
- [개발 수칙](https://github.com/traffic-tacos/gateway-api/.cursor/rules/rules-must-be-followed.mdc)

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

## 👏 기여자

<div align="center">

Traffic Tacos는 열정적인 팀원들의 협업으로 만들어지고 있습니다.

### 전체 기여 현황

| 레포지토리 | 주요 기여자 |
|-----------|------------|
| [gateway-api](https://github.com/traffic-tacos/gateway-api) | ![Contributors](https://img.shields.io/github/contributors/traffic-tacos/gateway-api?style=flat-square) |
| [reservation-api](https://github.com/traffic-tacos/reservation-api) | ![Contributors](https://img.shields.io/github/contributors/traffic-tacos/reservation-api?style=flat-square) |
| [inventory-api](https://github.com/traffic-tacos/inventory-api) | ![Contributors](https://img.shields.io/github/contributors/traffic-tacos/inventory-api?style=flat-square) |
| [payment-sim-api](https://github.com/traffic-tacos/payment-sim-api) | ![Contributors](https://img.shields.io/github/contributors/traffic-tacos/payment-sim-api?style=flat-square) |
| [reservation-worker](https://github.com/traffic-tacos/reservation-worker) | ![Contributors](https://img.shields.io/github/contributors/traffic-tacos/reservation-worker?style=flat-square) |
| [reservation-web](https://github.com/traffic-tacos/reservation-web) | ![Contributors](https://img.shields.io/github/contributors/traffic-tacos/reservation-web?style=flat-square) |

**총 커밋 수**: ![Total Commits](https://img.shields.io/badge/dynamic/json?color=blue&label=commits&query=%24.totalCommits&url=https%3A%2F%2Fapi.github.com%2Forgs%2Ftraffic-tacos%2Frepos&style=flat-square)

📊 [상세 기여 통계 보기](https://github.com/orgs/traffic-tacos/people) | 💻 [레포지토리 둘러보기](https://github.com/orgs/traffic-tacos/repositories)

</div>

## 💬 문의 및 지원

### 이슈 트래커
버그 리포트나 기능 요청은 [GitHub Issues](https://github.com/traffic-tacos/.github/issues)에 등록해주세요.

### 커뮤니케이션 채널
- 📧 **Email**: xnslqjtmghaf@gmail.com

### 보안 취약점 제보
보안 관련 이슈는 **security@traffic-tacos.dev**로 비공개로 제보해주세요.

## ⚖️ 라이선스

이 프로젝트는 **Internal Use Only** 라이선스로 배포됩니다.

```
Copyright (c) 2025 Traffic Tacos Team
All Rights Reserved.

본 소프트웨어 및 관련 문서 파일("소프트웨어")은 Traffic Tacos 팀의 
내부 사용 목적으로만 제공됩니다. 무단 복제, 배포, 수정, 또는 
상업적 사용을 금지합니다.
```

### 오픈소스 컴포넌트
이 프로젝트는 다음 오픈소스 라이브러리를 사용합니다:
- Go Fiber (MIT License)
- Spring Boot (Apache 2.0)
- React (MIT License)
- Terraform (MPL 2.0)
- Kubernetes (Apache 2.0)

## 🙏 감사의 말

Traffic Tacos 프로젝트에 기여해주신 모든 분들께 감사드립니다.

---

<div align="center">

**Traffic Tacos** 🌮

고성능 · 안정성 · 확장성을 모두 갖춘 Cloud-Native 티켓 예약 플랫폼

Made with ❤️ by [Traffic Tacos Team](#-팀-구성원)

[⬆ 맨 위로](#traffic-tacos-)

</div>
