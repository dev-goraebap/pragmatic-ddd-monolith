---
name: backend-pragmatism-design
description: "Use when structuring or refactoring a backend in a monolith with a single database (NestJS, Spring Boot, ASP.NET Core, etc.) and the question is where code goes and how to manage coupling between feature modules. Starts from a feature-based structure and progressively resolves coupling by borrowing DDD pragmatically: group related modules into bounded contexts (subdomains: core/generic/support), then pick a decoupling method per situation — folder-layer split, DIP interfaces, or in-process synchronous events — plus shallow CQRS read models. Borrows DDD only as a language for drawing boundaries: aggregates, entities, value objects, rich vs anemic models. Triggers on '이 코드 어디 둬', '모듈 결합 어떻게 풀어', 'BC로 묶어', '컨텍스트 간 참조', '이벤트로 분리', 'where does this go', 'how to decouple modules', 'split into bounded contexts'. Options-based, not prescriptive: offers a menu and leaves folder names and layout to per-project convention. Monolith + single DB only. Not for MSA, multi-DB, or distributed designs."
license: Apache-2.0
metadata:
  version: "0.3.0"
  author: dev-goraebap
---

# Backend Pragmatism Design

모놀리식 + 단일 DB 백엔드에서 **"이 코드 어디 두지, 모듈 간 결합 어떻게 풀지"** 를 다루는 설계 스킬입니다. 정답 하나를 강요하지 않고, **상황에 맞는 선택지**를 제시합니다. DDD는 통째로 따르지 않고 "경계를 긋는 언어"만 실용적으로 빌립니다.

