# API 설계

- 상태: 초안
- 범위: MVP API 목록과 공통 규칙. 각 API의 요청/응답 상세는 구현 직전에 `04-api/` 아래 별도 파일로 작성한다.
- 구현 후에는 springdoc-openapi(Swagger UI)로 자동 생성된 명세와 이 문서를 대조한다.

## 1. 공통 규칙

### URL / 메서드
- 기본 경로: `/api`
- 리소스는 복수형 명사: `/api/problems`, `/api/sessions`
- 조회 `GET`, 생성 `POST`, 삭제 `DELETE`. 부분 수정이 필요해지면 `PATCH`

### 사용자 식별
- 모든 요청은 **actor** 기준으로 처리한다 ([ADR-0006](../02-architecture/adr/ADR-0006-guest-first-actor-model.md)).
- 게스트: 쿠키의 게스트 토큰으로 식별. 쿠키가 없으면 첫 요청에서 발급
- 회원: 인증 방식 보류 ([ADR-0008](../02-architecture/adr/ADR-0008-authentication-deferred.md))
- 게스트 쿠키 속성: `HttpOnly`, `SameSite=Lax`, (배포 시) `Secure`

### 에러 응답
Spring의 `ProblemDetail`(RFC 9457 형식)을 사용한다.

```jsonc
{
  "type": "https://example.com/problems/guest-limit-exceeded",
  "title": "Guest limit exceeded",
  "status": 429,
  "detail": "오늘 게스트 실습 횟수를 모두 사용했습니다.",
  "code": "GUEST_LIMIT_EXCEEDED"
}
```

- `code`는 프론트가 분기 처리할 때 쓰는 고정 문자열이다.

### 주요 에러 코드 (초안)

| code | HTTP | 의미 |
|---|---|---|
| `PROBLEM_NOT_FOUND` | 404 | 문제 없음 |
| `GUEST_LIMIT_EXCEEDED` | 429 | 게스트 실행 한도 초과 |
| `SESSION_ALREADY_ACTIVE` | 409 | 이미 실행 중인 실습 세션이 있음 (게스트 동시 1개) |
| `SESSION_NOT_OWNED` | 403 | 다른 actor의 세션에 접근 |
| `SESSION_NOT_RUNNING` | 409 | 실행 중이 아닌 세션에 제출 |

## 2. API 목록 (IF一覧) — MVP

### 게스트 / 진행 기록

| ID | 메서드 | 경로 | 설명 | 관련 |
|---|---|---|---|---|
| API-ACT-001 | GET | `/api/me` | 현재 actor 정보 (게스트 여부, 남은 실행 횟수) | FR-GST-001 |
| API-ACT-002 | GET | `/api/me/progress` | 푼 문제, 테스트 결과 | FR-PRG-001 |

### 몸풀기 / 실력 테스트

| ID | 메서드 | 경로 | 설명 | 관련 |
|---|---|---|---|---|
| API-TST-001 | POST | `/api/placement-tests` | 테스트 시작 (문제 세트 반환) | FR-TST-001, 002 |
| API-TST-002 | POST | `/api/placement-tests/{testId}/answers` | 문항 답안 제출 | FR-TST-002 |
| API-TST-003 | GET | `/api/placement-tests/{testId}/result` | 결과 (수준 판정) | FR-TST-003 |

### 문제

| ID | 메서드 | 경로 | 설명 | 관련 |
|---|---|---|---|---|
| API-PRB-001 | GET | `/api/problems` | 목록 (카테고리, 난이도, 유형 필터) | FR-PRB-001 |
| API-PRB-002 | GET | `/api/problems/{slug}` | 상세 | FR-PRB-001 |
| API-PRB-003 | POST | `/api/problems/{slug}/submissions` | 퀴즈 답안 / 플래그 제출 / 상태 검사 요청 | FR-JDG-001~003 |

### 실습 세션 (터미널)

| ID | 메서드 | 경로 | 설명 | 관련 |
|---|---|---|---|---|
| API-SES-001 | POST | `/api/problems/{slug}/sessions` | 실습 시작 → 대기열 등록. 세션 ID와 상태(QUEUED) 반환 | FR-TRM-001, 002, 005 |
| API-SES-002 | GET | `/api/sessions/{sessionId}` | 세션 상태 (QUEUED / STARTING / RUNNING / ENDED), 대기 순번, 남은 시간 | FR-TRM-002 |
| API-SES-003 | DELETE | `/api/sessions/{sessionId}` | 실습 종료 (컨테이너 회수) | FR-TRM-004 |
| API-SES-004 | WS | `/ws/sessions/{sessionId}/terminal` | 터미널 입출력 스트림 | FR-TRM-003 |

## 3. 설계 메모

### 제출 API를 하나로 둔 이유
퀴즈 답안, 플래그, 상태 검사는 모두 "이 문제에 대한 답을 냈다"는 같은 행위다. 채점 방식(judge_type)은 문제 데이터가 결정하므로 API는 하나로 두고, 요청 본문만 유형별로 다르게 한다.

```jsonc
// 퀴즈 (객관식)
{ "choiceId": 3 }
// 퀴즈 (단답형) 또는 플래그
{ "answer": "FLAG{...}" }
// 상태 검사형 (본문 없이 세션만 지정)
{ "sessionId": "..." }
```

### 상태 조회를 폴링으로 시작하는 이유
대기 순번은 몇 초 단위로만 바뀌므로 MVP에서는 `GET /api/sessions/{id}` 폴링으로 충분하다. 터미널 연결에는 어차피 WebSocket을 쓰므로, 필요해지면 상태 알림도 WebSocket으로 옮긴다.

### WebSocket 보안 체크리스트
- 핸드셰이크 시 세션 소유 actor 확인 ([NFR-SEC-030](../01-requirements/non-functional-requirements.md))
- `Origin` 헤더 검사 (허용 목록 방식)
- 세션 상태가 RUNNING일 때만 연결 허용
- 입력 메시지 크기 제한 (대용량 붙여넣기로 서버 메모리를 소모시키는 것 방지)
