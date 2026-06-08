# decoupling — 컨텍스트 안 구성과 컨텍스트 간 결합

> 컨텍스트 *안*을 어떻게 배치하고, 컨텍스트 *사이* 참조를 어떻게 다스릴지. 실제로 풀 때 펼친다.
> 배경: [백엔드 실용주의 디자인](https://goraebap.xyz/posts/backend-pragmatism-design/) · 예제: [tiny-hr](https://github.com/dev-goraebap/tiny-hr).
> 예제 도메인: 사원·부서·직급(organization), 인증·계정(iam), 휴가(leave)·결재(approval), 알림(notification).

## 0. 컨텍스트 안 구성 (플랫 vs 레이어, 차등 적용)

컨텍스트로 묶었으면 그 안 배치를 정한다. 두 갈래이고 **복잡도에 따라 차등 적용**한다.

### 선택지 A — 모듈 그대로(플랫)
컨텍스트 폴더 밑에 모듈을 평평하게 둔다. 각 모듈이 자기 컨트롤러·서비스·도메인·리포지토리를 그대로 품는다. 기능 응집(스크리밍 아키텍처의 장점)이 살아 있고 **추가 투자 0**. 애그리거트 한두 개에 자기 영역 CRUD 위주면 충분.

```
organization/
├── employee/
├── department/
└── rank/
```

### 선택지 B — 레이어로 나눈다
컨텍스트 안을 `domain` / `application` / `adapter`로 가른다.

```
leave/
├── domain/                 # 핵심 규칙 (Entity, VO, Repository 인터페이스) — 애그리거트별
│   ├── leave_request/
│   └── annual_leave_balance/
├── application/            # 유스케이스 조율 (LeaveRequestService 등)
└── adapter/
    ├── web/                # 진입점 (LeaveController)
    └── persistence/        # 영속성 구현 (JpaLeaveRequestRepository)
```

의존은 모두 안쪽 `domain`을 향한다. 입력 어댑터(`adapter/web`)는 `application`을 거쳐 `domain`으로, 출력 어댑터(`adapter/persistence`)는 `domain`이 정의한 리포지토리 인터페이스를 구현하며 `domain`을 향한다(DIP). 요구사항이 흐르는 축(application)과 데이터가 함께 변하는 축(domain)을 갈라, **변경의 파급을 가둔다.** (스프링이면 리포지토리 인터페이스만 선언하면 런타임에 구현체가 주입돼 이 방향이 자연히 유지된다.)

> 이름(`domain`/`application`/`adapter`)과 3분할은 *절대 규칙이 아니라 한 가지 예*다. 핵심은 "요구사항의 동적 흐름과 도메인 모델의 정적 변경 단위를 분리해 제어한다". 팀에 맞는 이름·깊이로 바꿔도 된다.

### 차등 적용 — 1티어 / 2티어
모든 컨텍스트에 레이어를 씌우지 않는다. 가르는 기준 하나: **안에 여러 애그리거트가 얽히고, 그것들을 가로지르는 오케스트레이션이 잦은가.**

| 티어 | 컨텍스트 | 구조 |
| --- | --- | --- |
| 1티어 | 단일·소수 애그리거트, 횡단 오케스트레이션 거의 없음 (`organization`, `notification`) | A — 플랫 |
| 2티어 | 다중 애그리거트 + 횡단 오케스트레이션 잦음 (`leave`) | B — 레이어 |

> 레이어는 **복잡한 곳에** 준다. *핵심이라서가 아니라 복잡해서.* 지원·일반이라도 복잡하면 레이어로, 핵심이라도 단순하면 플랫.

---

## 컨텍스트 간 참조 — 이유 3가지

컨텍스트 밖을 참조하는 이유는 대개 셋이다. 이유에 따라 방법이 갈린다.

1. 다른 컨텍스트의 **데이터를 조회** → §읽기 모델(얕은 CQRS).
2. 다른 컨텍스트 기능이 **한 트랜잭션으로 함께 보장** → 방법 A / B / C.
3. **기술적 외부 모듈**(이메일·푸시·스토리지) 사용 → 보통 그냥 가져다 씀(인터페이스 없이 구체 클래스 + 테스트는 mock).

> 같은 컨텍스트 *안*의 모듈끼리는 굳이 아래를 쓰지 않는다 — 자유롭게 참조한다.

## 방법 A. 폴더 레이어 2분할 (app / module)

§0의 레이어 사고(`domain`/`application`/`adapter`)를 **컨텍스트 경계 *너머*로 확장**한 것. 참조당하는 컨텍스트의 도메인을 아래 레이어로 내려, 위쪽 여러 컨텍스트가 공평하게 쓰게 한다. 레이어가 늘수록 기능 기반의 장점이 옅어지니 **두 레이어**로.

- **상위 `app`**: 요청을 받아 흐름을 조율하는 컨트롤러·오케스트레이션 모듈(예: 사원 온보딩 API, 휴가 신청 API).
- **하위 `module`**: 핵심 규칙·모델이 응집된 컨텍스트 모듈들(`organization`, `iam`, `notification`…).

규칙: ① 상위 → 하위 참조 가능. ② **하위의 독립 컨텍스트끼리 직접 참조 금지.** ③ 같은 레이어 형제 모듈 직접 참조도 원칙 금지, 남는 예외는 팀 합의.

```
app (요청 조율)      사원온보딩 / 휴가신청 ...
        │  (공평하게 하위를 가져다 씀)
        ▼
module (도메인 응집) organization / iam / notification
   (다른 컨텍스트 직접 참조 ❌)
```

- **상·하위는 1:1 대칭이 아니다.** 위아래로 같은 모듈을 쪼갠 게 아니라 *책임*이 다르다 — 상위는 요청 흐름 조율, 하위는 도메인 응집. 상위 하나가 하위 여럿을 쓰고, 하위 하나가 상위 여럿에 쓰인다(**N:M**).
- 필요할 때 하나씩 내려도, 일관성을 위해 한 번에 분리해도 된다.
- 트레이드오프: 결합을 *끊는 게 아니라*, 모놀리식의 강점인 '모든 데이터 직접 접근'을 살리려 **타협·우회**하는 것. 상위가 하위의 구체 엔티티·리포지토리에 직접 손을 뻗으니 진정한 캡슐화는 희생된다. 이게 걸리면 방법 B로.
- 언제: 중규모 이상, 결합이 늘 때. (컨텍스트 2~3개·단방향이면 과하다 → 그냥 둬라.)

## 방법 B. OpenHostService(OHS) + 의존 역전(SPI)

헥사고날은 컨텍스트 간 모든 결합을 차단하려 양쪽에 포트·어댑터·ACL을 세워 학습 곡선·클래스 증폭이 크다. 대신 **다른 영역에 서비스를 제공할 만큼 성숙한 컨텍스트(특히 일반·지원 하위 도메인)의 역할을 인정**하고, 참조를 *없애는* 대신 **한쪽으로만 흐르게** 다스린다.

> 여기서 "API/서비스"는 **HTTP Web API(컨트롤러)가 아니라**, 같은 프로젝트 안에서 다른 패키지가 직접 자바/타입스크립트 메서드로 호출하는 **코드 레벨 클래스·인터페이스**다.

### 1) Open Host Service (OHS)
제공자 컨텍스트가 외부에 안정적으로 열어주는 입구를 정의하고, 클라이언트는 그 입구만 단방향으로 호출한다. 클라이언트는 제공자의 내부 영속성·제약을 알 필요가 없다.

예) 사원 온보딩 시 계정 발급 — `organization`은 `iam`이 공개한 창구만 호출:

