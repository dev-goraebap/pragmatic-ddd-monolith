# model

> 한 BC의 도메인 모델 설계. Entity / VO / Policy / Specification / Domain Service / Domain Error의 관계를 잡는다. 카테고리: Design.

## 사용 시점

- 신규 BC 시작.
- 기존 BC의 도메인 모델이 anemic하거나 분산되어 다시 잡고 싶을 때.

## 입력

- BC 이름과 책임 범위.
- 핵심 비즈니스 규칙(불변식, 정책, 판정 조건).

## 절차

1. (TBD) **Aggregate Root 식별**: 불변식 경계 단위.
2. (TBD) **Entity vs VO 구분**: 식별자가 필요한가 vs 값 자체로 동등성이 결정되는가.
3. (TBD) **Policy 추출 자리** 후보: 값/결정 계산.
4. (TBD) **Specification 추출 자리** 후보: 판정(boolean).
5. (TBD) **Domain Service 자리** 후보: 다중 애그리게이트 협력.
6. (TBD) **Domain Error 분류**.
7. (TBD) Params/Props 인터페이스 정의 (정적 팩토리·상태 전이 입력).
8. (TBD) Solitary Unit Test로 검증.

## 출력

- `module/<bc>/domain/` 하위 파일 풀세트.
- 각 도메인 요소의 단위 테스트.

## 관련

- [[policy]] - Policy 추출 상세.
- [[specification]] - Specification 추출 상세.
- [[error-model]] - 에러 분류.
- [[repository]] - 모델에 대응하는 Repository.

## 프레임워크 노트

### 생성 경로 단일화 idiom

| 프레임워크 | 표현 |
| --- | --- |
| TypeScript (NestJS) | `private constructor` + `static create(props)` 정적 팩토리 + public 필드 |
| Kotlin (Spring) | data class + companion object 팩토리. 또는 `init` 블록의 require로 invariant 검증 |
| Java (Spring) | private constructor + static factory method. JPA가 요구하면 `protected` 무인자 constructor 추가 |
| C# (ASP.NET) | private constructor + static factory method. record 또는 class |

### 도메인 vs ORM 엔티티 분리 여부

본 스킬의 원리("도메인 의도가 영속 부수 정보에 끌려가지 않음")는 두 경로 모두와 호환된다.

- **분리 경로**: 도메인 클래스와 ORM 표현을 별도 클래스로 두고 Repository가 매핑. (NestJS+Drizzle/Prisma의 일반적 형태, Spring에서도 가능)
- **단일 경로**: JPA `@Entity` / EF Core entity가 곧 도메인 엔티티. audit 컬럼과 기술 필드는 어노테이션(`@CreatedDate`, `@LastModifiedDate`, `@Version`)으로 격리하고, 도메인 메서드 노출만 통제.

어느 경로를 택할지는 팀 결정(PROJECT.md). 본 스킬은 어느 쪽도 강제하지 않는다.

### 시간/ID 표준 API

- TypeScript: `new Date()`, `crypto.randomUUID()`
- Java: `Instant.now()`, `UUID.randomUUID()`
- Kotlin: `Clock.System.now()` (kotlinx-datetime), `UUID.randomUUID()`
- C#: `DateTime.UtcNow`, `Guid.NewGuid()`

테스트 시계 모킹: Jest `useFakeTimers`, JUnit `MutableClock` (또는 `@MockBean Clock`), xUnit + 테스트 더블 Clock.
