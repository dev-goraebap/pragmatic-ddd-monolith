---
name: backend-pragmatism-design
description: "Use when structuring or refactoring a backend in a monolith with a single database (NestJS, Spring Boot, ASP.NET Core, etc.) and the question is where code goes and how to manage coupling between feature modules. Starts from a feature-based structure and progressively resolves coupling by borrowing DDD pragmatically: group related modules into bounded contexts (core/generic/support), structure each context by complexity (flat vs layered, tiered), then pick a cross-context decoupling method — folder-layer split, OpenHostService + dependency inversion, or in-process synchronous events — plus shallow CQRS read models. Borrows DDD only as a language for drawing boundaries: aggregates, entities, value objects. Triggers on '이 코드 어디 둬', '모듈 결합 어떻게 풀어', 'BC로 묶어', '컨텍스트 간 참조', '이벤트로 분리', 'where does this go', 'how to decouple modules'. Options-based, not prescriptive: leaves folder names and layout to per-project convention. Monolith + single DB only. Not for MSA, multi-DB, or distributed designs."
license: Apache-2.0
metadata:
  version: "0.5.0"
  author: dev-goraebap
---

# Backend Pragmatism Design

모놀리식 + 단일 DB 백엔드에서 **"이 코드 어디 두지, 모듈 간 결합 어떻게 풀지"** 를 다루는 설계 스킬입니다. 정답 하나를 강요하지 않고, **상황에 맞는 선택지**를 제시합니다. DDD는 통째로 따르지 않고 "경계를 긋는 언어"만 실용적으로 빌립니다.