배경과 "왜"는 두 글에 있습니다 — [경계를 긋는 언어, 도메인 주도 설계](https://goraebap.xyz/posts/language-of-boundaries/), [백엔드 실용주의 디자인](https://goraebap.xyz/posts/backend-pragmatism-design/). 용어가 낯설면 [reference/concepts.md](./reference/concepts.md)를 먼저 보세요.

## 전제

- **모놀리식 + 단일 DB.** MSA·다중 DB·분산(결과적 일관성)은 적용 범위 밖.
- OOP와 DI를 제공하는 프레임워크(NestJS, Spring Boot, ASP.NET Core 등).

## 이 스킬의 태도

- **단순함 우선.** 과한 규정보다 상황별 선택.
- **세부는 프로젝트 컨벤션에 위임.** 폴더명·파일 배치·suffix·레이어 이름 등은 **권장**일 뿐 강제하지 않는다. (아래 "컨벤션 위임" 참조)

---

## 1. 출발: 기능 기반 구조

기능 단위로 폴더를 나눈다(Screaming Architecture). 폴더 하나하나를 **모듈**이라 부른다. 모듈은 대체로 **애그리거트 단위**(함께 일관성을 지키는 변경의 단위)다.

```
src/
├── auth
├── user
├── post/            # 권장 예시 — 내부 배치는 팀 컨벤션
│   ├── post.controller
│   ├── post.service
│   ├── post.entity
│   ├── post.repository
│   └── dto/
├── comment
├── notification
└── tag
```

## 2. 결합은 자연스럽게 생긴다

작은 규모에서도 모듈끼리 결합이 생긴다 (회원가입 시 `auth`가 `user`를, 발행 시 `post`가 `notification`을 호출 등). 원인은 하나다:

> **요구사항의 흐름과 도메인 모델은 서로 다른 축이다.** 모듈은 애그리거트 단위인데, 하나의 요구사항이 한 애그리거트로 끊어지는 일은 드물다. 그래서 오케스트레이션 서비스가 다른 모듈의 애그리거트를 끌어다 쓰게 된다.

이게 곧 나쁜 건 아니다. 참조가 **단방향이고 관리 가능한 수준이면 그대로 둬도 된다.** 다만 "모듈이 잘 분리됐나, 이 참조가 합리적인가"라는 의문이 든다면, 그때가 **자신이 타협할 기준을 정할 때**다.

## 3. 관련 모듈을 컨텍스트로 묶기

무심코 나눈 모듈들은 더 큰 묶음으로 다시 묶일 수 있다. 그 묶음이 **바운디드 컨텍스트(이하 컨텍스트)** 다. 하위 도메인은 느슨하게 **핵심(Core)·일반(Generic)·지원(Support)** 으로 본다. (핵심 = 이름값 크거나, 복잡도 높거나, 다른 영역을 끌어다 흐름을 주도. 핵심은 여러 개일 수 있음.)

```
src/                    # 권장 예시
├── content/            # 핵심: 콘텐츠 (post, category, comment, tag)
├── iam/                # 일반: 인증·계정 (auth, user)
└── notification/       # 지원: 알림
```

- **한 컨텍스트 = 여러 모듈(애그리거트).** 그래야 그 컨텍스트의 도메인 모델이 다 모인다.
- **같은 컨텍스트 안에서는 모듈끼리 자유롭게 참조한다.** 협력이 잦은 게 자연스럽다. 안에서까지 우회용 공유 서비스를 만드는 건 마이너스.

자세히는 [reference/concepts.md](./reference/concepts.md).

## 4. 컨텍스트 간 결합 풀기 (선택지)

컨텍스트를 잘 나눠도 **컨텍스트 사이 참조**는 남는다. 이게 진짜 풀 영역이다. 참조하는 이유는 셋:

① 다른 컨텍스트의 **데이터 조회** · ② 다른 컨텍스트 기능이 **한 트랜잭션으로 보장** · ③ **기술적 외부 모듈** 사용.

푸는 방법은 상황에 따라 고른다. (상세·트레이드오프: [reference/decoupling.md](./reference/decoupling.md))

| 방법 | 핵심 | 언제 |
| --- | --- | --- |
| **그냥 둔다** | 단방향 참조를 인정 | 컨텍스트 2~3개, 참조가 단방향 |
| **폴더 레이어 2분할** | 참조당하는 쪽(도메인+인프라)을 하위로, 컨트롤러+오케스트레이션을 상위로. 같은 레이어 직접 참조 금지 | 중규모 이상, 결합이 늘어남 |
| **DIP 인터페이스** | 내부를 감추고 필요한 것만 노출, 순환 조짐 시 의존성 역전 | 한 레이어 유지하며 캡슐화·순환 차단 |
| **동기 인-프로세스 이벤트** | 참조를 끊음. 발행 모듈은 사실만 알리고 핸들러가 처리. 같은 tx 동기 실행 | 부수효과·알림 등 호출 방향을 끊고 싶을 때 |

## 5. 얕은 CQRS — 조회 분리

쓰기(write)는 컨텍스트 경계·규칙을 엄격히 지킨다. 조회(read)는 그 틀을 똑같이 따를 필요가 없다. 조회는 두 갈래로 푼다:

- **화면용 데이터**: 쓰기 모델을 거치지 말고 **전용 쿼리 서비스**가 화면 모양대로 직접 가져온다. 컨텍스트를 가로질러 조인하거나 DB 뷰를 써도 된다.
- **비즈니스 흐름용 조회**: 다른 컨텍스트 데이터를 검증·입력으로 쓸 때. **내 컨텍스트 언어로 된 읽기 전용 모델**(예: 알림 컨텍스트의 `Recipient`)을 만들어 자기 리포지토리로 같은 테이블을 조회한다. (이벤트는 *행위 호출*이라 조회 문제를 대신 풀어주지 않는다.)

## 6. 실용 노트 (과하지 않게)

- **트랜잭션**: 모놀리식에선 여러 애그리거트를 한 tx로 묶어도 된다("1 tx = 1 애그리거트"는 분산 전제). 단 **외부 I/O(이메일·푸시·업로드)는 tx 밖**.
- **rich vs anemic**: 규칙이 복잡하고 변동 잦은 핵심일수록 도메인 모델에 규칙을 모으면(rich) 보상이 크다. 단순 CRUD(지원 하위도메인, 예: 공지)는 anemic이어도 괜찮다.
- **ORM 엔티티**: 모놀리식이면 `@Entity` 하나가 영속성과 도메인 엔티티를 겸해도 된다. 단 데이터 그릇으로 두지 말고 **행동·규칙을 담아야** 도메인 엔티티 자격을 얻는다.
- **값 객체(VO)**: 정말 필요할 때만. 이메일·금액처럼 일반적인 값은 검증 라이브러리로 충분하다.

## 컨벤션 위임

폴더명, 모듈 내부 파일 배치(컨트롤러/서비스/도메인/인프라를 어떻게 쪼갤지), 파일 suffix, 레이어 이름 등 **세부는 모두 팀/프로젝트 컨벤션**이다. 이 스킬의 폴더 예시는 전부 *권장*일 뿐 강제가 아니다. 프로젝트 규약을 한 문서로 적어두길 **권장**하되, 그 문서의 **위치나 이름도 강제하지 않는다.**

## 참고

- [reference/concepts.md](./reference/concepts.md) — 빌려온 DDD 어휘(하위도메인·컨텍스트·도메인 모델·애그리거트).
- [reference/decoupling.md](./reference/decoupling.md) — 결합 푸는 3가지 방법과 읽기 모델의 상세·트레이드오프.
- 블로그 두 글 — 배경과 "왜".