```java
// iam 컨텍스트가 외부에 공식 제공하는 OHS (구체 클래스)
@Service
@Transactional
public class AuthOpenHostService {
    private final UserAccountRepository userAccountRepository;

    public void provisionAccount(String userAccountId, String email) {
        if (isEmailRegistered(email)) throw new IllegalArgumentException("이미 사용 중인 이메일입니다.");
        userAccountRepository.save(UserAccount.provision(userAccountId, email));
    }
}
```

**인터페이스를 둘지는 상황에 따라:**
- 이미 별도 서비스 클래스로 떼어 제공한다면 인터페이스를 또 둘 가치가 적다 → 구체 클래스를 그냥 호출.
- 기존 도메인 서비스에 외부용 기능만 얹는다면 → *노출 범위만* 인터페이스로 캡슐화해 내놓는다.

### 2) 의존 역전(DIP)을 활용한 SPI(Service Provider Interface)
OHS가 단방향(클라이언트 → 제공자)인데 **역방향 호출까지 필요**해 순환이 생기면, 허브가 SPI를 소유해 끊는다.

예) 휴가(`leave`) ↔ 결재(`approval`):
1. **정방향**: 휴가 신청 완료 → 결재 OHS `submit()` 호출. (휴가 → 결재)
2. **역방향**: 결재 확정(승인/반려/취소) → 휴가가 연차 잔액을 차감/원복. (결재 → 휴가?) ← 그대로 두면 순환.

