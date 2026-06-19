# collaboration — 컨텍스트 간 협업 (contract · Conformist · ACL)

> 상류가 `contract`로 공개하고, 하류가 Conformist 또는 ACL로 수용하는 실제 코드와 판단 기준. 그리고 양방향 처리·PR 체크리스트·용어집.
> SKILL.md의 4~6장 심화. 예시 도메인: 공사(`construction`, 상류) / 위험성평가(`riskassessment`, 하류).

![A(하류)·B(상류) 협업 구조 — OHS/PL, ACL, Conformist](diagram.png)

> A 컨텍스트(하류): Presentation → application/{command,query} → domain(Repository 인터페이스) → infrastructure. 상류 B가 견고하면 OHS에 직접 접근(Conformist), 격리가 필요하면 infrastructure의 ACL이 상류 OHS를 호출. B 컨텍스트(상류): `contract`로 OHS/PL만 공개.

---

## 1. 상류: `contract` (OHS + PL)

상류는 `module/<bc>/contract` 하나로만 공개한다. **OHS** = 공개 서비스 인터페이스, **PL** = 계약에 쓰는 공개 타입(도메인 엔티티 직접 노출 금지).

```java
// module/construction/contract/WorkType.java — PL (안정적 공개 데이터 모델)
public record WorkType(String id, String name) {}

// module/construction/contract/ConstructionContract.java — OHS (공개 서비스 인터페이스)
public interface ConstructionContract {
    boolean exists(String constructionId);
    List<WorkType> findWorkTypes(String constructionId);
}
```

### 구현 방식 (상류 `application`에서)

- **방법 A — 직관적**: 기존 애플리케이션 서비스가 `implements ConstructionContract`. 내부/외부 호출을 한 서비스로 통합.
- **방법 B — 역할 분리**: 외부 노출용 브릿지 서비스(예: `ConstructionContractService`)를 `application` 아래 따로 두고 위임. 내부 유스케이스와 외부 스펙의 결합을 낮추고 싶을 때.

어느 쪽이든 **하류엔 구현체가 아니라 `contract` 인터페이스만** 드러나야 한다.

---

## 2. 하류 선택지 A — Conformist (순응)

상류 `contract`·PL을 하류 `application`에서 **직접** 가져다 쓴다. 변환 계층 없음.

```java
// 하류 application — 상류 계약을 그대로 수용
import com.example.app.module.construction.contract.ConstructionContract;

private final ConstructionContract constructionContract; // 그대로 사용
```

**언제**: 상류 스펙·모델이 보편적이고 변경 가능성이 낮아, 그대로 받아도 하류 도메인 순수성에 무리가 없을 때 (예: 전역 공통 '결재', '파일 첨부').

---

## 3. 하류 선택지 B — ACL (부패 방지 계층)

하류가 **자기 도메인 언어로 포트(인터페이스)** 를 선언하고, `infrastructure/adapter`가 상류 `contract`를 호출·번역한다. 하류 `application`은 상류의 존재를 모른다.

```java
// 1) 하류 application — 내가 필요한 만큼 내 언어로 선언 (Port)
//    module/riskassessment/application/command/provider/ConsProvider.java
public interface ConsProvider {
    boolean exists(String id);
    List<WorkType> findWorkTypes(String constructionId);
    record WorkType(String id, String name) {}   // 하류 내부 전용 타입
}

// 2) 하류 infrastructure — 포트 구현, 여기서만 상류와 대화 (Adapter)
//    module/riskassessment/infrastructure/adapter/ConsAdapter.java
@Component
public class ConsAdapter implements ConsProvider {
    private final ConstructionContract constructionContract; // 상류 contract 주입

    @Override
    public boolean exists(String id) {
        // 필요하면 여기서 상류 응답을 하류 도메인 형식으로 다듬고 검증
        return constructionContract.exists(id);
    }
}
```

```
riskassessment (하류)                         construction (상류)
  application(ConsProvider 포트) ┄의존역전┄ infrastructure/adapter(ConsAdapter)
                                              └──── 여기서만 대화·번역 ────▶ contract
```

**언제**:
- 상류가 설계 진행 중이거나 스펙이 자주 바뀔 우려 (어댑터를 Mock으로 대체해 병행 개발).
- 상류 도메인 성격이 특이해 하류 핵심 로직에 원치 않는 용어·모델을 들이고 싶지 않을 때.
- 외부 연동 때문에 **하류 application 단위 테스트가 깨지는 걸 방어**하고 싶을 때(가짜 포트 주입).

### 비교

