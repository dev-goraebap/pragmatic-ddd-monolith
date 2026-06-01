---
name: backend-pragmatism-design
description: "Use when the user designs, refactors, reviews, audits, or architects backend code in OOP+DI frameworks (NestJS, Spring Boot, ASP.NET Core, Micronaut, Quarkus). Covers layered architecture with pragmatic DDD borrowing: bounded contexts (BC), aggregate roots, entities, value objects, policies, specifications, domain services, repositories, application services, shallow CQRS (separate query services), domain errors, unit of work, transaction boundaries, request-scoped context, dependency direction enforcement (app → module → shared), and test strategy (Solitary unit / Sociable use-case / Integration). Make sure to use this skill whenever the user asks 'where should this code go', wants to split or merge bounded contexts, extract a policy or specification from messy service code, harden a use case for production (transactions, idempotency, error mapping), remove over-abstraction (dead interfaces, useless ViewModel mappers, single-implementation ports), design a new repository, set up unit of work, plan a multi-BC orchestration, or migrate an existing service toward this style. Triggers on phrases like 'design this feature', 'refactor this service', 'is this layer correct', 'split this BC', 'review this PR', 'add transaction handling', 'where does query service go', '이 코드 어디 둬야 해', 'BC 나눠줘', 'repository 설계', 'application service에 뭐 둬야 해', 'use case test 작성', 'aggregate', 'policy 추출', 'specification', 'unit of work', '의존성 방향'. Also triggers on whether an interface is justified, whether DI inversion is needed, whether shallow CQRS applies. Applies to single-database monoliths or comparably cohesive units. Not for frontend, DI-less raw Express/Fastify, or Go-style explicit-wiring backends."
license: Apache-2.0
metadata:
  version: "0.1.0"
  author: dev-goraebap
---

# Backend Pragmatism Design

OOP와 DI를 제공하는 백엔드 프레임워크 환경에서, **실용적인 레이어드 아키텍처 + 일부 DDD 차용**을 적용하기 위한 설계 스킬입니다. 헥사고날/클린/완전한 CQRS의 도그마를 따르지 않습니다.

설계 사상과 배경 narrative는 [README.md](./README.md)에 있고, 본 SKILL.md는 **라우터와 공통 원칙만** 담습니다. 작업 종류가 정해지면 해당 `reference/*.md`만 펼쳐서 따르면 됩니다. impeccable 스킬의 라우터 패턴을 차용했습니다.

## 전제

