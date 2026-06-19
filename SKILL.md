---
name: pragmatic-ddd-monolith
description: "Use when structuring or refactoring a single-deployment monolith with a single database (Spring Boot / Java) by borrowing DDD pragmatically. Bounded contexts are packages under module/<bc>, each laid out in layers — domain/ (aggregates + repository interfaces), application/{command,query} (CQS), infrastructure/adapter, and contract/ (Open Host Service + Published Language, only when the context is upstream). Cross-context dependencies flow one way (downstream → upstream) and only through the upstream's contract; the downstream picks Conformist (use the contract directly) or ACL (declare its own port + an adapter that translates). Domain stays pure and self-validating but may use JPA annotations directly. Triggers on '이 코드 어디 둬', '컨텍스트 나눠', 'contract 어디 둬', 'OHS/PL', 'ACL', 'command query 분리', '상류 하류', 'where does this go', 'how should bounded contexts talk'. Single deployment + single DB only — NOT modules deployed separately, MSA, or multi-DB."
license: Apache-2.0
metadata:
  version: "0.6.0"
  author: dev-goraebap
---

# Pragmatic DDD Monolith

**단일 배포 모놀리식 + 단일 DB**(Spring Boot / Java)에서 DDD를 실용적으로 빌려 **바운디드 컨텍스트를 패키지로 나누는** 표준 아키텍처입니다. 모듈을 따로 배포하지 않습니다(그건 MSA). 정석 DDD를 교조적으로 따르기보다 **생산성과의 타협**을 우선합니다.

## 전제

- **단일 배포 모놀리식 + 단일 DB.** 모듈 별도 배포·MSA·다중 DB는 적용 범위 밖.
- **Spring Boot / Java** (OOP + DI) 기준. 다른 OOP+DI 스택에도 구조는 응용 가능.

## 두 가지 경계 (섞지 말 것)

| 경계 | 무엇을 나누나 |
| --- | --- |
| **컨텍스트 경계** | 비즈니스 영역 (위험성평가 / 공사 / 결재 …) |
| **레이어 경계** | 한 컨텍스트 *내부*의 기술 책임 (표현 / 응용 / 도메인 / 인프라) |

이 둘을 또렷이 구분하는 게 복잡도 관리의 핵심이다. 컨텍스트는 보통 서브도메인과 1:1로 매핑하되, 너무 크거나 합치는 게 나으면 팀이 조율한다.

## 1. 패키지 구조

```
com.example.app
├── common/      # 도메인 무관 공통 인프라 (config·entity·exception·paging 등)
├── config/      # 애플리케이션 전역 설정
├── util/        # 무상태 순수 유틸리티
└── module/      # ★ 비즈니스 컨텍스트들
    ├── riskassessment/   # 하류(downstream) 예시
    ├── construction/     # 상류(upstream) 예시
    └── ...
```

- `module/<bc>` 하나 = 독립 비즈니스 컨텍스트. 내부는 레이어 구조(아래).
- `common`/`util`은 자유롭게 가져다 쓰되, **거기서 개별 비즈니스 컨텍스트를 역참조하지 않는다.**

## 2. 컨텍스트 내부 레이어

```
module/<bc>/
├── domain/                     # 비즈니스 규칙·불변식. 애그리거트별 패키지
│   └── <agg>/  { Root(애그리거트 루트), 자식 객체, Repository(인터페이스) }
├── application/
│   ├── command/                # 쓰기 유스케이스. dto/ · mapper/ · provider/(ACL 포트)
│   └── query/                  # 조회. <X>ReadQuery(조회 전용 IF) · dto/(View·Projection)
├── infrastructure/adapter/     # Repository 구현, ACL 어댑터, 외부 연동
└── contract/                   # ★ 상류일 때만. OHS(XxxContract) + PL(공개 타입)
```

- **의존은 바깥 → 안(domain).** 도메인은 외부 레이어나 타 컨텍스트 기술 스펙을 직접 참조하지 않는다.
- **도메인 = 영속성 겸용 허용.** 매퍼 분리를 강요하지 않고 **도메인 엔티티에 JPA 애노테이션 직접 사용을 허용**한다.
- 지키려는 3가지: ① **단위 테스트 용이성**(DB/스프링 없이 객체 단위로) ② **의존성 격리** ③ **자가 검증**(생성자·정적 팩토리·비즈니스 메서드에서 불변식 예외).

구조 개요 — 컨텍스트 내부 레이어와 상류(B)/하류(A) 협업(OHS·PL·ACL·Conformist):

![아키텍처 구조 개요](reference/diagram.png)

## 3. CQS — 쓰기와 읽기 분리

`application`을 **command(쓰기)** 와 **query(읽기)** 로 패키지째 가른다.

