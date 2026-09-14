# 🎂 MakeAWish: Custom Cake O2O Commerce Platform

<p align="center">
  <img src="https://img.shields.io/badge/Platform-O2O%20Custom%20Cake-FF69B4?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Architecture-Event--Driven%20Microservices-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Mobile-React%20Native%20Expo-000020?style=for-the-badge&logo=expo&logoColor=white" />
  <img src="https://img.shields.io/badge/Web-React%2019%20Vite-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/Backend-Spring%20Boot%203.2-6DB33F?style=for-the-badge&logo=springboot&logoColor=white" />
  <img src="https://img.shields.io/badge/AI-FastAPI%20%26%20Gemini-4285F4?style=for-the-badge&logo=google-gemini&logoColor=white" />
</p>

MakeAWish(메이크어위시)는 수제 맞춤형 케이크 주문 과정에서 발생하는 상담 피로도와 제작 소통 미스를 해결하기 위한 O2O 커머스 플랫폼입니다. 소비자의 자연어 요청을 분석하는 대화형 주문서 생성, 케이크 실물 질감을 보존하는 도안 인페인팅, 소상공인용 실시간 주문 관리 웹 및 선결제 에스크로 시스템을 제공합니다.

---

## 🏗 전체 시스템 아키텍처 (System Architecture)

```mermaid
flowchart TD
    subgraph Layer1 ["📱 Client Layer (사용자 접점)"]
        App["📱 소비자 모바일 앱<br/><b>MakeAWish-FE</b><br/>React Native · Expo SDK 54"]
        Web["💻 사장님 관리자 웹<br/><b>MakeAWish-FE-Owner</b><br/>React 19 · Vite · Zustand"]
    end

    subgraph Layer2 ["⚙️ Microservice Core (애플리케이션 계층)"]
        Spring["⚙️ 메인 비즈니스 서버<br/><b>MakeAWish-BE</b><br/>Spring Boot 3.2 (AWS EB)<br/>STOMP 채팅 · SSE 알림 · 토스 결제"]
        AI["🧠 AI 마이크로서비스<br/><b>MakeAWish-AI</b><br/>FastAPI · Gemini Flash<br/>멀티모달 인페인팅 · 동적 슬롯필링"]
    end

    subgraph Layer3 ["🗄️ Persistence & Storage (클라우드 인프라)"]
        RDS[("🗄️ 메인 데이터베이스<br/><b>AWS RDS (MySQL 8.0)</b><br/>RDB + JSON 하이브리드")]
        S3[("📦 오브젝트 스토리지<br/><b>AWS S3 Bucket</b><br/>도안 · 마스크 · 포토 리뷰")]
    end

    %% 연결 관계
    App <==>|"REST / WSS (채팅)"| Spring
    Web <==>|"REST / SSE (주문알림)"| Spring
    Spring <==>|"비동기 WebClient / Webhook"| AI
    Spring -->|"Spring Data JPA"| RDS
    Spring -->|"Presigned URL"| S3
    App -.->|"도안 조회 / 업로드"| S3
    AI -.->|"생성 도안 저장"| S3

    %% 모던 테크 파스텔 스타일링
    classDef client fill:#EBF5FF,stroke:#3B82F6,stroke-width:2px,color:#1E3A8A;
    classDef backend fill:#ECFDF5,stroke:#10B981,stroke-width:2px,color:#064E3B;
    classDef ai fill:#F5F3FF,stroke:#8B5CF6,stroke-width:2px,color:#4C1D95;
    classDef storage fill:#FFFBEB,stroke:#F59E0B,stroke-width:2px,color:#78350F;
    classDef layer fill:#F8FAFC,stroke:#CBD5E1,stroke-width:1.5px,stroke-dasharray: 4 4,color:#475569;

    class Layer1,Layer2,Layer3 layer;
    class App,Web client;
    class Spring backend;
    class AI ai;
    class RDS,S3 storage;
```

---

## 📦 서브 프로젝트 저장소 개요

| 프로젝트 디렉토리 | 담당 플랫폼 및 역할 | 핵심 기술 스택 | 세부 README |
| :--- | :--- | :--- | :---: |
| **`MakeAWish-FE`** | **소비자 모바일 앱** (탐색, AI 도안 생성, 결제, 1:1 상담) | React Native 0.76, Expo SDK 54, Expo Router v3, NativeWind v4, Naver Map SDK, Toss Payments | [바로가기](./MakeAWish-FE/README.md) |
| **`MakeAWish-FE-Owner`** | **사장님 관리자 웹** (실시간 주문 칸반, 동적 스키마 빌더, AI 답글) | React 19, Vite, Zustand, Tailwind CSS, SSE (`text/event-stream`), Recharts | [바로가기](./MakeAWish-FE-Owner/README.md) |
| **`MakeAWish-BE`** | **메인 API 오케스트레이터** (주문 무결성, 하버사인 반경 검색, 결제 멱등성) | Java 17, Spring Boot 3.2, Spring Security 6, JPA/Hibernate, MySQL 8.0, STOMP WS | [바로가기](./MakeAWish-BE/README.md) |
| **`MakeAWish-AI`** | **AI 마이크로서비스** (동적 슬롯필링, 실물 질감 보존 인페인팅, 3단계 파서) | Python 3.10+, FastAPI, Pydantic v2, Google Gemini 3.5 Flash, Gemini 멀티모달 Inpainting | [바로가기](./MakeAWish-AI/README.md) |

---

## 🔄 End-to-End 핵심 주문 라이프사이클

1. **도안 생성 & 슬롯필링**: 소비자가 모바일 앱에서 자연어로 요청하면 AI(FastAPI)가 매장 고유의 주문서 양식(`custom_schema`)에 맞추어 필수 옵션을 추출하고 주문 요약 카드를 생성합니다.
2. **실시간 주문 접수**: 사장님 관리자 웹은 SSE(`SseEmitter`)를 통해 페이지 새로고침 없이 즉시 칸반 보드 [신규 주문] 컬럼에 주문 카드를 수신합니다.
3. **추가금 산정 & 견적 발송**: 사장님이 도안 난이도 및 요청사항을 확인한 후 작업 추가금을 책정하여 최종 견적서를 발송합니다.
4. **토스페이먼츠 선결제**: 소비자 앱의 1:1 상담 채팅방에 견적 카드가 갱신되며, 인앱 웹뷰 브릿지를 통해 토스페이 선결제를 진행합니다.
5. **제작 착수 & 노쇼 방지**: 결제 완료 상태(`PAID`)를 확인한 사장님이 안심하고 제작(`MAKING`)에 착수하여 노쇼를 원천 차단합니다.
