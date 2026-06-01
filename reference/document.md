# document

> 기존 코드베이스를 스캔해 ARCHITECTURE.md(현재 구조 스냅샷)를 역생성한다. 카테고리: Setup.

## 사용 시점

- 기존 코드가 있고 신규 합류자가 구조를 빠르게 파악해야 할 때.
- 리팩토링 전후 비교용 스냅샷이 필요할 때.
- 컨벤션 위반/누락이 어디에 분포하는지 한눈에 보고 싶을 때.

## 입력

- 현재 코드베이스 트리.
- (선택) PROJECT.md - 기대 BC 목록과 대조.

## 절차

1. (TBD) `app/`, `module/`, `shared/` 트리를 스캔해 레이어별 디렉터리 목록 수집.
2. (TBD) BC별 Aggregate Root 후보 추출. 코드 표지는 프레임워크/ORM마다 다르므로 PROJECT.md의 표지 정의(파일 suffix, ORM 어노테이션, 클래스 명명 규약 등)를 따른다.
3. (TBD) Repository / 외부 어댑터 / Query Service 목록 정리.
4. (TBD) 단방향 의존성 위반 위치를 별도 섹션에 표시.
5. (TBD) ARCHITECTURE.md 생성.

## 출력

- `ARCHITECTURE.md` 스냅샷. 예상 섹션:
  - 레이어 트리
  - BC별 도메인 요소 인벤토리
  - 의존성 흐름 다이어그램
  - 위반·이상 신호 목록

## 관련

- [[teach]] - 기대값(PROJECT.md)이 먼저 있어야 위반 신호를 의미있게 분류.
- [[audit]] - 발견된 위반을 깊이 진단.
- [[boundaries]] - 인벤토리 결과로 BC 재분할 검토.

## 프레임워크 노트

- NestJS: `@Module` providers/exports 그래프, `@Injectable` 클래스 분류.
- Spring: `@Component` / `@Service` / `@Repository` 어노테이션 분류, `@Configuration` 그래프.
- ASP.NET Core: `Program.cs`의 `services.AddXxx` 등록 그래프, `Microsoft.Extensions.DependencyInjection` 식별자.
