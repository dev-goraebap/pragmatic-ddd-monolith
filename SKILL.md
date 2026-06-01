---
name: backend-pragmatism-design
description: "Use when designing, reviewing, or migrating backend code in OOP+DI frameworks (NestJS, Spring Boot, ASP.NET Core, Micronaut, Quarkus) and the question is WHERE code should physically live. Structure-first: a pragmatic layered architecture on three layers with a strict one-directional reference graph — app/<feature> (presentation + orchestration + read query), module/<bc> (domain + persistence, by aggregate), shared/ (business-independent tech). Answers where code goes, whether a layer/reference direction is correct, splitting or merging bounded contexts, where query service / repository / orchestration service belong, and avoiding app→app or module→module coupling. Borrows DDD; applies shallow CQRS and variance-based DIP. Triggers on '이 코드 어디 둬야 해', 'BC 나눠줘', '레이어 맞아?', 'repository 어디 둬', 'query service 위치', '의존성 방향', 'where should this go'. For single-DB monoliths; error taxonomy and test tactics are delegated to per-project convention. Not for frontend, DI-less Express/Fastify, or Go-style backends."
license: Apache-2.0
metadata:
  version: "0.2.0"
  author: dev-goraebap
---

# Backend Pragmatism Design

OOP와 DI를 제공하는 백엔드 프레임워크에서 "이 코드가 물리적으로 어디에 살아야 하는가"를 결정하는 **구조-first** 설계 스킬이다. 헥사고날·클린·완전한 CQRS의 도그마를 따르지 않는다.

핵심은 명령어 모음이 아니라 세 물리 레이어와 그 사이의 단방향 참조 그래프다. 대부분의 판단은 이 문서 하나로 끝나며, 깊은 설명이 필요할 때만 해당 `reference/*.md`를 펼친다.

왜(Why) 이렇게 설계했는지는 [README.md](./README.md)에 있다. 이 문서는 무엇(What)을 어디에 두는지를 다룬다.

## 전제

