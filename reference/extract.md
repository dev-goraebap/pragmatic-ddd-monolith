# extract

> 반복 로직을 `_shared/` 또는 Domain Service로 승격한다. 카테고리: Refine.

## 사용 시점

- 같은 도메인 무관 VO(`Money`, `Period`, `Email`)가 BC 두 곳에서 복붙된 시점.
- 같은 가드/인터셉터/파이프가 여러 리소스에서 반복.
- 한 BC 내에서 두 애그리게이트 협력 로직이 한 서비스에 누적.

## 절차

1. (TBD) **승격 후보 확정**: 2개 이상 위치에서 실제 중복인가 확인. 1개이면 보류.
2. (TBD) **승격지 결정**:
   - 도메인 무관 VO/에러 베이스 → `module/_shared/domain/`.
   - 비즈니스 무관 기술 인프라(UoW, RequestContext) → `shared/`.
   - 화면 공통 가드/DTO → `app/_shared/`.
   - BC 내 다중 애그리게이트 협력 → `module/<bc>/domain/` 안의 Domain Service.
3. (TBD) 이동 + 호출자 import 갱신.
4. (TBD) [[audit]] 회귀로 의존 방향 위반 없는지.

## 출력

- 줄어든 중복과 명확해진 책임 위치.

## 관련

- [[boundaries]] - 승격 결과가 새 공유 커널을 만들 때.
- [[refactor]] - 더 넓은 정렬 작업.

## 프레임워크 노트

- NestJS: `module/_shared/`를 `@Global()` 모듈로 노출하거나 명시적 import.
- Spring: 공통 패키지에 `@Configuration` 빈으로 노출.
- ASP.NET Core: 별도 어셈블리 또는 공유 프로젝트.