| 항목 | Conformist | ACL |
| --- | --- | --- |
| 상류 의존 침투 | `application`까지 직접 | `infrastructure/adapter`로 격리 |
| 데이터 타입 | 상류 PL 그대로 | 하류 고유 언어 타입 |
| 적합 상황 | 표준성·안정성 높은 상류 | 가변 스펙 / 하류 독립 보호 |
| 초기 비용 | 낮음 (추가 코드 불필요) | 중간 (포트+어댑터+매핑) |

---

## 4. 양방향 참조가 생기면

순환(양방향 의존)은 코드로 우회하지 말고 **경계 재논의 신호**로 받는다.

- **방향 정리**: 흐름의 주도권을 쥔 쪽을 상류로 정해 단방향으로. 역방향 조회가 필요하면 *받는 쪽*이 contract를 제공하거나 이벤트를 발행.
- **경계 재정의**: 아무리 봐도 '운명 공동체(Partnership)'면 두 컨텍스트를 **하나로 병합**.

---

## 5. PR 체크리스트

- [ ] **의존성 침투**: 하류가 상류의 내부 패키지(`domain`/`application`/repository)를 import하지 않고 오직 `contract`만 보는가.
- [ ] **양방향 의존**: 두 컨텍스트가 서로 직접 참조하지 않는가. (보이면 4장 재검토)
- [ ] **정보 은닉**: 상류가 외부에 안 흘려도 되는 내부 구현(application 클래스 등)을 노출하지 않는가.
- [ ] **도메인 엔티티 은닉**: PL이 JPA 도메인 엔티티를 그대로 반환·포함하지 않는가. (공개용 DTO/record 경유)
- [ ] **CQS**: `application/command`와 `application/query`가 엉키지 않는가. (command가 query 서비스 호출 금지, query가 도메인 Repository 직접 참조 금지)
- [ ] **도메인 독립성**: `domain`이 외부 구체 기술(웹 어노테이션·타 컨텍스트 포트)을 참조하지 않는가. (JPA 애노테이션은 허용)
- [ ] **자가 검증·테스트**: 핵심 도메인이 DB·스프링 없이 단위 테스트되고, 불변식을 스스로 검증하는가.
- [ ] **ACL 격리**: ACL이면 상류 호출 코드가 `infrastructure/adapter`에만 있고 `application`엔 포트만 보이는가.
- [ ] **공통 영역 순수성**: `common`/`util`에 특정 도메인 이름이 새어 들어가지 않았는가.

---

## 6. 용어집

| 용어 | 약속 |
| --- | --- |
| **바운디드 컨텍스트 (BC)** | 비즈니스 도메인의 경계. 코드에서는 `module/<bc>` 루트 패키지 하나. |
| **레이어 (Layer)** | 한 컨텍스트 내부의 기술 관심사(표현·응용·도메인·인프라) 수평 분리. |
| **CQS** | 상태 변경(Command)과 조회(Query)를 별개 채널로 분리. |
| **상류 (Upstream)** | 데이터·서비스를 공급하는 쪽. 계약 주도권 보유. |
| **하류 (Downstream)** | 상류 스펙을 수용해 비즈니스를 수행하는 의존 쪽. |
| **OHS (Open Host Service)** | 상류가 외부에 공개하는 표준 인터페이스(`XxxContract`). |
| **PL (Published Language)** | 계약에서 합의한 공개 데이터 교환 포맷(도메인 엔티티 직접 노출 금지). |
| **Conformist** | 상류 모델·인터페이스를 하류가 번역 없이 그대로 수용. |
| **ACL (Anti-Corruption Layer)** | 외부 스펙 변경·결합으로부터 하류 도메인을 지키는 번역 격리 계층. |
| **포트/어댑터** | 안쪽에 명세(포트), 바깥에 기술 구현(어댑터)을 주입해 의존 역전·테스트 격리. |

## 프레임워크 노트

- **Repository 구현**: Spring Data JPA / MyBatis / QueryDSL 등 자유. 도메인엔 인터페이스만, 구현은 `infrastructure/adapter`.
- **ReadQuery(조회)**: Spring Data JPA `@Query` 프로젝션, QueryDSL, MyBatis 매퍼 등. 화면 DTO로 직접 프로젝션.
- **contract 공개 범위 제어**: 패키지 분리 + 리뷰로. 더 강하게는 Java 모듈(JPMS)·ArchUnit·Spring Modulith로 `contract` 외 참조를 검증 가능.
- **ACL 포트 테스트**: 어댑터를 Mock/Fake로 대체해 하류 `application` 단위 테스트를 외부와 격리.