- OOP: 클래스·상속·다형성을 1급 기법으로 쓴다.
- DI 컨테이너: 생성자 주입을 지원한다 (NestJS, Spring, ASP.NET Core, Micronaut, Quarkus 등).
- 단일 DB 모놀리식 또는 그에 준하는 응집 단위. 마이크로서비스 분할은 적용 범위 밖이다.
- 런타임 식별자로 추상 타입을 쓸 수 있다 (TypeScript `abstract class`, JVM/C# `interface`).

전제가 깨지는 환경(순수 함수형, DI 없는 Express/Fastify, Go 등)은 적용 범위 밖이다.

## 자기 규율: 원리 vs 실행 vs 컨벤션

이 스킬의 모든 진술은 셋 중 하나다. 작성·수정·리뷰의 점검 기준으로 삼는다.

- **A. 설계 원리** — 프레임워크/언어와 무관하게 "왜"가 성립한다. 본문에 둔다.
- **B. 프레임워크 실행 방식** — 같은 원리를 NestJS/Spring/.NET이 다르게 구현한다. 각 reference 하단 프레임워크 노트에 둔다.
- **C. 팀 컨벤션** — 같은 프레임워크 안에서도 팀이 다르게 고른다 (폴더명, 파일 suffix, 에러 코드 명명, soft-delete, audit 컬럼명, 검증 라이브러리 등). PROJECT.md로 외부화한다. → [reference/project-convention.md](./reference/project-convention.md)

본문에 B나 C가 섞이면 이 스킬은 특정 스택에서만 작동하게 된다.

---

## 핵심: 세 레이어와 단방향 참조 그래프

```
app/<feature>/    Presentation + Application(오케스트레이션) + Infrastructure(조회)
       │
       ▼
module/<bc>/       Domain + Infrastructure(영속성/외부 어댑터)
       │
       ▼
shared/            비즈니스 무관 기술 인프라  (아키텍처 최하위층)
```

참조는 항상 위에서 아래로만 흐른다.

- `app → module` ✅ (비즈니스 조립)
- `app → shared`, `module → shared` ✅
- `module → app` ❌ — 도메인은 화면 계약을 몰라야 한다.
- `module → module` ❌ — BC 간 조립은 항상 app이 중재한다.
- `shared → app`, `shared → module` ❌ — shared는 비즈니스 독립이어야 한다.

레이어 안의 슬라이스(layer 내 한 단위)끼리도 격리한다.

- `app/<a>` ↔ `app/<b>` ❌ (기능 슬라이스끼리 직접 참조 금지)
- `module/<x>` ↔ `module/<y>` ❌ (BC끼리 직접 참조 금지)
- `shared/<f1>` → `shared/<f2>` 는 허용하되 단방향만 (순환 금지)

이 단방향성이 모놀리식의 가장 큰 늪인 **순환 참조를 구조적으로 차단**한다.

## 배치 한 줄 휴리스틱

> 불변 비즈니스 규칙인가? → `module`
> 화면·외부·액터 요구에 결합되는가? → `app`
> 비즈니스와 무관한 기술인가? → `shared`

신규 팀원에게 줄 단 하나의 판단 기준이다.

---

## app/&lt;feature&gt; — 기능(feature) 슬라이스

app은 여러 module을 자유롭게 주입받아 유스케이스를 조립하는 **단일 결합 지점**이다. 그래서 app끼리는 격리하고, 조립 책임만 여기 모은다.

### 기능-first, 플랫, 격리

- 슬라이스는 기능(사용자/액터 관점의 유스케이스 묶음) 단위다. URL 계층을 그대로 따라간 중첩 폴더(`app/orgs/departments/`)는 만들지 않는다. module처럼 평면 슬라이스로 둔다.
- `app/<feature>`는 형제 슬라이스를 참조하지 않는다. 공유가 필요하면, 도메인이면 module로 내리고 횡단 글루면 `app/shared`로 보낸다.
- app 슬라이스는 module과 모양만 닮았을 뿐 1:1 미러가 아니다. 한 기능이 여러 BC를 조립하고(1:N), 한 BC가 여러 기능에 쓰인다(N:1). **N:M이 정상**이다. 1:1 미러가 되면 BC-first로 회귀한 것이다.

### 채널은 폴더 축이 아니라 슬라이스 내부 프래그먼트(slice 안의 역할 단위)다

진입점(driving/inbound adapter)은 REST 컨트롤러만이 아니다. CLI, 스케줄/크론, 메시지·이벤트 컨슈머, 웹훅 등이 있다. 이들을 채널별 최상위 폴더(`app/http/`, `app/jobs/`)로 가르면 같은 유스케이스가 채널마다 복제된다.

그래서 채널은 슬라이스 안의 프래그먼트로 둔다. 채널 핸들러는 얇게(입력 파싱 → 호출 → 출력 포맷) 두고, **오케스트레이션은 채널-중립 `service` 한 곳에만** 둔다. URL 중첩은 컨트롤러의 라우트 경로(`@Controller('orgs/:id/departments')`)로만 표현한다.

```
app/checkout/
├── checkout.controller.ts       # 채널 프래그먼트 (REST)
├── checkout.cron.ts             # 채널 프래그먼트 (스케줄)  ← 폴더 아님
├── checkout.service.ts          # 오케스트레이션 (여러 CUD; module 조립)
├── checkout-query.service.ts    # 얕은 CQRS 읽기 (DB 직접 JOIN → Response DTO)
└── dto/
```

- 오케스트레이션은 `service` 하나에 여러 CUD 기능을 담는다. 액터가 늘면 `admin-checkout.service.ts` 식으로 분리할 수 있다.
- 슬라이스 내부 정돈 방식(역할 폴더 vs 파일명 규약)은 프로젝트 컨벤션에 위임한다. 권장 그림은 "기능 폴더가 바깥, 역할 프래그먼트가 안" — 자세히는 [reference/app-layer.md](./reference/app-layer.md).

## module/&lt;bc&gt; — 도메인 + 영속성 동거

도메인 엔티티와 그 Repository 구현은 1:1로 강하게 묶인다. 그래서 도메인과 그 영속성은 같은 BC 폴더 안에 동거시킨다.

- 애그리게이트 단위로 묶는다 (권장 기본값, SHOULD). 애그리게이트가 1개면 플랫, 여러 개면 애그리게이트별 폴더로 나누고 각 폴더 안에 `domain/`과 `infrastructure/`를 함께 둔다.
- 도메인 요소: Entity / Value Object / Policy / Specification / Domain Service / Domain Error.
- Repository는 애그리게이트 루트당 하나다. DIP의 핵심 슬롯이다(아래).

```
module/billing/
├── invoice/
│   ├── domain/          # Invoice 애그리게이트, VO, Policy, Spec, InvoiceRepository(추상)
│   └── infrastructure/  # InvoiceRepository 구현, ORM 엔티티
└── payment/
    ├── domain/
    └── infrastructure/
```

자세히는 [reference/module-layer.md](./reference/module-layer.md).

## shared 3종

각 레이어는 자기 공용물을 담는 `shared` 하위를 가질 수 있다. 기본 명명은 `_shared`가 아니라 `shared`이며, 모든 폴더명은 PROJECT.md로 덮어쓸 수 있다.

| 위치 | 무엇 | 규칙 |
| --- | --- | --- |
| `shared/` (top 레이어) | 비즈니스 무관 기술 인프라 (DB 커넥션, 로깅, 설정, 유틸) | 기술 noun으로 fragment, fragment 간 단방향 |
| `app/shared` | 횡단 글루 (Guard, Interceptor, Decorator/어노테이션, 공통 DTO) | 서비스(오케스트레이션) 이주 금지 |
| `module/shared` | 크로스-BC 공유 도메인 커널 (공통 VO·도메인 베이스 클래스·공용 도메인 에러 기반 타입) | `module/<bc> → module/shared` ✅, 역방향 ❌ |

---

## DIP: 인터페이스는 테스트 더블 슬롯이다

인터페이스의 존재 이유는 헥사고날 형식 충족이 아니라 **테스트에서 더블을 교체 주입할 슬롯**이다. 변동성 차이가 큰 경계에만 제한적으로 적용한다.

- Repository는 추상으로 노출한다. 인메모리 테스트 더블 교체가 핵심 가치다. (필수)
- 외부 서드파티 어댑터(결제·메일·Slack 등)는 기본은 구체 클래스다. 행위 검증(모킹)으로 충분하면 인터페이스를 만들지 않는다. 어댑터 설계 일관성을 강화할 목적일 때만 선택적으로 추상화한다.
- 그 외 클래스에 무차별로 인터페이스를 부여하지 않는다.
- 시간과 ID는 추상화하지 않는다. `Clock`/`IdGenerator` 인터페이스를 만들지 말고, 언어 표준 API를 도메인에서 직접 호출한다. 테스트는 시계 모킹 도구로 고정한다.

## 얕은 CQRS

- Write(CUD)는 도메인 모델을 통한다. Repository로 로드 → 도메인 메서드로 전이 → Repository로 저장.
- Read(화면 조회)는 `app/<feature>` 안의 Query Service가 DB를 직접 JOIN해 Response DTO를 바로 만든다. 도메인 모델을 거치지 않으며, 크로스 BC JOIN을 허용한다.
- **CUD에서 Query Service를 호출하지 않는다.** 화면 스펙 변경이 비즈니스를 깨뜨리지 않게 하기 위함이다.

## 테스트는 폴더가 결정한다 (구조적 흔적)

| 위치 | 테스트 종류 | 필수 여부 |
| --- | --- | --- |
| `module/<bc>/domain/` | Solitary Unit (의존성 0) | 필수 |
| `app/<feature>/*.service` | Sociable Use Case | 필수 |
| Repository 구현체, Query Service | Integration | 선택 (생략 권장) |

폴더 구조만으로 어떤 테스트를 써야 하는지가 정해진다. 테스트 작성 택틱과 도구 선택은 프로젝트 컨벤션이다.

## 에러 모델은 프로젝트 컨벤션이다

도메인 에러 분류, 코드 명명, HTTP 매핑 기법은 이 스킬이 강제하지 않는다. → [reference/project-convention.md](./reference/project-convention.md)

---

## 안티패턴

- app↔app / module↔module 직접 참조 — 격리 위반. 조립은 app, 공유는 module 또는 shared로.
- 채널을 최상위 폴더 축으로 (`app/http/`, `app/consumers/`) — 오케스트레이션 중복을 부른다.
- URL 계층을 그대로 베낀 app 중첩 폴더 — 기능 슬라이스로 평탄화하라.
- Query Service를 `module/<bc>/infrastructure/`에 — 반환 DTO가 app에 있어 역방향 의존이 생긴다. app 슬라이스 안에 둔다.
- CUD 서비스가 Query Service 호출 — 화면 스펙이 비즈니스를 오염시킨다.
- 무차별 인터페이스 / 단일 구현 포트 / 죽은 ViewModel 매퍼 — 테스트 더블 슬롯이 아니면 만들지 않는다.
- `Clock`/`IdGenerator` 추상화 — 표준 API를 직접 호출한다.
- `app/shared`로 오케스트레이션 서비스 이주 — app/shared는 횡단 글루 전용이다.
- 도메인 로직이 app 서비스에 누적 — 중복의 진원지. module 도메인으로 내려라.

## 조건부 references

대부분은 위 본문으로 끝난다. 아래 상황에서만 해당 문서를 펼친다 (전부 미리 로드하지 않는다).

- 참조/의존성 위반을 따지거나 BC 경계를 나눌 때 → [reference/dependency-rules.md](./reference/dependency-rules.md)
- app 슬라이스 설계 / 채널·service·query 배치 / app 중복 처리 → [reference/app-layer.md](./reference/app-layer.md)
- BC 도메인 모델 / 애그리게이트·Repository·Policy·Spec 설계 → [reference/module-layer.md](./reference/module-layer.md)
- 기존 코드베이스를 이 구조로 옮길 때 → [reference/migration.md](./reference/migration.md)
- 위임 항목(폴더명·에러·테스트 택틱 등) 결정 / PROJECT.md 작성 → [reference/project-convention.md](./reference/project-convention.md)

## 프레임워크 노트 (대표 매핑)

| 추상 개념 | NestJS | Spring Boot | ASP.NET Core |
| --- | --- | --- | --- |
| 런타임 식별자 (DI 토큰) | `abstract class` | `interface` | `interface` |
| 요청 스코프 컨텍스트 | `AsyncLocalStorage` | `RequestContextHolder` / `ThreadLocal` | `AsyncLocal<T>` / `IHttpContextAccessor` |
| 컨테이너 등록 | `@Module` providers | `@Component` / `@Bean` | `services.AddScoped` |
| 트랜잭션 경계 | ORM 트랜잭션 + ALS | `@Transactional` | `TransactionScope` / EF Core |

상세는 각 reference의 프레임워크 노트 참조.