허브(결재)가 "결재가 확정되면 이렇게 처리하라"는 **SPI 인터페이스를 소유·정의**하고, 상대(휴가)가 구현해 **빈으로 등록**한다. 허브는 구현체 정체를 모른 채 `kind`로 디스패치한다.

```java
// approval(허브)이 소유하는 확장 포인트(SPI)
public interface ApprovalDecisionSpi {
    ApprovalRequestKind kind();                 // 디스패치 키
    void onApproved(ApprovalDecisionContext ctx);   // 같은 tx — 예외 시 전체 롤백
    void onRejected(ApprovalDecisionContext ctx);
    void onCancelled(ApprovalDecisionContext ctx, boolean wasApproved);
}
```
```java
// leave 컨텍스트가 SPI를 구현·등록
@Component
public class LeaveApprovalSpi implements ApprovalDecisionSpi {
    public ApprovalRequestKind kind() { return ApprovalRequestKind.LEAVE; }
    public void onApproved(ApprovalDecisionContext ctx) {
        AnnualLeaveBalance balance = loadBalance(ctx);
        balance.deduct(...);                    // 휴가 도메인 핵심 로직
        annualLeaveBalanceRepository.save(balance);
    }
    // onRejected, onCancelled ...
}
```
```java
// approval(허브) — 구현 목록을 주입받아 kind로 디스패치
@Service
public class ApprovalService {
    private final Map<ApprovalRequestKind, ApprovalDecisionSpi> spiByKind;  // List<...> 주입 후 매핑
    public void decide(String requestId, ApprovalRequestStatus status) {
        // 상태 전이 후
        if (approved) spiByKind.get(request.kind()).onApproved(toContext(request));
    }
}
```

소스 의존성은 끝까지 **휴가 → 결재** 한 방향이고, 런타임 제어 흐름만 결재 확정 시점에 역방향으로 돈다. 덕분에 **휴직·근태** 등 결재 연동 비즈니스를 추가해도 **결재 모듈은 수정 0** — 새 컨텍스트가 SPI를 구현·등록만 하면 된다.

- 트레이드오프: 폴더 방식(A)보다 기술 복잡도↑. A가 "모든 도메인을 다 열어둔" 게 걸릴 때 꺼내는 다음 카드.

## 방법 C. 동기 인-프로세스 이벤트

참조를 *허용하되 다스리는* A·B와 달리, **참조 자체를 끊는다.** 발행 모듈은 "사실"만 알리고 손을 뗀다.

```
EmployeeService ──EmployeeOnboarded 발행──▶ 인-프로세스 이벤트 버스 ──동기──▶ 알림 핸들러 ──▶ NotificationRepository
```

- **모놀리식 안에서 동기로** 처리되는 인-프로세스 이벤트(메시지 브로커 분산 비동기 아님). 같은 스레드·트랜잭션에서 실행되므로 Saga/Outbox 같은 분산 일관성 기법 없이 단일 커밋/롤백을 유지한다.
- **실제 외부 발송(이메일·푸시)은 tx 밖**으로(이력은 tx 안, 발송은 커밋 후).
- 트레이드오프: 결합이 느슨해지지만 **제어 흐름이 은닉돼 디버깅·추적이 어렵다**. 무분별한 이벤트화보다 OHS와 비교해 신중히. 규모가 커질수록 이점이 크다.

## 얕은 CQRS — 읽기 모델

진짜 복잡도를 키우는 건 **조회**다. 화면 데이터는 늘 여러 컨텍스트 속성을 모아야 한다(휴가+결재를 한 화면에). 쓰기는 경계·규칙을 엄격히 지키되, 조회는 두 갈래로 푼다.