배경과 "왜"는 두 글에 있습니다 — [경계를 긋는 언어, 도메인 주도 설계](https://goraebap.xyz/posts/language-of-boundaries/), [백엔드 실용주의 디자인](https://goraebap.xyz/posts/backend-pragmatism-design/). 용어가 낯설면 [reference/concepts.md](./reference/concepts.md)를 먼저 보세요. 예제 코드: [tiny-hr](https://github.com/dev-goraebap/tiny-hr).

## 전제

- **모놀리식 + 단일 DB.** (모든 컨텍스트가 하나의 DB를 공유) MSA·다중 DB·분산(결과적 일관성)은 적용 범위 밖.
- OOP와 DI를 제공하는 프레임워크(NestJS, Spring Boot, ASP.NET Core 등).

## 이 스킬의 태도

- **단순함 우선.** 과한 규정보다 상황별 선택.
- **세부는 프로젝트 컨벤션에 위임.** 폴더명·파일 배치·suffix·레이어 이름 등은 **권장**일 뿐 강제하지 않는다. (아래 "컨벤션 위임" 참조)

> 예제는 사내 연차관리 플랫폼 **tiny-hr**(사원·부서·직급 관리, 휴가 신청 → 결재 → 알림)을 사용합니다.

---

## 1. 출발: 기능 기반 구조

기능 단위로 폴더를 나눈다(Screaming Architecture). 폴더 하나하나를 **모듈**이라 부른다. 모듈은 보통 명사(엔티티) 하나당 만들어지고, 대체로 **애그리거트 단위**와 겹친다.

```
com.example.tinyhr
├── employee/        # 권장 예시 — 내부 배치는 팀 컨벤션
│   ├── EmployeeController
│   ├── EmployeeService
│   ├── Employee           # JPA 엔티티
│   ├── EmployeeRepository
│   └── dto/
├── department
├── rank
├── user_account
├── role
├── auth
├── notification
├── approval_request
└── leave_request
```

## 2. 결합은 자연스럽게 생긴다

작은 규모에서도 모듈끼리 결합이 생긴다 (사원 온보딩 시 `employee`가 `user_account`를, 결재 승인 시 `approval_request`가 `notification`을 호출 등). 원인은 하나다:

> **요구사항의 흐름과 도메인 모델은 서로 다른 축이다.** 모듈은 애그리거트 단위인데, 하나의 요구사항이 한 애그리거트로 끊어지는 일은 드물다. 그래서 오케스트레이션 서비스가 다른 모듈의 애그리거트를 끌어다 쓰게 된다.

이게 곧 나쁜 건 아니다. 참조가 **단방향이고 관리 가능한 수준이면 그대로 둬도 된다.** 다만 "모듈이 잘 분리됐나, 이 참조가 합리적인가"라는 의문이 든다면, 그때가 **자신이 타협할 기준을 정할 때**다.

## 3. 관련 모듈을 컨텍스트로 묶기

협력하는 모듈들은 *제각각 독립된 기능*이라기보다 **함께 하나의 도메인 모델을 이루는 조각**이다. 그 도메인 모델이 일관되게 통용되는 경계가 **바운디드 컨텍스트(이하 컨텍스트)** 다. 하위 도메인은 느슨하게 **핵심(Core)·일반(Generic)·지원(Support)** 으로 본다. (핵심 = 이름값 크거나, 복잡도 높거나, 다른 영역을 끌어다 흐름을 주도. 핵심은 여러 개일 수 있음.)

```
com.example.tinyhr            # 권장 예시
├── leave/            # 핵심: 휴가 (leave_request …)
├── approval/         # 핵심 흐름 허브: 결재 (approval_request)
├── organization/     # 지원: 조직 (employee, department, rank)
├── iam/              # 일반: 인증·계정 (user_account, role, auth)
└── notification/     # 지원: 알림 (모듈 하나여도 컨텍스트)
```

- **같은 컨텍스트 안에서는 모듈끼리 자유롭게 참조한다.** 협력이 잦은 게 자연스럽다. 안에서까지 우회용 공유 서비스를 만드는 건 마이너스.
- **컨텍스트 ≠ 크기.** 모듈(애그리거트)이 하나뿐이어도 컨텍스트다. 잣대는 개수가 아니라 **자기만의 언어와 책임**(알림은 '수신자·채널·발송 이력'). 작을 땐 '컨텍스트냐 기능이냐' 못 박지 말고, 언어가 또렷해지거나 짝이 생기면 키운다.
- **결합이 있다고 무작정 같은 컨텍스트로 합치지 않는다.** 사원 온보딩이 `employee`(organization)와 `user_account`(iam)를 엮어도, 둘은 책임(인사 vs 보안)이 달라 다른 컨텍스트로 남는다. 경계를 넘는 결합은 결함이 아니라 협력이다.
- **경계는 양방향으로 변한다.** 나눴던 모듈이 다시 묶이기도, 하나로 본 컨텍스트가 복잡해지며 쪼개지기도 한다.

자세히는 [reference/concepts.md](./reference/concepts.md).

## 4. 컨텍스트 안을 어떻게 구성할까 (차등 적용)

컨텍스트로 묶었으면 그 *안*을 어떻게 배치할지 정한다. 두 갈래이고, **복잡도에 따라 차등 적용**한다.

- **선택지 A — 모듈 그대로(플랫).** 컨텍스트 폴더 밑에 모듈을 평평하게 둔다. 기능 응집이 살아 있고 추가 투자 0. 애그리거트 한두 개에 자기 영역 CRUD 위주면 충분(예: `organization`, `notification`).
- **선택지 B — 레이어로 나눈다.** 컨텍스트 안을 `domain` / `application` / `adapter`로 가른다. 의존은 모두 안쪽 `domain`을 향한다(입력 어댑터 → application → domain, 출력 어댑터는 domain이 정의한 인터페이스를 구현 = DIP). 도메인 모델 축과 요구사항 흐름 축을 갈라, 흐름이 어디서 반복되고 무엇이 재사용되는지 한눈에 보이게 한다. (이름·3분할은 예시일 뿐 강제 아님.)

| 티어 | 컨텍스트 | 구조 |
| --- | --- | --- |
| 1티어 | 단일·소수 애그리거트, 횡단 오케스트레이션 거의 없음 (`organization`, `notification`) | A — 플랫 |
| 2티어 | 다중 애그리거트 + 횡단 오케스트레이션 잦음 (`leave`) | B — 레이어 |

> 레이어는 **복잡한 곳에만** 준다. *핵심이라서가 아니라 복잡해서.* 지원·일반이라도 복잡하면 레이어로, 핵심이라도 단순하면 플랫. (상세: [reference/decoupling.md](./reference/decoupling.md))

## 5. 컨텍스트 간 결합 풀기 (선택지)

컨텍스트를 잘 나눠도 **컨텍스트 사이 참조**는 남는다. 이게 진짜 풀 영역이다. 참조하는 이유는 셋:

① 다른 컨텍스트의 **데이터 조회** · ② 다른 컨텍스트 기능이 **한 트랜잭션으로 보장** · ③ **기술적 외부 모듈** 사용.

푸는 방법은 상황에 따라 고른다. (상세·트레이드오프: [reference/decoupling.md](./reference/decoupling.md))

| 방법 | 핵심 | 언제 |
| --- | --- | --- |
| **그냥 둔다** | 단방향 참조를 인정 | 컨텍스트 2~3개, 참조가 단방향 |
| **폴더 레이어 2분할 (app/module)** | 참조당하는 도메인을 하위(`module`)로, 컨트롤러+오케스트레이션을 상위(`app`)로. 같은 레이어 직접 참조 금지. 상·하위는 1:1 대칭 아님(책임이 다름, N:M) | 중규모↑, 결합이 늘 때 |
| **OpenHostService(OHS) + 의존 역전(SPI)** | 제공 컨텍스트가 공개 입구(OHS)만 노출. 역참조가 생기면 허브가 SPI 인터페이스를 소유하고 상대가 구현·등록(전략+DIP)해 단방향 유지 | 모든 도메인을 여는 폴더 방식이 부담스러울 때 |
| **동기 인-프로세스 이벤트** | 참조를 끊음. 발행 모듈은 사실만 알리고 핸들러가 처리. 같은 tx 동기 실행 | 부수효과·알림 등 호출 방향을 끊고 싶을 때 |

> OHS의 "API"는 HTTP Web API(컨트롤러)가 아니라 **같은 프로세스 안에서 다른 패키지가 직접 호출하는 코드 레벨 클래스/인터페이스**다.

## 6. 얕은 CQRS — 조회 분리

쓰기(write)는 컨텍스트 경계·규칙을 엄격히 지킨다. 조회(read)는 그 틀을 똑같이 따를 필요가 없다. 조회는 두 갈래로 푼다:

- **화면용 데이터**: 쓰기 모델을 거치지 말고 **전용 쿼리 서비스**가 화면 모양대로 직접 가져온다. 컨텍스트를 가로질러 조인하거나 DB 뷰를 써도 된다.
- **비즈니스 흐름용 조회**: 다른 컨텍스트 데이터를 검증·입력으로 쓸 때. **내 컨텍스트 언어로 된 읽기 전용 모델**(예: 알림 컨텍스트가 `employees` 테이블을 `Recipient`로 매핑)을 만들어 자기 리포지토리로 조회한다. (이벤트는 *행위 호출*이라 조회 문제를 대신 풀어주지 않는다.)

> 단, 읽기 전용 모델이 늘 정답은 아니다. 가져온 데이터가 **내 컨텍스트 언어로 매끄럽게 표현되지 않으면**, 억지로 감싸지 말고 *외부 컨텍스트의 데이터임을 그대로 인정*하는 편이 나을 수 있다.

## 7. 실용 노트 (과하지 않게)

- **트랜잭션**: 모놀리식에선 여러 애그리거트를 한 tx로 묶어도 된다("1 tx = 1 애그리거트"는 분산 전제). 단 **외부 I/O(이메일·푸시·업로드)는 tx 밖**.
- **rich vs anemic**: 규칙이 복잡하고 변동 잦은 핵심일수록 도메인 모델에 규칙을 모으면(rich) 보상이 크다. 단순 CRUD(지원 하위도메인, 예: 공지)는 anemic이어도 괜찮다.
- **ORM 엔티티**: 모놀리식이면 `@Entity` 하나가 영속성과 도메인 엔티티를 겸해도 된다. 단 데이터 그릇으로 두지 말고 **행동·규칙을 담아야** 도메인 엔티티 자격을 얻는다.
- **값 객체(VO)**: 정말 필요할 때만. 이메일·금액처럼 일반적인 값은 검증 라이브러리로 충분하다.

## 의사결정 흐름 (요약)

```
무질서한 기능 기반 구조
   └─ 모듈들을 응집할 수 있나?
        ├─ Yes → 컨텍스트로 묶기            (§3)
        └─ No  → 개별 독립 컨텍스트로 격리
              └─ 컨텍스트 안 비즈니스·횡단 오케스트레이션이 복잡한가?  (§4)
                   ├─ No  → 1티어 플랫
                   └─ Yes → 2티어 domain/application/adapter 레이어
                        └─ 컨텍스트 경계 간 참조가 생기나?           (§5~6)
                             ├─ 물리 제어   → app/module 폴더 레이어
                             ├─ API 정의    → OpenHostService + SPI
                             ├─ 메시지 단절 → 인-프로세스 동기 이벤트
                             └─ 조회 복잡도 → 얕은 CQRS / 읽기 전용 모델
```

> 정답 아키텍처는 없다. 비즈니스 성장 속도와 팀 규모·역량 사이의 트레이드오프에서 타협점을 찾는다.

## 컨벤션 위임

폴더명, 모듈 내부 파일 배치, 파일 suffix, 레이어 이름(`domain`/`application`/`adapter` 등) — **세부는 모두 팀/프로젝트 컨벤션**이다. 이 스킬의 폴더 예시는 전부 *권장*일 뿐 강제가 아니다. 프로젝트 규약을 한 문서로 적어두길 **권장**하되, 그 문서의 **위치나 이름도 강제하지 않는다.**

## 참고

- [reference/concepts.md](./reference/concepts.md) — 빌려온 DDD 어휘(하위도메인·컨텍스트·도메인 모델·애그리거트).
- [reference/decoupling.md](./reference/decoupling.md) — 컨텍스트 안 구성(플랫/레이어/티어)과 컨텍스트 간 결합 푸는 방법, 읽기 모델의 상세·트레이드오프.
- [tiny-hr](https://github.com/dev-goraebap/tiny-hr) — 예제 프로젝트. 블로그 두 글 — 배경과 "왜".
