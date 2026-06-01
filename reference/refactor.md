# refactor

> [[audit]] / [[critique]]에서 잡힌 설계 위반을 코드로 교정한다. 카테고리: Refine.

## 사용 시점

- 위반 목록이 정해졌고 실제 수정에 진입할 때.
- "이 BC를 통째로 본 가이드 형태로 정렬하자"는 결심이 섰을 때.

## 입력

- 위반 목록 또는 리팩토링 목표.
- 변경 가능한 테스트 커버리지.

## 절차

1. (TBD) **안전망 확보**: 대상 영역의 테스트가 그린이고 충분한지 확인. 부족하면 먼저 보강.
2. (TBD) **변경 단위 쪼개기**: 한 PR당 1~2개 위반 룰만.
3. (TBD) **레이어 이동**: 잘못된 위치의 클래스/파일을 옳은 디렉터리로.
4. (TBD) **책임 재분배**: Controller의 분기를 Service로, Service의 결정을 Domain으로.
5. (TBD) **죽은 추상 제거**: 자세한 절차는 [[distill]].
6. (TBD) **테스트 재배치**: Solitary는 도메인 옆, Sociable은 app 옆.
7. (TBD) [[audit]] 재실행으로 회귀 없음 확인.

## 출력

- 그린 테스트와 줄어든 위반 카운트.

## 관련

- [[audit]] / [[critique]] - 진단 선행.
- [[distill]] - 추상화 축소 전문.
- [[extract]] - 공통 요소 승격.
- [[boundaries]] - BC 자체 재배치가 필요할 때.

## 프레임워크 노트

- NestJS: `@Module` providers/exports 이동 시 import 사이드이펙트 확인.
- Spring: 패키지 이동 + `@ComponentScan` 경로 갱신.
- ASP.NET Core: 어셈블리 분리 시 DI 등록 옮기는 것을 빠뜨리지 않기.
