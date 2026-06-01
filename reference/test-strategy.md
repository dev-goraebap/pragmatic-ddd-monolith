# test-strategy

> 3-tier 테스트 배치(Solitary / Sociable / Integration)와 테스트 더블 위치. 카테고리: Topic.

## 사용 시점

- 신규 유스케이스/도메인 추가.
- 기존 테스트가 부풀어 누가 누구를 검증하는지 불명확할 때.
- CI 시간이 문제가 될 때.

## 핵심 원칙

| 대상 | 종류 | 특징 | 필수 | 등급 |
| --- | --- | --- | --- | --- |
| 도메인 모델 (`module/<bc>/domain/`) | Solitary Unit | 의존성 0. 도메인 invariant·Policy·Spec | 필수 | A |
| 유스케이스 (`app/<resource>/`) | Sociable Use Case | 진짜 Entity + InMemory Repository + Mock 어댑터 + Noop UoW | 필수 | A |
| Repository 구현·Query Service | Integration | 실제 DB | 선택 (생략 권장) | A |

테스트 파일을 대상 코드와 같은 폴더에 co-located 배치할지 별도 `__tests__/` 하위에 둘지는 팀 컨벤션 (PROJECT.md).

## 테스트 더블 위치

원리: 테스트 더블은 운영 빌드에 포함되지 않는 위치에 격리한다. [A]

- **인메모리 Repository (Stub)**: 상태 보유 검증이 필요할 때.
- **Fake 외부 어댑터**: Mock으로 행위 검증이 어려울 때만.

격리 기법(별도 디렉터리 `__tests__/`, suffix `*.spec.*`, 빌드 설정에서 exclude)은 빌드 도구와 팀 컨벤션에 따라 결정 (PROJECT.md). [C]

## 절차

1. (TBD) 도메인 추가 시: Solitary 테스트 동시 작성.
2. (TBD) 유스케이스 추가 시: Sociable 테스트 동시 작성.
3. (TBD) Stub은 상태 검증이 필요할 때만. 그 외는 Mock 라이브러리.
4. (TBD) Given/When/Then 한글 주석으로 시나리오 가독성 확보.

## 관련

- [[model]] - 도메인 테스트와 짝.
- [[craft]] - 유스케이스 테스트와 짝.
- [[repository]] - InMemory Stub 배치.

## 프레임워크 노트

### DI 토큰별 교체 주입 idiom

- **NestJS / TS**: Jest + `Test.createTestingModule({ providers: [...] })`로 토큰 단위 교체.
- **Spring**: JUnit 5 + Mockito. `@MockBean`은 Spring 컨텍스트가 뜨는 통합 테스트에서만. Solitary는 plain Mockito.
- **ASP.NET Core**: xUnit + Moq/NSubstitute. Sociable에서는 서비스 컬렉션 수동 구성, `WebApplicationFactory`는 통합 테스트에만.

### 빌드 격리 idiom

- **TS**: `tsconfig.build.json`에서 `**/__tests__/**` exclude.
- **Java/Kotlin**: Maven/Gradle의 `src/test/` 분리는 기본값.
- **C#**: 별도 테스트 프로젝트(`*.Tests.csproj`).
