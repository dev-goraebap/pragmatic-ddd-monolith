# project-convention

> 이 스킬이 의도적으로 위임하는 항목과 그것을 담는 `PROJECT.md` 템플릿. 스킬은 구조(참조 그래프)만 강제하고, 아래는 프로젝트마다 다르므로 팀이 정한다.

## 언제 펼치는가

- 폴더명·파일 suffix 등 명명 규칙을 스킬 기본값과 다르게 가져갈 때.
- 에러 모델, 테스트 작성 택틱, soft-delete, audit 컬럼 등 디테일을 확정할 때.
- 신규 프로젝트의 `PROJECT.md`를 처음 세팅할 때.

---

## 위임 원칙

스킬이 강제하는 것은 구조다 — 세 레이어 + 단방향 참조 그래프, 슬라이스 격리, 배치 휴리스틱, DIP 기준(테스트 더블 슬롯), 얕은 CQRS, 폴더→테스트 종류 매핑.

그 위에서 팀이 자유롭게 고르는 모든 디테일은 프로젝트가 정한다. 스킬은 "이게 정답"이라 말하지 않는다. 스킬 기본값(`app`/`module`/`shared`, `app/shared`, `module/shared` 등)을 그대로 써도 되고, PROJECT.md로 덮어써도 된다.

## 위임 항목

### 1. 폴더·파일 명명
- 레이어·공용 폴더명 오버라이드 (`shared` → `common` 등).
- 파일 suffix (`*.service.ts` / `*.controller.ts` / `*-query.service.ts` 또는 다른 규약).
- 슬라이스 내부 정돈 방식 — 역할 폴더 vs 플랫 + 파일명 규약.

### 2. 에러 모델 (완전 위임)
스킬은 도메인 에러가 HTTP를 몰라야 하고 전역 지점에서 매핑된다는 위치 원리만 둔다. 그 외는 전부 팀이 정한다.

- 분류 체계 (예: 400/401/403/404/409 5분류를 쓸지, 더 세분할지).
- 에러 코드 명명 (`<BC>_<CODE>` / `bc.code` / SCREAMING_SNAKE 등).
- wire 응답 포맷 (`{ code, message }` / RFC 9457 Problem Details 등).
- 메시지 다국어 정책.
- 도메인 에러 베이스 클래스 위치·명명.

### 3. 테스트 작성 택틱
폴더→테스트 종류 매핑(Solitary/Sociable/Integration)은 스킬이 정하지만, 작성법은 팀이 정한다.

- 테스트 파일 배치 — co-located vs 별도 `__tests__/`.
- Given/When/Then 주석 규약, 네이밍.
- 인메모리 Stub vs Mock 라이브러리 선택 기준.
- 빌드 격리 기법 (`tsconfig.build.json` exclude / 별도 테스트 프로젝트 등).

### 4. 영속성·도메인 디테일
- soft-delete 채택 여부와 `find*` 기본 동작(활성만 vs 전체), 아카이브 포함 조회 메서드 명명.
- batch 메서드 이름 (`saveAll` / `saveBatch` / `bulkInsert`).
- audit 컬럼 이름 (`createdAt`/`updatedAt`/`createdBy`/`deletedAt`의 표기).
- 도메인 vs ORM 엔티티 — 분리 경로 vs 단일 경로 선택.
- 추상 식별자 표현 (TS `abstract class` vs JVM/C# `interface`).
- 검증 라이브러리 (Zod/class-validator / Bean Validation / FluentValidation 등).

### 5. app 잔여 중복 처리
[app-layer.md](./app-layer.md) §6의 "도메인으로 내림"을 배제한 뒤 남는 순수 코드 중복을 어떻게 풀지(OOP 합성, 헬퍼 추출 등)는 팀의 기술적 판단이다.

---

## PROJECT.md 템플릿

```markdown
# PROJECT.md

## Bounded Contexts
- <bc-name>: <책임 한 줄>
- ...

## 도메인 용어집
- <용어>: <정의>

## 명명 규칙 (스킬 기본값 오버라이드)
- 레이어 폴더: app / module / shared   (기본값 유지)
- 공용 폴더: app/shared, module/shared, shared/<fragment>
- 파일 suffix: *.controller, *.service, *-query.service, *.cron, *.consumer
- 슬라이스 내부 정돈: <역할 폴더 | 플랫+파일명>

## 에러 모델
- 분류: <예: 400/401/403/404/409>
- 코드 명명: <규칙>
- 응답 포맷: <예: { code, message }>
- 다국어: <정책>
- 베이스 클래스 위치: <경로>

## 영속성
- soft-delete: <채택 여부 / find 기본 동작>
- batch 메서드 명명: <규칙>
- audit 컬럼: <이름들>
- 도메인/ORM 엔티티: <분리 | 단일>
- 추상 식별자: <abstract class | interface>
- 검증 라이브러리: <이름>

## 테스트
- 파일 배치: <co-located | __tests__>
- 더블 선택 기준: <Stub vs Mock>
- 빌드 격리: <기법>
```
