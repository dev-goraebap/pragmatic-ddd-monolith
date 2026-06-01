# craft

> shape 거쳐 end-to-end 구현. Controller, Application Service, DTO, Repository 메서드 추가, 도메인 메서드, 테스트까지. 카테고리: Design.

## 사용 시점

- shape에서 합의된 brief가 있고 실제 구현으로 진입할 때.
- 신규 유스케이스 전체 코드를 빠짐없이 깔고 싶을 때.

## 입력

- [[shape]] 결과 brief.
- PROJECT.md.

## 절차

1. (TBD) shape brief 재확인.
2. (TBD) `app/<resource>/`에 controller, service, query-service(필요 시), DTO 스캐폴드.
3. (TBD) `module/<bc>/domain/`에 부족한 Entity 메서드 / Policy / Spec / Error 추가.
4. (TBD) `module/<bc>/infrastructure/`에 Repository 메서드 추가.
5. (TBD) UoW로 트랜잭션 경계 명시.
6. (TBD) Sociable Use Case Test (`app/.../__tests__/`) 작성.
7. (TBD) 추가된 도메인 메서드의 Solitary Test 작성.

## 출력

- 동작하는 신규 유스케이스 풀세트.
- 테스트 그린.

## 관련

- [[shape]] - 선행 단계.
- [[model]] - 도메인 메서드 추가 시 참고.
- [[repository]] - Repository 메서드 컨벤션.
- [[test-strategy]] - 테스트 배치.

## 프레임워크 노트

- NestJS: `@Module` providers/exports 갱신, `@Injectable` 등록 누락 주의.
- Spring: `@Configuration` 빈 등록, `@Transactional` 위치.
- ASP.NET Core: `Program.cs` 등록, `IUnitOfWork` 스코프.
