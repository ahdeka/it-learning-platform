# ADR-0003: 기본 기술 스택

- 상태: 확정 (버전 번호는 프로젝트 생성 시점에 고정)
- 결정일: 2026-11-04

## 배경

뼈대를 만들기 위해 언어, 프레임워크, 빌드 도구, DB를 정해야 한다. Java / Spring Boot와 Next.js는 프로젝트 전제로 이미 정해져 있다.

## 결정

| 영역 | 선택 | 근거 |
|---|---|---|
| Java | **25 (LTS)** | 현재 LTS. Spring Boot 4.x가 지원 |
| Spring Boot | **4.x 최신 안정 버전** | 2026-11 시점에는 4.1.x가 최신 안정, 4.2가 2026-11 출시 예정. **생성 당일 start.spring.io에서 확인 후 고정** |
| 빌드 | **Gradle (Kotlin DSL)** | 요즘 표준. 일본 현장에서 Maven을 쓰더라도 개념은 동일 |
| Frontend | **Next.js 16.x + TypeScript (App Router)** | 프로젝트 전제 |
| DB | **PostgreSQL** | 무료, Docker로 바로 실행. JSONB로 문제 메타데이터를 유연하게 저장 가능. 대기열 구현에 `SKIP LOCKED` 사용 ([ADR-0005](ADR-0005-db-backed-queue.md)) |
| DB 마이그레이션 | **Flyway** | 스키마 변경 이력이 SQL 파일로 남음. 테이블 정의서 작성 근거가 됨 |
| 데이터 접근 | Spring Data JPA (복잡한 조회는 필요 시 별도 검토) | 표준 |
| API 문서 | springdoc-openapi (Swagger UI) | 설계서(Markdown)와 실제 구현 대조용 |
| 터미널 UI | xterm.js | 브라우저 터미널 사실상 표준 |
| Docker 제어 | docker-java (runner에서만) | Spike에서 검증 |
| 로컬 인프라 | docker-compose | 배포 시 이식성 ([NFR-OPS-001](../../01-requirements/non-functional-requirements.md)) |

## 나중에 정할 것

- 인증 방식 ([ADR-0008](ADR-0008-authentication-deferred.md))
- CSS 프레임워크 / UI 라이브러리
- CI/CD, 모니터링, 배포 환경

## 결과와 영향

- 버전은 프로젝트 생성 시 `build.gradle.kts`, `package.json`에 고정하고, 이 문서에 실제 버전을 기록한다.
