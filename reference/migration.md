# migration

> 기존 코드베이스(레이어-first / BC-first / URL-중첩 app)를 이 구조로 옮기는 단계. 현 상태 진단 → 슬라이스 재배치 → 참조 위반 교정 → 과잉 추상화 제거.

## 언제 펼치는가

- 기존 서비스를 이 구조로 마이그레이션할 때.
- 엔드포인트·폴더 구조를 한 번도 점검한 적 없어 정합성을 모를 때.
- 의존성 위반을 발견해 교정 계획이 필요할 때.

---

## 1. 현 상태 진단 (ARCHITECTURE 스냅샷)

먼저 현재를 그대로 읽어 표로 만든다.

1. 최상위 폴더가 레이어-first(`presentation/`, `application/`, `domain/` 아래 BC)인지, BC-first(`org/` 아래 자체 레이어)인지, 혼합/URL-중첩인지 식별한다.
2. BC(도메인 응집 단위) 인벤토리를 만든다 — 각 BC의 Aggregate, Repository, 외부 어댑터.
3. [dependency-rules.md](./dependency-rules.md) §5 체크리스트로 참조 위반 목록(파일·라인·룰 ID)을 작성한다.
4. 화면 조회 로직이 어디 있는지, 특히 `infrastructure`에 박힌 Query를 표시한다.

산출물을 `ARCHITECTURE.md`로 남기면 이후 작업의 기준선이 된다.

## 2. 타깃 매핑

| 현재 | → | 타깃 |
| --- | --- | --- |
| 비즈니스 불변 규칙(도메인) | → | `module/<bc>/<aggregate>/domain/` |
| 도메인의 영속성(Repository 구현, ORM 엔티티) | → | 같은 애그리게이트의 `infrastructure/` |
| 컨트롤러 + 오케스트레이션 서비스 + DTO | → | `app/<feature>/` (기능 슬라이스) |
| 화면 조회 로직(JOIN→DTO) | → | `app/<feature>/*-query.service` |
| 채널 핸들러(크론·컨슈머·웹훅) | → | 해당 기능 슬라이스의 프래그먼트 |
| 횡단 글루(Guard·Interceptor·Decorator) | → | `app/shared` |
| 크로스-BC 공유 도메인 개념 | → | `module/shared` |
| 비즈니스 무관 기술 | → | `shared/`(top) |

배치가 헷갈리면 한 줄 휴리스틱으로 돌아간다 — 불변 규칙이면 `module`, 화면·외부·액터 결합이면 `app`, 비즈니스 무관 기술이면 `shared`.

## 3. 단계별 이동

안전망을 먼저 깔고, 변경을 작게 쪼갠다.

1. 안전망 — 옮길 범위에 Solitary/Sociable 테스트가 있는지 확인하고, 없으면 핵심 경로부터 추가한다.
2. module 먼저 안정화 — 도메인+영속성을 애그리게이트 폴더로 모으고 Solitary 테스트를 통과시킨다.
3. app 재배치 — URL 계층을 베낀 중첩 폴더를 기능 슬라이스로 평탄화하고, 채널을 프래그먼트로 내린다.
4. 참조 위반 교정 — §1에서 잡은 위반을 방향대로 정리한다.
   - `module → module` → app으로 조립을 옮긴다.
   - `module → app`(특히 Query Service) → Query Service를 app 슬라이스로 옮긴다.
   - `app ↔ app` → 도메인이면 module로 내리고, 글루면 `app/shared`로 옮긴다.
5. 과잉 추상화 제거 — 테스트 더블 슬롯이 아닌 인터페이스, 단일 구현 포트, 죽은 ViewModel 매퍼, `Clock`/`IdGenerator` 추상화를 제거한다.
6. 회귀 확인 — 매 단계 테스트와 정적 의존성 룰(프레임워크 노트)을 통과시킨다.

## 4. 흔한 교정 케이스

- Query Service가 `module/<bc>/infrastructure/`에 있음 → app 슬라이스로 옮긴다. 중간 ViewModel·변환 매퍼는 삭제하고 Response DTO를 직접 생성한다.
- 오케스트레이션이 특정 BC 안에 있어 BC끼리 호출 → app으로 격상하고 각 module을 직접 주입한다.
- 채널마다 복제된 유스케이스 → 단일 `service`로 합치고 채널 핸들러를 얇게 만든다.
- anemic 도메인 + app 서비스에 비즈니스 분기 누적 → 규칙을 도메인 메서드/Policy/Spec으로 내린다. ([module-layer.md](./module-layer.md), [app-layer.md](./app-layer.md) §6)

## 프레임워크 노트

- NestJS: `@Module` 트리 재배치, providers/exports 갱신. 정적 룰은 `dependency-cruiser`/`eslint-plugin-boundaries`.
- Spring: 패키지 이동, `@ComponentScan` 경로 확인. 룰 강제는 `ArchUnit`.
- ASP.NET Core: 어셈블리·네임스페이스 정리, DI 등록 갱신. 룰 강제는 `NetArchTest`/`ArchUnitNET`.
