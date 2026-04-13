# Team-LMS Portfolio (Fullstack / Infra)

> (공공 SI) **관리자/학생/교수 LMS 플랫폼** 팀 프로젝트 포트폴리오입니다.  
> 원본 코드는 **팀 Org 레포**에 있으며, 이 레포지토리는 본인의 **풀스택 개발 및 인프라 설계 기여도**를 중심으로 요약한 개인 포트폴리오 페이지입니다.

## Service (Live)
- https://teamlms.duckdns.org
- 👉 [Demo Accounts](#demo-accounts)

---

## Links
- Org Repository (Source): https://github.com/team-lms-2026-1/LMS-project
- Portfolio Repo (this): https://github.com/essee2001-afk/lms-portfolio
- Figma (prototype, view-only): https://www.figma.com/proto/RxRjps2RteNVlH6J0vTxx9/Team-LMS-FIGMA-?node-id=17-20&t=CECAPJa6TQnkEFcM-1
- Use Case Diagram (draw.io): https://drive.google.com/file/d/1vGpn3qZlbVyoDKWs38B_-1B4RwdgZc89/view?usp=sharing

---

## Demo Video

[![Demo Video](assets/demo-thumbnail.png)](https://youtu.be/Cv6gBFy2WnE)

---

## My Role / Contribution
**개발 포지션: Fullstack & Infrastructure**

### 1. Infrastructure & DevOps 
- 단일 EC2 기반의 Nginx + Docker(Next.js, Spring Boot) 운영 환경 아키텍처 구축
- Frontend/Backend 도메인 특성에 맞춘 이원화된 GitHub Actions CI/CD 파이프라인 자동화
- AWS 서비스(RDS PostgreSQL, S3) 연동 및 VPC/Security Group 통합 격리망 구축
- Flyway를 활용한 DB 스키마(Migration) 버전 관리 설계
- Next.js 기반 BFF(Backend-For-Frontend) 패턴을 통한 보안 통신(HttpOnly Cookie) 구조 확립

### 2. Fullstack Development (Backend + Frontend)
- **설문조사(Surveys):** 문항 생성 로직 설계, 답변 수집 및 통계 대시보드 풀스택 연동
- **멘토링(Mentoring):** 교수-학생 간 멘토링 세션 신청/승인/내역 추적에 대한 도메인 로직 및 UX 개발
- **비밀번호 재설정:** 학교 계정 이메일을 입력하면 해당 메일로 보안 토큰이 포함된 재설정 권한 링크를 자동 발송하여 안전하게 비밀번호를 초기화하고 변경하는 플로우 구현

### 3. Frontend Development
- **계정 로그 관리(Account Log):** 관리자 환경에서의 사용자 접속 시점 및 활동 내역(Audit) 조회 화면 모듈화
- **학과 관리(Department Admin):** 직관적인 인터페이스의 학과 CUD (생성/수정/삭제) 및 목록 확인 페이지 컴포넌트 개발

---

## Technical Highlights
특히 **안정적인 시스템 인프라 시나리오 구축** 및 **BFF 패턴을 통한 인증/보안 아키텍처 설계**에 힘을 쏟았습니다.

### 1. 운영 환경을 고려한 프론트/백엔드 맞춤형 CI/CD 이원화 구축
단순 획일적인 파이프라인 구성에서 벗어나, 런타임 특성이 다른 두 서비스의 빌드 속도 및 안정성을 최적화했습니다.
- **Frontend 배포 (`frontend-deploy.yml`)**: Node.js 환경의 의존성을 완벽히 통제하기 위해 **멀티 스테이지 도커 빌드(Multi-stage build)**를 활용. 실행에 필요한 파일들만 압축한 단일 이미지를 빌드하여 `GHCR`에 배포하고, EC2가 이를 풀(Pull)하여 실행하게 구현.
- **Backend 배포 (`deploy.yml`)**: 매번 무거운 이미지를 굽는 낭비를 막기 위해, 로컬(CI환경)에서 컴파일된 `app.jar` 만 EC2로 가볍게 넘기는 **SCP 전송 방식**을 채택. 가벼운 공식 자바 도커 이미지(`eclipse-temurin`)에 실행 파일만 갈아 끼워 실행시켜 **배포 소요 시간을 극적으로 단축**했습니다.

### 2. BFF (Backend-For-Frontend) 채택으로 토큰 관리/보안 원천 강화
로컬 스토리지에 JWT 토큰을 두는 흔한 방식의 인증/탈취 취약성과 CORS 정책을 해결하기 위해, Next.js 프론트엔드 환경을 BFF 레이어로 승격시켜 활용했습니다.
- 사용자는 `HttpOnly Cookie` 형태로만 토큰을 갖게 되며 브라우저(JS)에서 접근할 수 없어 탈취 위험을 없앴습니다.
- 브라우저는 BFF 역할을 하는 Next.js API 도메인과 통신하여 **CORS 정책 오류를 피하고**, Next.js 라우트가 쿠키에서 토큰을 추출해 백엔드 API 요청 시 `Authorization: Bearer <token>` 헤더를 서버 투 서버(Server-to-Server) 로 꽂아주는 보안 브릿지 역할을 구현해 냈습니다.

### 3. 단일 서버 컨테이너 네트워크 리소스 격리 통제
- EC2 인바운드 보안그룹은 가장 필수적인 22(SSH), 80(HTTP), 443(HTTPS) 포트만 열어두고, Nginx 설정에서 프록시 서버만 통과하게 하여 **가상 서버의 애플리케이션 서비스 포트(`3000`, `8080`) 노출을 완벽히 차단**했습니다.
- 도커 내부에 `lms-network`라는 프라이빗 브릿지 가상망을 구성, Next.js 화면과 Spring Boot 서버가 오직 EC2 내부망에서만 서로를 바라보며 데이터 흐름을 제한하도록 조치했습니다.
- **AWS RDS 또한 인바운드를 EC2 내부의 특정 보안그룹 전용으로 한정**하여 데이터베이스 외부 해킹 시도를 봉쇄했습니다.

### 4. Flyway를 활용한 DDL 마이그레이션 관리 자동화
개발 및 유지보수 과정에서 테이블 및 컬럼 변경이 잦은 점을 고려해 수동 쿼리 실행으로 인한 버전 불일치 휴먼 에러를 방지했습니다. 
- `Flyway` 모듈을 연동해 `.sql` 버전 관리가 형상관리(Git)와 함께 물리도록 구성. GitHub Actions 백엔드 빌드 후 실행 파일이 교체되어 서버가 부트스트랩될 때 DB 스키마가 로직에 맞게 스스로 최신 상태 코드로 마이그레이션되도록 자동화했습니다.

---

## 🏗️ Architecture & Infrastructure Structure

- 전체 배포 파이프라인 및 통신 흐름 다이어그램 등 상세 아키텍처는 별도 문서에 정리되어 있습니다.
- **[👉 Architecture & Infrastructure 상세 보기](architecture.md)**

---

## Tech Stack

### Backend
| 기술 | 버전 | 채용 이유 |
|---|---:|---|
| Java | 17 | LTS 기반 안정적인 서버 런타임 |
| Spring Boot | 3.5.9 | 표준 백엔드 프레임워크 |
| Spring Data JPA | - | JpaRepository + 파생 메서드 + `@Query` 중심 조회/CRUD |
| Querydsl | - | 복잡한 검색/동적 조건 조회에 선택 적용 |
| Spring Security + JWT | - | Stateless 인증 구성 |
| RBAC (2계층 권한) | - | URL Role + Method Authority 이중 접근 제어 |
| PostgreSQL | - | 운영 안정성 높은 RDB |
| Flyway + ddl validate | - | 마이그레이션/스키마 검증 기반 배포 안정성 |
| AWS S3 (Presigned URL) | - | 파일 업로드/다운로드 처리 |
| Spring AI + OpenAI | gpt-4o-mini | MBTI 추천 AI (schema 제약 + fallback 적용) |
| OpenAPI (Swagger UI) | - | API 문서화/테스트, 협업 커뮤니케이션 향상 |

### Frontend / BFF
| 기술 | 버전 | 채용 이유 |
|---|---:|---|
| Next.js (BFF) | 14.2.5 | BFF 계층으로 API 집계/인증 처리 |
| HttpOnly Cookie Auth | - | 토큰 노출 최소화(클라이언트 저장소 회피) |
| next-intl | - | 다국어 지원(ko/en/ja) |
| Chart.js / Recharts | - | 대시보드/통계 시각화 |

### Infrastructure / DevOps
| 기술 | 버전 | 채용 이유 |
|---|---:|---|
| AWS EC2 | - | 서비스 호스팅(단일 서버) |
| Nginx + Let's Encrypt | - | HTTPS(SSL 종료) + Reverse Proxy |
| AWS RDS (PostgreSQL) | - | 운영 DB |
| AWS S3 | - | 파일 저장(Presigned URL) |
| Docker | - | 컨테이너 기반 실행/배포 |
| Docker Compose | - | 로컬 개발환경 재현 |
| GitHub Actions | - | 배포 자동화(Backend 빌드 중심, 테스트 스킵/프론트 CI 제한) |

---

## UI Screenshots (My Works)

### 1. 설문(Survey) 관리 화면
| 목록 및 생성 | 상세 |
|---|---|
| ![Survey1](assets/survey.png) | ![Survey2](assets/survey2.png) |

### 2. 멘토링(Mentoring) 화면
| 멘토링 승인 및 내역 | 멘토링 채팅 |
|---|---|
| ![Mentoring1](assets/mentoring.png) | ![Mentoring2](assets/mentoring2.png) |

### 3. 비밀번호 재설정 화면
| 이메일 링크 발송 | 비밀번호 변경 |
|---|---|
| ![Password1](assets/password.png) | ![Password2](assets/password2.png) |


---

## ERD
- 상세 ERD 문서: [ERD 상세 보기](docs/ERD.md)

---

## Demo Accounts
> 데모용 계정이며 비밀번호는 운영상 변경될 수 있습니다.

- **Admin**: `a20001122` / `Admin!2345`
- **Professor**: `p20260001` / `Professor!2345`
- **Student**: `s20260001` / `Student!2345`
