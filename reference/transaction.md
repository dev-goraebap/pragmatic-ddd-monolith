# transaction

> UoW 패턴, 트랜잭션 경계, 전파 정책, 외부 호출 격리. 카테고리: Topic.

## 사용 시점

- 다중 쓰기(여러 Repository 호출)가 한 유스케이스에 있을 때.
- 외부 API 호출과 DB 쓰기가 같은 흐름에 섞일 때.
- 동시성 충돌이 의심될 때.

## 핵심 원칙

1. **트랜잭션 경계는 Application Service**가 설정한다. Controller도 Repository도 아니다. [A]
2. **트랜잭션 컨텍스트를 시그니처에 노출하지 않는다.** 호출자가 `tx` 객체나 `DbContext`를 들고 다니지 않도록, 프레임워크가 제공하는 자동 전파 메커니즘을 사용한다. [원리는 A, 메커니즘은 B]
3. **외부 호출은 트랜잭션 바깥**. Outbox 또는 사후 트리거. [A]
4. **읽기 전용 트랜잭션**은 명시적으로 구분. [A]

## 절차

1. (TBD) UoW 진입점 식별.
2. (TBD) 묶을 쓰기 작업 나열.
3. (TBD) 외부 호출 위치 점검 후 외부로 옮김.
4. (TBD) 동시성 정책 결정(낙관/비관/없음).
5. (TBD) 롤백 시 사이드이펙트(파일, 메모리 캐시) 정리 hook.

## 관련

- [[harden]] - 운영 견고화의 일부.
- [[repository]] - tx 캡슐화는 Repository 구현체에서.

## 프레임워크 노트

- NestJS / TS: TypeORM `DataSource.transaction` 또는 Prisma `$transaction` + `AsyncLocalStorage`로 tx 전파. 라이브러리: `nestjs-cls`, `typeorm-transactional`.
- Spring: `@Transactional`(propagation, readOnly, isolation 명시). 프록시 self-invocation 함정 주의.
- ASP.NET Core: `TransactionScope` 또는 EF Core `DbContext.Database.BeginTransaction`. Scoped DbContext가 기본 단위.
