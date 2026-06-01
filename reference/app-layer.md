# app-layer

> `app/<feature>` 슬라이스 설계. 기능-first 구성, 채널 프래그먼트, 단일 오케스트레이션 service, 얕은 CQRS 읽기, app/shared, 중복 처리 원칙. SKILL.md "app/<feature>" 섹션의 심화.

## 언제 펼치는가

- 새 app 슬라이스를 설계하거나, REST 외 채널(크론·컨슈머·웹훅)을 붙일 때.
- app 서비스 중복을 어떻게 풀지 결정할 때.
- Query Service를 어디에 어떻게 둘지 정할 때.

---

## 1. 기능(feature) 슬라이스

app은 여러 module을 자유롭게 주입받는 단일 결합 지점이다. 그래서 다음 원칙을 따른다.

- 슬라이스는 사용자/액터 관점의 기능 단위다. URL 계층을 그대로 베낀 중첩 폴더(`app/orgs/departments/`)는 만들지 않는다. module처럼 평면 슬라이스로 두고 형제끼리 격리한다.
- module과 1:1 미러가 아니다. 한 기능이 여러 BC를 조립하고(1:N), 한 BC가 여러 기능에 쓰인다(N:1). N:M이 정상이다. 단순 CRUD 기능이 한 module과 겹치는 것은 우연이지 규칙이 아니다.
- 단순 기능은 플랫으로, 연관된 여러 기능이 한 묶음으로 그룹화되면 그룹 폴더가 바깥에 오고 역할 프래그먼트가 안에 온다(권장):

```
app/orders/
├── ordering/                    # 기능 1
│   ├── ordering.controller.ts
│   ├── ordering.service.ts
│   └── dto/
└── fulfillment/                 # 기능 2
    ├── fulfillment.controller.ts
    ├── fulfillment.cron.ts       # 채널은 폴더가 아니라 프래그먼트
    ├── fulfillment.service.ts
    └── dto/
```

내부 정돈 방식(역할 폴더 vs 파일명 규약)은 프로젝트 컨벤션에 위임한다. 위는 권장 그림일 뿐 강제가 아니다.

## 2. 채널은 슬라이스 내부 프래그먼트다

진입점(driving/inbound adapter)은 REST만이 아니다. CLI, 스케줄/크론, 메시지·이벤트 컨슈머, 웹훅, 그리고 WebSocket/GraphQL/gRPC 같은 프로토콜 변형이 있다. 공통점은 "바깥에서 들어와 app의 조립 로직을 깨운다"는 것이다.

이들을 채널별 최상위 폴더(`app/http/`, `app/jobs/`)로 가르면 같은 유스케이스가 채널마다 복제된다. 그래서 얇은 어댑터 / 단일 service 원칙을 따른다.

- 채널 핸들러(controller, cron handler, consumer, webhook)는 입력 파싱 → service 호출 → 출력 포맷만 한다. 오케스트레이션 로직은 두지 않는다.
- 오케스트레이션은 채널-중립 `service` 한 곳에만 둔다. 이로써 채널 분할로 인한 중복이 구조적으로 사라진다.
- URL 중첩은 폴더가 아니라 컨트롤러 라우트 경로(`@Controller('orgs/:id/departments')`)로 표현한다.

## 3. 오케스트레이션 service

- `service` 하나에 여러 CUD 기능을 담는다 (`checkout.service.ts`).
- 트랜잭션 경계는 service가 설정한다. Controller도 Repository도 아니다.
- 트랜잭션 컨텍스트는 시그니처에 노출하지 않는다. 호출자가 `tx`/`DbContext`/`EntityManager`를 들고 다니지 않도록 프레임워크 자동 전파에 위임한다.
- 외부 호출은 트랜잭션 바깥에 둔다(Outbox 또는 사후 트리거). 읽기 전용 흐름은 명시적으로 구분한다.
- 액터가 늘면 그 기점으로 분리할 수 있다 (예: `admin-checkout.service.ts`).

