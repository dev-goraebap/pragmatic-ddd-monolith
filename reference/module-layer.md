# module-layer

> `module/<bc>` 슬라이스 설계. 애그리게이트 단위 구성, 도메인 요소(Entity/VO/Policy/Spec/Domain Service/Domain Error), Repository=DIP 슬롯, module/shared 커널, Solitary 테스트. SKILL.md "module/<bc>" 섹션의 심화.

## 언제 펼치는가

- 신규 BC를 시작하거나, anemic하거나 분산된 도메인 모델을 다시 잡을 때.
- Aggregate Root에 대응하는 Repository를 신설·확장할 때.
- Policy나 Specification을 추출할 때.

---

## 1. 애그리게이트 단위 구성 (A-style, 권장 기본값)

도메인 엔티티와 그 Repository 구현은 1:1로 강하게 묶인다. 그래서 도메인과 그 영속성을 같은 애그리게이트 폴더 안에 함께 둔다.

- 애그리게이트가 1개면 플랫으로 둔다.
- 여러 개면 애그리게이트별 폴더로 나누고, 각 폴더 안에 `domain/`과 `infrastructure/`를 함께 둔다.

```
module/billing/
├── invoice/
│   ├── domain/          # Invoice(Aggregate Root), VO, Policy, Spec, InvoiceRepository(추상)
│   └── infrastructure/  # InvoiceRepository 구현, ORM 엔티티
└── payment/
    ├── domain/
    └── infrastructure/
```

## 2. 도메인 요소

| 요소 | 정체 | 위치 |
| --- | --- | --- |
| Entity / Aggregate Root | 식별자를 갖고 상태가 전이됨. 불변식 경계 단위. | `domain/` |
| Value Object | 값 자체로 동등성 결정, 불변. | `domain/` |
| Policy | 상태 없는 규칙. 입력(Entity/VO) → 값/결정 산출(how/what value). | `domain/` |
| Specification | 상태 없는 술어. 입력 → boolean(whether). and/or/not 조합 가능. | `domain/` |
| Domain Service | 다중 애그리게이트가 협력해야 하는 도메인 로직. | `domain/` |
| Domain Error | 도메인 규칙 위반 표현. HTTP를 모른다. 분류·매핑은 [project-convention.md](./project-convention.md). | `domain/` |

### Policy vs Specification
- Policy는 "얼마/어떻게"를 계산한다 (할인액, 등급). Application Service가 호출해 결과를 Entity 메서드 인자로 넘긴다(단방향).
- Specification은 "~인가?"를 판정한다 (취소 가능한가, 적격인가). boolean을 반환한다.

### 생성 경로 단일화
정적 팩토리로 생성 경로를 하나로 모으고 거기서 불변식을 검증한다. 구체 idiom은 프레임워크 노트 참조.

## 3. Repository = DIP의 핵심 슬롯

1. Aggregate Root 1개당 Repository 1개다. 추상 식별자는 `domain/`, 구체 구현은 `infrastructure/`에 둔다. 추상의 존재 이유는 인메모리 테스트 더블 슬롯이다.
2. 시그니처는 도메인 타입을 입출력한다. ORM 타입이 추상에 새지 않게 한다.
3. Batch API를 제공해 단건 루프로 N+1을 유발하지 않는다 (메서드 이름은 팀 컨벤션).
4. 트랜잭션 컨텍스트를 시그니처에 노출하지 않는다. 프레임워크 자동 전파에 위임한다.
5. 영속 부수 정보(audit, version)는 Repository 경계에서 처리한다. 처리 위치가 원리이고, 구체 기법(요청 컨텍스트 주입 / ORM 어노테이션 / DB default)은 프레임워크·ORM에 따라 다르다.

### 도메인 vs ORM 엔티티 분리 여부
원리("도메인 의도가 영속 부수 정보에 끌려가지 않음")는 두 경로 모두와 호환된다. 어느 쪽을 택할지는 팀 결정(PROJECT.md)이다.

- 분리 경로: 도메인 클래스와 ORM 표현을 별도로 두고 Repository가 매핑한다.
- 단일 경로: ORM 엔티티가 곧 도메인 엔티티다. audit·기술 필드는 어노테이션으로 격리하고 도메인 메서드 노출만 통제한다.

## 4. module/shared

크로스-BC 공유 도메인 커널이다. 특정 BC에 속하지 않지만 비즈니스성이 있는 것 — 공통 VO(`Money`, `DateRange`), 도메인 베이스 클래스, 공용 도메인 에러 기반 타입 등. 단방향만 허용한다: `module/<bc> → module/shared` ✅, 역방향 ❌. 비즈니스 무관 기술은 여기가 아니라 top-level `shared/`로 간다.

## 5. 테스트 — Solitary Unit (필수)

`module/<bc>/domain/`은 의존성 0의 Solitary Unit Test로 검증한다. 도메인 invariant·Policy·Spec을 외부 없이 검증할 수 있어야 한다. 작성 택틱과 도구는 프로젝트 컨벤션이다.

## 프레임워크 노트

### 생성 경로 단일화
| 프레임워크 | 표현 |
| --- | --- |
| TypeScript (NestJS) | `private constructor` + `static create(props)` + public 필드 |
| Kotlin (Spring) | data class + companion object 팩토리, `init`의 require로 invariant 검증 |
| Java (Spring) | private constructor + static factory. JPA 요구 시 `protected` 무인자 constructor 추가 |
| C# (ASP.NET) | private constructor + static factory. class 또는 record |

### 추상 식별자(Repository)
- NestJS / TS: `abstract class`를 domain에서 export하고 infrastructure에서 상속한다. (`interface`는 런타임에 사라져 DI 토큰으로 부적합)
- Spring: domain 패키지의 `interface`, 구현체는 infrastructure에 `@Repository`.
- ASP.NET Core: domain의 `interface IFooRepository`, `services.AddScoped<IFooRepository, FooRepository>`.

### audit 처리
- NestJS + Drizzle/Prisma (분리 경로): Repository 구현이 요청 컨텍스트(ALS)에서 actorId를 읽어 INSERT/UPDATE 시 주입. `createdAt`/`updatedAt`은 DB default 또는 ORM 훅.
- Spring + JPA: `@CreatedDate`/`@LastModifiedDate`/`@CreatedBy` + `@EnableJpaAuditing`/`AuditorAware`.
- ASP.NET Core + EF Core: `SaveChangesInterceptor` 또는 `SaveChangesAsync` 오버라이드에서 주입.

### batch / N+1
- NestJS / TS: Drizzle/Kysely `inArray`, Prisma `findMany({ where: { id: { in } } })`.
- Spring Data JPA: `findAllById`, `@Query`의 `IN (:ids)`, fetch join.
- EF Core: `Where(x => ids.Contains(x.Id))`, `Include` + `AsSplitQuery`.

### 시간/ID 표준 API (추상화 금지)
- TS: `new Date()`, `crypto.randomUUID()` · Java: `Instant.now()`, `UUID.randomUUID()` · Kotlin: kotlinx-datetime, `UUID.randomUUID()` · C#: `DateTime.UtcNow`, `Guid.NewGuid()`
- 테스트 시계 모킹: Jest `useFakeTimers`, JUnit `MutableClock`/`@MockBean Clock`, xUnit 테스트 더블 Clock.
