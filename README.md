# (가칭) IT 학습 플랫폼

브라우저에서 리눅스 환경을 띄워 인프라·보안·해킹 문제를 직접 풀어보는 실습형 IT 학습 커뮤니티.

> **현재 상태**: 설계 및 로컬 개발 단계 (배포 계획 없음)

## 핵심 기능

- **웹 터미널 실습**: 문제마다 격리된 Docker 컨테이너를 띄워 브라우저에서 리눅스 명령어로 문제 해결
- **몸풀기 / 실력 테스트**: 회원가입 없이 바로 시작해서 내 수준 확인
- **레벨 시스템**: 문제를 풀면 레벨 상승
- 부가 기능: IT 뉴스(RSS 수집), 게시판, 코딩테스트

## 디렉터리 구조

```
.
├── backend/   # Spring Boot API 서버 (사용자 요청 처리, 문제·채점·진행 기록)
├── runner/    # 컨테이너 실행 전담 프로세스 (Docker 제어 권한은 여기만 가짐)
├── frontend/  # Next.js 웹 클라이언트
├── infra/     # 로컬 개발용 docker-compose, 문제용 이미지 정의
└── docs/      # 설계 문서 (기획서, 요구사항, ADR, API, DB, WBS)
```

## 문서

설계 문서는 [`docs/`](docs/README.md)에서 관리합니다. 코드와 같은 커밋으로 함께 갱신하는 것을 원칙으로 합니다.

## 기술 스택

| 영역 | 기술 |
|---|---|
| Backend / Runner | Java 25, Spring Boot 4.x, Gradle (Kotlin DSL) |
| Frontend | Next.js 16.x, TypeScript |
| DB | PostgreSQL, Flyway |
| 실습 환경 | Docker |
| 로컬 인프라 | docker-compose |

선택 근거는 [ADR](docs/02-architecture/adr/) 참고.