- **OOP**: 클래스·상속·다형성을 1급 기법으로 사용한다.
- **DI 컨테이너**: 생성자 주입을 지원한다. (NestJS, Spring, ASP.NET Core, Micronaut, Quarkus, Angular(server) 등)
- **단일 DB 모놀리식 또는 그에 준하는 응집 단위**. 마이크로서비스 분할은 적용 범위 밖.
- **런타임 식별자로 추상 타입 사용 가능**. (TypeScript는 `abstract class`로, JVM/C#은 `interface`로)

전제가 깨지는 환경(순수 함수형, DI 없는 Express/Fastify, Go 등)은 적용 범위 밖입니다.

## 자기 규율: 원리 vs 실행 vs 컨벤션

본 스킬이 다루는 모든 진술은 셋 중 하나로 분류한다.

- **A. 설계 원리**: 프레임워크/언어와 무관하게 "왜"가 성립. SKILL.md 본문과 각 reference 본문에 둔다.
- **B. 프레임워크 실행 방식**: 같은 원리를 NestJS/Spring/.NET이 다르게 구현. 각 reference 하단의 **프레임워크 노트**에 둔다.
- **C. 팀 컨벤션**: 같은 프레임워크 안에서도 팀이 다르게 선택 가능 (디렉터리 명명, 파일 suffix, 에러 코드 명명, soft-delete 채택 여부, audit 컬럼 이름, 검증 라이브러리 등). **PROJECT.md**로 외부화한다.

본문에 실행 방식(B)이나 컨벤션(C)이 섞이면 본 스킬은 특정 스택에서만 작동하는 것이 된다. 본문을 작성·수정할 때 이 분류를 점검 기준으로 삼는다.

## 설정

작업 시작 전에 프로젝트 컨텍스트를 한 번 확인합니다.

1. **PROJECT.md** (있으면 로드, 없으면 생략 가능): BC 목록, 도메인 용어집, 그리고 위 자기 규율의 **C 항목**들 (디렉터리 명명 규약, 파일 suffix, 에러 코드 명명 규칙, soft-delete 채택 여부, audit 컬럼 이름, 검증 라이브러리 선택 등).
2. **ARCHITECTURE.md** (선택): 현 코드베이스의 레이어 구조 스냅샷.

두 파일이 없으면 `teach`로 PROJECT.md를 먼저 채우고, 기존 코드가 충분히 쌓여 있으면 `document`로 ARCHITECTURE.md를 역생성합니다. 같은 세션 내 재로딩은 하지 않습니다 (해당 파일이 갱신된 직후만 예외).

## 공통 설계 법칙

모든 명령에 공통으로 적용됩니다. 개별 reference 문서는 이 법칙 위에서 작동합니다.

### 1. 레이어 3분할과 단방향 의존

```
app/<resource>/    Presentation + Application + Infrastructure(조회)
module/<bc>/       Domain + Infrastructure(영속성/외부 어댑터)
shared/            비즈니스 무관 기술 인프라
```

참조 방향:

- `app → module` 허용 (BC 조립)
- `app, module → shared` 허용
- `module → module` 금지 (BC 간 직접 결합은 항상 `app`이 중재)
- `module → app` 금지
- `shared → app, module` 금지

순환 참조는 구조적으로 차단합니다.

### 2. 변동성 차이가 큰 경계에만 DIP

- **Repository**: 추상 식별자로 노출. 인메모리 테스트 더블 교체가 핵심 가치.
- **외부 서드파티 어댑터**: 기본은 구체 클래스. 행위 검증으로 충분하지 않을 때만 추상화.
- 그 외 클래스에 무차별 인터페이스를 부여하지 않는다.

> 인터페이스의 존재 이유는 헥사고날 형식 충족이 아니라 **테스트 더블 슬롯 제공**이다.

### 3. 얕은 CQRS

- **Write(CUD)**: 도메인 모델을 통한다. Repository로 로드, 도메인 메서드로 전이, Repository로 저장.
- **Read(화면 조회)**: `app/<resource>/` 안의 Query Service에서 DB를 직접 쿼리. 도메인 모델을 거치지 않는다. 크로스 BC JOIN 허용.
- **CUD에서 Query Service 호출 금지**. 화면 스펙 변경이 비즈니스를 깨뜨린다.

### 4. 도메인 보존

- 도메인은 외부 화면 명세나 인프라 결정과 무관하게 독립 설계 가능해야 한다.
- 도메인 모델은 의존성 0의 Solitary Unit Test로 검증된다.
- 영속 스키마의 부수 정보(생성/수정 시각, 작성자, soft-delete 플래그 등)가 도메인 의도를 흐리지 않도록 한다. 구체적 기법(별도 클래스로 분리할지, ORM 어노테이션으로 격리할지, JPA 엔티티에 그대로 두고 도메인 메서드 노출만 통제할지)은 프레임워크/ORM에 따라 갈린다. [원리는 A, 기법은 B]

### 5. 시간과 ID 추상화를 만들지 않는다

- Clock / IdGenerator 같은 인터페이스를 만들지 않는다. 언어/런타임 표준 API를 도메인에서 직접 호출한다.
- 테스트에서 시간 고정이 필요하면 테스트 도구의 시계 모킹 기능을 사용한다.
- 언어별 표준 API 호출 예시는 [reference/model.md](./reference/model.md)의 프레임워크 노트 참조. [B]

### 6. 도메인 에러 분류 체계

- 도메인 에러는 의미별 5종(400 BadRequest / 401 Unauthorized / 403 Forbidden / 404 NotFound / 409 Conflict)으로 분류한다.
- 도메인 에러 클래스가 HTTP 상태로 자동 매핑되도록 프레임워크의 전역 예외 처리 지점에서 연결한다. 매핑 기법(필터 / advice / middleware)은 프레임워크마다 다르다. [B]
- 에러 코드 명명 규칙(prefix, case, 길이)과 메시지 다국어 정책은 팀 컨벤션이다. [C → PROJECT.md]

### 7. 테스트 3-tier

| 위치 | 종류 | 필수 여부 |
| --- | --- | --- |
| `module/<bc>/domain/` | Solitary Unit Test | 필수 |
| `app/<resource>/` | Sociable Use Case Test | 필수 |
| Repository 구현체, Query Service | Integration Test | 선택 (생략 권장) |

## 명령어

| 명령 | 카테고리 | 한 줄 설명 | 본문 |
| --- | --- | --- | --- |
| `teach` | Setup | PROJECT.md(BC 목록·컨벤션·용어집) 세팅 인터뷰 | [reference/teach.md](./reference/teach.md) |
| `document` | Setup | 기존 코드베이스에서 ARCHITECTURE.md 역생성 | [reference/document.md](./reference/document.md) |
| `shape [feature]` | Design | 코드 작성 전 유스케이스 설계 (touch surface, 트랜잭션 경계) | [reference/shape.md](./reference/shape.md) |
| `craft [feature]` | Design | shape 거쳐 end-to-end 구현 | [reference/craft.md](./reference/craft.md) |
| `model [bc]` | Design | 한 BC의 도메인 모델(Entity/VO/Policy/Spec/Error) 설계 | [reference/model.md](./reference/model.md) |
| `boundaries` | Design | BC 분할·병합·이름 정합 검토 | [reference/boundaries.md](./reference/boundaries.md) |
| `critique [target]` | Evaluate | 책임·의존성·복잡도 종합 설계 리뷰 | [reference/critique.md](./reference/critique.md) |
| `audit [target]` | Evaluate | 단방향 의존성 위반·레이어 누수 기계 체크 | [reference/audit.md](./reference/audit.md) |
| `refactor [target]` | Refine | 설계 어긋난 기존 코드 교정 | [reference/refactor.md](./reference/refactor.md) |
| `distill [target]` | Refine | 과잉 추상화·중간 매퍼·죽은 인터페이스 제거 | [reference/distill.md](./reference/distill.md) |
| `harden [target]` | Refine | 트랜잭션·동시성·에러·멱등성 견고화 | [reference/harden.md](./reference/harden.md) |
| `extract [target]` | Refine | 반복 로직을 `_shared/` 또는 Domain Service로 승격 | [reference/extract.md](./reference/extract.md) |
| `transaction [target]` | Topic | UoW, 트랜잭션 경계, 전파 | [reference/transaction.md](./reference/transaction.md) |
| `query [target]` | Topic | 얕은 CQRS, Query Service, N+1 방지 | [reference/query.md](./reference/query.md) |
| `repository [target]` | Topic | Repository 설계 (추상 식별자, soft-delete, batch) | [reference/repository.md](./reference/repository.md) |
| `error-model [target]` | Topic | 도메인 에러 분류·HTTP 매핑 | [reference/error-model.md](./reference/error-model.md) |
| `test-strategy [target]` | Topic | Solitary/Sociable/Integration 작성 기준 | [reference/test-strategy.md](./reference/test-strategy.md) |
| `policy [target]` | Topic | Policy 추출 시점·Entity와의 관계 | [reference/policy.md](./reference/policy.md) |
| `specification [target]` | Topic | Specification 추출 시점·Policy와의 차이 | [reference/specification.md](./reference/specification.md) |

## 라우팅 규칙

1. **인자 없음**: 위 명령어 표를 사용자에게 그대로 보여주고 무엇을 할지 묻는다.
2. **첫 단어가 명령과 일치**: 해당 reference 문서를 로드하고 그 절차를 따른다. 첫 단어 뒤는 모두 대상(target).
3. **명령과 일치하지 않음**: 일반 백엔드 설계 질문으로 간주한다. 위 공통 설계 법칙과 [README.md](./README.md)의 철학을 적용해 답한다.

setup 단계(PROJECT.md / ARCHITECTURE.md 로드)는 첫 명령 시점에 한 번만 실행하고 세션 내 재실행하지 않는다. 예외: `teach`나 `document`가 방금 파일을 갱신한 경우.

## 프레임워크 노트

본문은 모두 프레임워크 중립으로 서술합니다. 각 reference 문서 하단에 `프레임워크 노트` 섹션을 두어 NestJS·Spring·ASP.NET Core에서의 매핑을 한두 줄로 정리합니다.

대표 매핑:

| 추상 개념 | NestJS | Spring Boot | ASP.NET Core |
| --- | --- | --- | --- |
| 런타임 식별자 (DI 토큰) | `abstract class` | `interface` | `interface` |
| 요청 스코프 컨텍스트 | `AsyncLocalStorage` | `RequestContextHolder` / `ThreadLocal` | `AsyncLocal<T>` / `IHttpContextAccessor` |
| 컨테이너 등록 | `@Module` providers | `@Component` / `@Bean` | `services.AddScoped` |
| DTO 검증 | Zod / class-validator | Bean Validation | FluentValidation / DataAnnotations |
| 테스트 더블 | Jest mocks | Mockito | Moq / NSubstitute |
| 트랜잭션 | TypeORM/Prisma 트랜잭션 + ALS | `@Transactional` | `TransactionScope` / EF Core |

상세는 각 reference의 프레임워크 노트 참조.

## 참고

- [README.md](./README.md) - 왜 이렇게 설계했는가 (배경, 트레이드오프, 철학)
- [reference/](./reference/) - 명령별 본문
