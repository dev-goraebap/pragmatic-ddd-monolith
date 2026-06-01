# dependency-rules

> 세 레이어의 참조 그래프, 레이어 내 슬라이스 격리, BC 경계 정합, 그리고 기계적으로 잡을 수 있는 위반 체크리스트. SKILL.md "핵심" 섹션의 심화.

## 언제 펼치는가

- 참조 방향이 맞는지, 어떤 import가 금지인지 따질 때.
- BC를 나누거나 합치거나 이름을 정리할 때.
- CI에 넣을 정적 의존성 규칙을 세울 때.

---

## 1. 레이어 간 참조 (항상 위→아래)

```
app/<feature>/   →   module/<bc>/   →   shared/
```

| 방향 | 허용 | 이유 |
| --- | --- | --- |
| `app → module` | ✅ | app은 여러 module을 주입받아 유스케이스를 조립한다. |
| `app → shared`, `module → shared` | ✅ | shared는 최하위 기술층. 누구나 쓴다. |
| `module → app` | ❌ | 도메인은 화면 계약(DTO·라우트)을 몰라야 한다. |
| `module → module` | ❌ | BC 간 조립은 항상 app이 중재한다. |
| `shared → app`, `shared → module` | ❌ | shared는 비즈니스 독립이어야 한다. |

여러 BC를 조합해야 할 때는 app 서비스가 각 module의 Repository·Domain Service를 직접 주입받아 처리한다. 이것이 아웃바운드 포트나 이벤트 우회를 없애는 핵심이다.

## 2. 레이어 내 슬라이스 격리

슬라이스는 한 레이어 안의 한 단위(`app`의 기능, `module`의 BC, `shared`의 fragment)를 뜻한다.

| 규칙 | 허용 | 비고 |
| --- | --- | --- |
| `app/<a>` ↔ `app/<b>` | ❌ | 기능 슬라이스끼리 직접 참조 금지. |
| `module/<x>` ↔ `module/<y>` | ❌ | BC끼리 직접 참조 금지. |
| `shared/<f1>` → `shared/<f2>` | ✅(단방향) | 순환만 없으면 fragment 간 참조 허용. |

### app↔app 참조가 필요해 보일 때

거의 항상 설계 신호다. 다음 순서로 해소한다.

1. 그게 비즈니스 규칙인가? → module 도메인(메서드/Policy/Spec/Domain Service)으로 내린다.
2. 횡단 글루(Guard·Interceptor·Decorator·공통 DTO)인가? → `app/shared`.
3. 읽기 데이터가 필요한가? → Query Service가 직접 JOIN한다. 다른 app 슬라이스를 부르지 않는다.
4. 그래도 남는 순수 코드 중복은 팀의 기술적 판단(OOP 합성 등)에 맡긴다. → [project-convention.md](./project-convention.md)

app 슬라이스가 다른 app 슬라이스의 `service`를 호출하고 싶어진다면, 십중팔구 그 로직은 도메인이라 module로 내려가야 한다는 신호다.

## 3. shared 3종의 경계

| 위치 | 담는 것 | 담지 않는 것 |
| --- | --- | --- |
| `shared/` (top) | DB 커넥션·로깅·설정·순수 유틸 등 비즈니스 무관 기술 | 도메인 냄새가 나는 모든 것 |
| `app/shared` | Guard·Interceptor·Decorator·공통 DTO | 오케스트레이션 service |
| `module/shared` | 크로스-BC 공유 도메인 커널(공통 VO·도메인 베이스 클래스) | 특정 BC에 속한 것 |

판단법은 간단하다. 도메인 냄새가 나면 top-level `shared/`가 아니다. 여러 BC가 공유하는 도메인 개념이면 `module/shared`, 특정 BC 것이면 그 `module/<bc>`로 간다.

## 4. BC 경계 정합

BC를 나누거나 합칠 때 보는 신호:

- 변경 동조성 — 늘 함께 바뀌는 두 BC는 한 BC일 수 있다(병합 후보). git log로 동반 변경 빈도를 확인한다.
- 호출 방향 집중 — app이 두 BC를 거의 항상 한 방향으로 묶으면 한쪽이 하위 개념일 수 있다.
- 용어 충돌 — 같은 단어가 BC마다 다른 뜻이면 분할, 다른 단어가 같은 개념이면 이름을 통일한다.

전략은 넷이다 — 추출(sub-BC 분리) / 분리(split) / 병합(merge) / 이름 변경(rename). 실제 이동은 [migration.md](./migration.md).

## 5. 기계적 위반 체크리스트

정적 분석이나 리뷰로 잡는다.

1. `module/*` → `app/*` import — 금지
2. `module/<a>/` → `module/<b>/` import — 금지
3. `shared/` → `app/*` 또는 `module/*` import — 금지
4. `app/<a>` ↔ `app/<b>` 직접 import — 금지
5. Repository가 `domain/` 밖에 선언 — 누수
6. Query Service가 `module/` 안에 있음 — 위치 위반(역방향 의존)
7. Controller에 비즈니스 분기 로직 — 책임 위반
8. Application Service에 SQL/ORM 직접 호출 — 책임 위반(읽기 제외)
9. `tx` 객체가 service 시그니처에 노출 — UoW 캡슐화 위반
10. `app/shared`에 오케스트레이션 service 존재 — 글루 전용 위반

## 프레임워크 노트

레이어 간 참조 규칙은 정적 도구로 강제할 수 있다.

- NestJS / TS: `dependency-cruiser` 또는 `eslint-plugin-boundaries`로 1~4번 룰 강제.
- Spring: `ArchUnit`으로 패키지 의존성 룰 강제. 패키지 이동 시 `@ComponentScan` 경로 확인.
- ASP.NET Core: `NetArchTest` 또는 `ArchUnitNET`. 어셈블리·네임스페이스 경계로도 일부 강제 가능.

`@Module`·패키지·어셈블리 구조를 위 폴더 경계와 일치시키면, 위반이 컴파일·린트 단계에서 드러난다.
