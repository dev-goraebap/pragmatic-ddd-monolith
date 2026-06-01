# distill

> 과잉 추상화·중간 매퍼·죽은 인터페이스·공허한 ViewModel을 제거한다. 카테고리: Refine.

## 사용 시점

- "이 인터페이스 구현체가 하나뿐이고 테스트도 안 쓴다"는 인터페이스가 보일 때.
- DTO → ViewModel → Entity → Response DTO 같은 다단 변환.
- 한 줄짜리 위임만 하는 Service / Mapper.

## 입력

- 추상화 의심 후보.

## 절차

1. (TBD) **테스트 더블 슬롯 검사**: 이 인터페이스가 테스트에서 실제 교체되는가? 아니라면 제거 후보.
2. (TBD) **사용처 카운트**: 구현체 1개, 호출자 1~2개라면 인라인 후보.
3. (TBD) **중간 매퍼 제거**: ViewModel을 거치지 않고 Query Service가 Response DTO를 직접 생성하는 형태로 변경.
4. (TBD) **죽은 Port 제거**: 한 번도 교체된 적 없는 abstract class/interface를 구체 클래스 직접 의존으로 변경.
5. (TBD) 변경 후 테스트와 [[audit]] 회귀 확인.

## 출력

- 줄어든 파일 수와 단순해진 호출 경로.

## 관련

- [[refactor]] - 더 넓은 교정의 한 부분.
- [[repository]] - Repository만은 추상화 유지 기준.

## 프레임워크 노트

- NestJS: 추상 식별자 제거 시 provider 등록 및 생성자 타입 동시 변경.
- Spring: `interface` 제거 시 `@Autowired` 타입 갱신, `@Primary` 충돌 확인.
- ASP.NET Core: `services.AddScoped<IFoo, Foo>()`를 `services.AddScoped<Foo>()`로 단순화.
