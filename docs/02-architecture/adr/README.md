# ADR (Architecture Decision Record)

"왜 이렇게 했는가"를 남기는 문서입니다. 결정 하나당 파일 하나, 짧게 씁니다.

## 목록

| 번호 | 제목 | 상태 |
|---|---|---|
| [ADR-0001](ADR-0001-docs-as-code.md) | 설계 문서는 레포 안 Markdown을 원본으로 한다 | 확정 |
| [ADR-0002](ADR-0002-monorepo.md) | 모노레포 구조 | 확정 |
| [ADR-0003](ADR-0003-base-tech-stack.md) | 기본 기술 스택 | 확정 (버전은 생성 시점에 고정) |
| [ADR-0004](ADR-0004-separate-runner.md) | 컨테이너 실행을 별도 runner 프로세스로 분리 | 확정 |
| [ADR-0005](ADR-0005-db-backed-queue.md) | 실행 대기열은 PostgreSQL 테이블로 시작 | 확정 |
| [ADR-0006](ADR-0006-guest-first-actor-model.md) | 게스트 우선 이용 정책과 actor 모델 | 확정 |
| [ADR-0007](ADR-0007-deployment-strategy-deferred.md) | 배포 전략 | 보류 |
| [ADR-0008](ADR-0008-authentication-deferred.md) | 인증 방식 (세션 / JWT) | 보류 |

## 상태 값

- **제안(Proposed)**: 논의 중
- **확정(Accepted)**: 합의 완료
- **보류(Deferred)**: 의도적으로 결정을 미룸
- **대체됨(Superseded by ADR-XXXX)**: 새 결정으로 바뀜. 기존 파일은 지우지 않는다

## 템플릿

새 ADR은 [template.md](template.md)를 복사해서 씁니다.