## 4. 얕은 CQRS 읽기 — Query Service

- 위치는 `app/<feature>/` 안이다. `module/`에 두지 않는다. 반환 타입(Response DTO)이 app에 있어 역방향 의존이 생기기 때문이다.
- Domain/Repository를 거치지 않고 DB를 직접 쿼리한다. 크로스 BC JOIN을 허용한다.
- Response DTO를 직접 생성한다. 중간 ViewModel은 두지 않는다.
- CUD service에서 호출하지 않는다. 화면 스펙 변경이 비즈니스를 깨뜨리기 때문이다.
- 읽기 전용 트랜잭션을 명시한다. N+1은 단일 쿼리나 배치 로딩으로 방지한다.

```
app/checkout/
├── checkout.service.ts          # 쓰기: module 조립
└── checkout-query.service.ts    # 읽기: DB 직접 JOIN → Response DTO
```

## 5. app/shared

횡단 글루만 담는다 — Guard, Interceptor, Decorator/어노테이션, 공통 DTO. 오케스트레이션 service를 여기로 옮기지 않는다. app이 도메인-first로 정돈된 이상, 공통 조립 로직은 service가 아니라 도메인(module)으로 가야 한다.

## 6. app 서비스 중복 처리 (의사결정 순서)

중복을 보면 근본 원인부터 분류한다. 첫 질문은 항상 "이거 도메인 아냐?"다.

1. 비즈니스 규칙·불변식인가? → module 도메인으로 내린다 (도메인 메서드 / Policy / Specification / Domain Service). 가장 흔한 진짜 원인이다.
2. 프레젠테이션 글루인가? (공통 DTO/Guard/Mapper) → `app/shared`.
3. 읽기 쿼리·프로젝션인가? → 대개 중복을 허용한다. Query Service를 결합하면 화면 스펙 변경이 번진다. 정말 동일하면 순수 SQL 헬퍼만 추출한다.
4. 우연히 비슷할 뿐이거나 곧 갈라지는가? → 중복을 허용한다. 투기적으로 추상화하지 않는다.

채널-first가 아니라 기능-first + 얇은 어댑터/단일 service를 택했으므로, 채널에서 오던 중복은 이미 구조적으로 해소된 상태다. 여기 남는 중복만 위 순서로 처리한다.

## 프레임워크 노트

### 요청 스코프 컨텍스트 / 트랜잭션 전파
- NestJS / TS: `AsyncLocalStorage`(`nestjs-cls`). 트랜잭션은 TypeORM `DataSource.transaction`/`typeorm-transactional` 또는 Prisma `$transaction`.
- Spring: `@Transactional`(propagation·readOnly·isolation 명시). 프록시 self-invocation 함정 주의. `RequestContextHolder`/`ThreadLocal`.
- ASP.NET Core: `TransactionScope` 또는 EF Core `BeginTransaction`. Scoped `DbContext`가 기본 단위. `IHttpContextAccessor`/`AsyncLocal<T>`.

### Query Service 구현
- NestJS / TS: Drizzle/Kysely 쿼리 빌더 또는 raw SQL. ORM 풀 그래프 로딩 회피.
- Spring: Spring Data Projections, JPQL DTO projection, 또는 jOOQ.
- ASP.NET Core: Dapper 또는 EF Core projection(`Select(x => new ResponseDto { ... })`).

### 채널 프래그먼트
- NestJS: `@Controller` / `@Cron`(`@nestjs/schedule`) / `@EventPattern`·`@MessagePattern`(microservices) / 웹훅은 별도 컨트롤러.
- Spring: `@RestController` / `@Scheduled` / `@KafkaListener`·`@RabbitListener`.
- ASP.NET Core: Controller/Minimal API / `IHostedService`·`BackgroundService` / 메시지 컨슈머.
