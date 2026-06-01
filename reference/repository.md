# repository

> Repository 설계. 추상 식별자, batch 메서드, 트랜잭션·audit 캡슐화. 카테고리: Topic.

## 사용 시점

- 신규 Aggregate Root에 대응하는 Repository 신설.
- 기존 Repository에 메서드 추가.
- N+1 발생 위치 발견 시.

## 핵심 원칙

1. **추상 식별자는 도메인에, 구체 구현은 인프라스트럭처에 둔다.** Aggregate Root 1개당 Repository 1개. 추상의 존재 이유는 테스트 더블 슬롯 제공이다. [A]
2. **메서드 시그니처는 도메인 타입 입출력**으로 유지한다. ORM 타입이 추상 인터페이스에 새지 않게 한다. [A]
3. **Batch API를 제공한다.** 단건 호출 루프로 N+1을 유발하지 않도록 한 번에 여러 건을 처리하는 메서드를 갖춘다. 메서드 이름은 팀 컨벤션. [원리는 A, 이름은 C]
4. **트랜잭션 컨텍스트는 시그니처에 노출하지 않는다.** 호출자가 `tx`/`DbContext`/`EntityManager`를 들고 다니지 않도록 프레임워크의 자동 전파에 위임한다. 자세한 메커니즘은 [[transaction]]. [원리는 A, 메커니즘은 B]
5. **영속 부수 정보(audit, version 등)는 Repository 경계에서 처리한다.** 처리 위치 자체가 원리이며, 구체 기법(요청 컨텍스트 자동 주입 / ORM 어노테이션 / DB default)은 프레임워크와 ORM에 따라 달라진다. [원리는 A, 기법은 B]

## 팀이 별도로 결정할 항목 (PROJECT.md)

- **soft-delete 채택 여부**. 채택한다면 `find*`의 기본 동작(활성 레코드만 반환 vs 전체), 아카이브 포함 조회 메서드의 이름 규칙.
- **batch 메서드 이름** (`saveAll` vs `saveBatch` vs `bulkInsert` 등).
- **audit 컬럼 이름** (`createdAt`/`updatedAt`/`createdBy`/`updatedBy`/`deletedAt`의 영문 표기).
- **추상 식별자의 표현** (TS의 `abstract class` vs JVM/C#의 `interface`; 이건 사실상 B에 가깝지만 팀 코딩 규약과 묶이는 일이 잦음).

## 절차

1. (TBD) Aggregate Root 1개당 Repository 1개로 정렬.
2. (TBD) 메서드 시그니처를 도메인 타입으로 통일.
3. (TBD) batch API 제공 여부 점검.
4. (TBD) audit/version 처리 위치를 Repository 경계로 일관화 (구체 기법은 프레임워크 노트 참조).
5. (TBD) 인메모리 테스트 더블을 동시에 작성.

## 관련

- [[transaction]] - tx 캡슐화 메커니즘.
- [[query]] - 읽기 측은 Repository를 거치지 않는다.
- [[test-strategy]] - Stub 위치.

## 프레임워크 노트

### 추상 식별자

- **NestJS / TS**: `abstract class`를 도메인에서 export하고 infrastructure에서 상속. TS `interface`는 런타임에 사라져 DI 토큰으로 부적합.
- **Spring**: 도메인 패키지의 `interface`. 구현체는 infrastructure 패키지에 `@Repository`로.
- **ASP.NET Core**: 도메인 프로젝트의 `interface IFooRepository`. 구현체는 infrastructure 프로젝트, `services.AddScoped<IFooRepository, FooRepository>`.

### audit/감사 정보 처리

- **NestJS + Drizzle/Prisma/TypeORM (도메인 분리 경로)**: 도메인 엔티티에 audit 필드를 두지 않고, Repository 구현체가 요청 스코프 컨텍스트(`AsyncLocalStorage`)에서 actorId를 읽어 INSERT/UPDATE 시 주입. `createdAt`/`updatedAt`은 DB default 또는 ORM 훅.
- **Spring + JPA**: `@CreatedDate`/`@LastModifiedDate`/`@CreatedBy`/`@LastModifiedBy`를 JPA 엔티티에 직접 부여하고 `@EnableJpaAuditing` + `AuditorAware`로 자동 채움. 도메인 엔티티와 JPA 엔티티를 분리한다면 NestJS 경로에 가깝게.
- **ASP.NET Core + EF Core**: `SaveChangesInterceptor` 또는 `DbContext.SaveChangesAsync` 오버라이드에서 `IHttpContextAccessor` 통해 actorId 주입. 또는 별도 `IAuditable` 인터페이스를 엔티티에 부여.

### batch / N+1

- **NestJS / TS**: Drizzle/Kysely의 `inArray`, Prisma의 `findMany({ where: { id: { in } } })`. 단건 `findById` 루프 금지.
- **Spring Data JPA**: `findAllById(Iterable<ID>)` 또는 `@Query`로 `IN (:ids)`. fetch join으로 연관 로딩.
- **EF Core**: `Where(x => ids.Contains(x.Id))`. `Include` + `AsSplitQuery`로 조정.
