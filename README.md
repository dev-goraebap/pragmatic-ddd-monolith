# pragmatic-ddd-monolith

> **단일 배포 모놀리식**에서 DDD를 실용적으로 빌려 바운디드 컨텍스트를 패키지로 나누는 설계 스킬.
> 객체지향 + DI 프레임워크(Spring Boot, NestJS, .NET 등)에서 두루 쓸 수 있고, 예시는 Spring Boot/Java로 듭니다.
> 모듈을 따로 배포하지 않습니다(그건 MSA). 정석 DDD를 교조적으로 따르기보다 생산성과의 타협을 우선합니다.

## 설치

```bash
npx skills add dev-goraebap/pragmatic-ddd-monolith
```

## 무엇을 하는가

기능 기반 구조에서 출발해, 두 가지 경계(**컨텍스트**와 **레이어**)를 명확히 나누고 컨텍스트 간 의존을 단방향으로 다스립니다.

- **패키지 구조** — `common/` · `config/` · `util/` · `module/<bc>/`.
- **컨텍스트 내부 레이어** — `domain/`(애그리거트 + Repository 인터페이스) · `application/{command,query}`(CQS) · `infrastructure/adapter` · `contract/`(상류일 때만: OHS + PL).
- **CQS** — 쓰기는 도메인 Repository로 불변식 검증, 읽기는 `ReadQuery`로 화면 DTO(크로스 컨텍스트 DB 직접 JOIN 허용).
- **컨텍스트 간** — 하류 → 상류 단방향, 하류는 상류의 `contract`만 참조. 하류는 **Conformist**(직접 사용) 또는 **ACL**(자기 포트 + 어댑터 번역)을 선택.
- **도메인** — 순수성·자가검증을 지키되, 실용성을 위해 **ORM 애노테이션(JPA 등) 직접 사용을 허용**.

구성:

- [SKILL.md](./SKILL.md) — 본체(전제·두 경계·패키지·레이어·CQS·상하류·contract·Conformist/ACL·배치표).
- [reference/collaboration.md](./reference/collaboration.md) — contract(OHS/PL) 코드, Conformist/ACL 코드·비교, 양방향 처리, PR 체크리스트, 용어집.

## 적용 범위

단일 배포 모놀리식, 객체지향 + DI 프레임워크(Spring Boot, NestJS, .NET 등). **모듈 별도 배포·MSA·분산(결과적 일관성)은 적용 범위 밖.**

## 라이선스

Apache-2.0. [LICENSE](./LICENSE) 참조.