### 1) 화면용 데이터 → 전용 쿼리 서비스
쓰기 모델을 거치지 말고 **화면이 필요한 모양 그대로 가져오는 쿼리 서비스**를 둔다. **컨텍스트를 가로질러 조인하거나 DB 뷰를 써도 된다** — 쓰기를 안 건드리는 순수 읽기라 경계 규칙을 강요할 이유가 없다.

### 2) 비즈니스 흐름용 조회 → 내 언어의 읽기 전용 모델
다른 컨텍스트 데이터를 *검증*하거나 흐름의 입력으로 쓸 때. (동기 이벤트는 *행위를 호출*하는 것이라 조회를 대신 풀어주지 않는다.) 컨텍스트마다 OHS를 여는 대신, *테이블이 하나라고 도메인 모델까지 하나여야 하는 건 아니다*를 활용한다.

```java
// notification 컨텍스트에 격리된 읽기 전용 모델 — organization의 employees 테이블을 읽기 전용 매핑
@Table(name = "employees")
@Entity @Getter @NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Recipient {
    @Id private String id;
    private String email;
    private boolean receivingActive;   // 알림에서만 필요한 관점의 속성
    // 생성·수정 비즈니스 메서드 없음 — 오직 조회.
}
```

쓰기(생성·변경·삭제)는 `organization`의 `Employee`를 통해서만 일어나고, 알림은 `Recipient`로 자기 시각의 속성만 읽는다. DB 결합은 유지하되 코드 수준 모듈 참조 경계는 도려낸다.

> **caveat — 늘 정답은 아니다.** 가져온 데이터가 **내 컨텍스트 언어로 매끄럽게 표현되지 않으면**, 억지로 읽기 전용 모델로 감싸기보다 *그것이 외부 컨텍스트의 데이터임을 그대로 인정*하는 편이 나을 수 있다. 어디까지 내 언어로 끌어올지는 상황 판단.

## 선택 가이드 (요약)

| 상황 | 권장 |
| --- | --- |
| 컨텍스트 내부, 단일·소수 애그리거트 | 플랫(선택지 A) |
| 컨텍스트 내부, 다중 애그리거트 + 횡단 잦음 | 레이어(선택지 B) |
| 컨텍스트 2~3개, 참조 단방향 | 그냥 둔다 |
| 중규모↑, 결합 증가 | 폴더 레이어 2분할(app/module) |
| 폴더 방식이 너무 열려 부담 | OpenHostService + SPI(DIP) |
| 부수효과·알림 등 방향 끊기 | 동기 이벤트 |
| 화면 조회 | 전용 쿼리 서비스(경계 가로질러 OK) |
| 흐름 내 타 컨텍스트 조회 | 내 언어 읽기 전용 모델(안 맞으면 외부 데이터로 인정) |

## 프레임워크 노트

- **동기 이벤트 버스**: NestJS `@nestjs/event-emitter`(EventEmitter2, 기본 동기) · Spring `ApplicationEventPublisher` + `@EventListener`(또는 `@TransactionalEventListener`) · .NET MediatR in-process notifications.
- **트랜잭션 전파**(이벤트 핸들러가 같은 tx): NestJS `AsyncLocalStorage`(`nestjs-cls`) · Spring `@Transactional` 전파 · .NET `TransactionScope`/scoped `DbContext`.
- **SPI 디스패치**: Spring은 `List<Spi>` 주입 후 `kind`로 `EnumMap` 매핑 · NestJS는 멀티 프로바이더 토큰 주입 · .NET은 `IEnumerable<ISpi>` 주입.
- **읽기 전용 모델/뷰**: TypeORM `@ViewEntity` 또는 쿼리빌더 `getRawMany`→DTO · JPA 읽기 전용 매핑(`@Immutable`)·DTO projection·`@Subselect` · EF Core keyless `ToView`/`ToSqlQuery`.
- **외부 어댑터**(이메일·푸시·스토리지): 인터페이스 없이 구체 클래스로 두고 테스트에서 mock. (단 Repository는 추상 유지 — 인메모리 테스트 더블 슬롯)