| 구분 | 위치 | 모델 | DB 접근 |
| --- | --- | --- | --- |
| **Command** | `application/command` | 도메인 애그리거트 | `domain/.../Repository`로 로드·검증·저장 |
| **Query** | `application/query` | 읽기 전용(View·Projection) | `<X>ReadQuery` 등 조회 전용 |

- **유스케이스 처리를 위해 객체를 메모리에 로드**하는 건(예: `findById`) 화면 조회가 아니라 **쓰기 경로** → 도메인 Repository 사용.
- **command 서비스가 query 서비스를 직접 호출하지 않는다.** (비즈니스 ↔ 화면 결합도 ↓)
- **query는 도메인 Repository를 직접 참조하지 않는다.** `ReadQuery`로 조회. **화면 조회는 컨텍스트를 가로질러 DB 직접 JOIN을 허용**(애플리케이션 조인 대신). 단, 조회에서 도메인 판별 로직(예: 수정 가능 여부)이 꼭 필요하면 예외적으로 도메인 엔티티를 조회해 재활용.

## 4. 컨텍스트 간 관계 — 상류(Upstream) / 하류(Downstream)

- **상류**: 기능·데이터를 제공하는 쪽. 계약(인터페이스)의 주도권을 가진다.
- **하류**: 상류에 의존하는 쪽. 상류 스펙을 수용해 비즈니스를 수행한다.

규칙:
1. **의존성은 하류 → 상류 단방향.**
2. **하류는 상류의 `contract` 패키지만 참조한다.** 상류의 `domain`·`application` 내부 직접 참조 금지.
3. **상류는 하류를 모른다.** 약속된 `contract`만 안정적으로 유지.
4. 양방향/순환 참조가 보이면 코드로 우회하지 말고 **경계를 재논의**한다 — 방향을 정리(받는 쪽이 contract 제공/이벤트)하거나, 운명 공동체면 **두 컨텍스트를 병합**.

## 5. 상류의 공개 창구 — `contract` (OHS + PL)

상류는 오직 `module/<bc>/contract`로만 외부에 공개한다. 나머지는 비공개 내부 구현.

- **OHS (Open Host Service)** = `XxxContract` 인터페이스. 호출자는 이 인터페이스만 본다.
- **PL (Published Language)** = 계약에 쓰는 공개 타입(record 등). **도메인 엔티티를 직접 노출하지 않는다** — 내부 모델/스키마 변경의 충격을 완충.
- 구현은 상류 `application`에서: (A) 기존 서비스가 `implements`, 또는 (B) 외부용 브릿지 서비스로 분리. 어느 쪽이든 **하류엔 인터페이스만** 드러난다.

(코드 예시: [reference/collaboration.md](./reference/collaboration.md))

## 6. 하류의 선택 — Conformist vs ACL

하류가 결합 수준을 **스스로** 정한다. 정답은 없고 변경 빈도·중요도로 고른다.

| | **Conformist (순응)** | **ACL (부패 방지 계층)** |
| --- | --- | --- |
| 상류 의존 침투 | `application`까지 직접 | `infrastructure/adapter`로 격리 |
| 데이터 타입 | 상류 PL 그대로 | 하류 고유 언어 타입 |
| 적합 | 표준적·안정적 상류 (결재·파일 등) | 상류 불안정 / 도메인 독립 보호 / 테스트 격리 |
| 비용 | 낮음 | 중간 (포트+어댑터) |

- **Conformist**: 하류 `application`이 상류 `contract`·PL을 직접 임포트해 사용.
- **ACL**: 하류가 `application/command/provider`에 **자기 언어의 포트**를 선언하고, `infrastructure/adapter`가 상류 contract를 호출·번역. 하류 `application`은 상류 존재를 모른다(테스트 시 가짜 포트 주입 용이).

(코드·판단 기준: [reference/collaboration.md](./reference/collaboration.md))

## 어디에 둘지 (요약)

| 역할 | 위치 |
| --- | --- |
| 도메인 개념·불변식·JPA 엔티티·Repository 인터페이스 | `module/<bc>/domain/<agg>/` |
| 쓰기 유스케이스 | `module/<bc>/application/command/` |
| 조회 서비스·쿼리 | `module/<bc>/application/query/` |
| 외부 연동 어댑터·Repository 구현 | `module/<bc>/infrastructure/adapter/` |
| 타 컨텍스트에 공개하는 API·공용 타입 | `module/<bc>/contract/` (상류만) |
| 도메인 무관 공통/유틸 | `common/` · `util/` |

## 컨벤션 위임

`contract` 패키지명, PL 접미사 규칙, 레이어 명명, 한 컨텍스트를 여러 명이 나눠 맡는 방식 등 **세부는 팀 컨벤션**이다(미정 가능). 이 스킬의 예시 이름·구조는 권장일 뿐 강제가 아니다.

## 참고

- [reference/collaboration.md](./reference/collaboration.md) — contract(OHS/PL) 코드, Conformist/ACL 코드·비교, 양방향 처리, PR 체크리스트, 용어집.
