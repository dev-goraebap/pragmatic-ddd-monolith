# backend-pragmatism-design

> OOP + DI 백엔드를 위한 **실용주의 레이어드 + 일부 DDD + 얕은 CQRS** 설계 스킬.
> 프레임워크 중립 (NestJS, Spring Boot, ASP.NET Core, Micronaut, Quarkus 등).

## 설치

```bash
npx skills add dev-goraebap/backend-pragmatism-design
```

또는 특정 에이전트 디렉터리에 직접:

```bash
npx skills add dev-goraebap/backend-pragmatism-design --skill backend-pragmatism-design
```

## 빠른 시작

```
/backend-pragmatism-design                       # 명령어 메뉴
/backend-pragmatism-design shape <feature>       # 코드 작성 전 유스케이스 계획
/backend-pragmatism-design critique <target>     # 종합 설계 리뷰
/backend-pragmatism-design refactor <target>     # 설계 어긋난 코드 교정
```

명령어 전체 목록은 [SKILL.md](./SKILL.md) 참조.

## 자기 규율: 원리 vs 실행 vs 컨벤션

본 스킬은 셋만 다룬다.

- **A. 설계 원리** - 프레임워크와 무관하게 "왜"가 성립하는 진술. SKILL.md / reference 본문에.
- **B. 프레임워크 실행 방식** - 같은 원리를 NestJS/Spring/.NET이 다르게 구현. 각 reference 하단 "프레임워크 노트"에.
- **C. 팀 컨벤션** - 같은 프레임워크 안에서도 팀이 다르게 선택 가능. **PROJECT.md**로 외부화.

본문에 B나 C가 섞이면 본 스킬은 특정 스택의 코드에서만 작동하는 것이 된다. 이 분류가 스킬의 자기 규율이다. 예: "audit 컬럼을 도메인 엔티티에서 빼라"는 NestJS+Drizzle에서는 자연스럽지만 JPA에서는 부담스럽다. 따라서 본문에는 "도메인 의도가 영속 부수 정보에 끌려가지 않도록 한다"(A)만 두고, 구체적 기법은 프레임워크 노트로 옮긴다.

## 무엇을 하는가

- 19개 서브커맨드(Setup / Design / Evaluate / Refine / Topic)로 백엔드 설계·리팩토링·리뷰 작업을 정형화.
- 라우터 패턴: 짧은 SKILL.md(공통 원칙 + 라우팅)와 각 서브커맨드별 reference 문서로 progressive disclosure.
- 프레임워크 중립 본문 + 프레임워크별 매핑 노트.

## 철학

(TODO: 사용자가 작성)

이 섹션에 다음을 담을 예정입니다.

- 왜 헥사고날/클린/완전 CQRS의 도그마를 따르지 않는가
- `app/<resource>/` ↔ `module/<bc>/` ↔ `shared/`의 단방향 의존이 무엇을 해결하는가
- 변동성이 큰 경계에만 DIP를 적용하는 기준
- Application Service를 BC 밖으로 끌어올린 이유
- 얕은 CQRS와 인프라스트럭처의 물리적 분산
- 모놀리식에서 BC 간 격리 비용에 대한 입장
- 테스트 3-tier와 폴더 구조의 대응
- 이 설계가 가져다주는 실질적 이점

## 라이선스

Apache-2.0. [LICENSE](./LICENSE) 참조.

## 영감

[impeccable](https://github.com/anthropics/skills) 스킬의 라우터+서브커맨드 패턴에서 영감을 얻었습니다.
