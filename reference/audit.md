# audit

> 단방향 의존성 위반·레이어 누수 등 기계적으로 잡을 수 있는 패턴을 일괄 체크. 카테고리: Evaluate.

## 사용 시점

- CI에 포함하고 싶은 정적 검증.
- 정기 점검 (주간 / 월간).
- [[document]] 결과의 위반 신호를 깊이 진단할 때.

## 입력

- 검사 범위(전체 또는 일부 경로).

## 절차

체크리스트:

1. (TBD) `module/*` → `app/*` import: 금지.
2. (TBD) `module/<a>/` → `module/<b>/` import: 금지.
3. (TBD) `shared/` → `app/*` 또는 `module/*` import: 금지.
4. (TBD) `app/*/<resource>` ↔ `app/*/<other-resource>` 직접 import: 금지.
5. (TBD) Repository가 `domain/`이 아닌 곳에 선언: 누수.
6. (TBD) Query Service가 `module/` 안에 있음: 위치 위반.
7. (TBD) Entity가 audit 필드(`createdAt`, `createdBy`, `deletedAt`)를 직접 보유: 컨벤션 위반.
8. (TBD) Controller에 비즈니스 분기 로직: 책임 위반.
9. (TBD) Application Service에 SQL 또는 ORM 직접 호출: 책임 위반.
10. (TBD) `tx` 객체가 서비스 시그니처에 노출: UoW 캡슐화 위반.

## 출력

- 위반 목록 (파일·라인·룰 ID·심각도).
- P0~P3 우선순위.

## 관련

- [[critique]] - 정성적 리뷰.
- [[refactor]] - 위반 교정.

## 프레임워크 노트

- NestJS: ESLint + `eslint-plugin-boundaries` 또는 `dependency-cruiser`로 1~4번 룰 자동화 권장.
- Spring: ArchUnit으로 패키지 의존성 룰 강제.
- ASP.NET Core: NetArchTest 또는 ArchUnitNET.
