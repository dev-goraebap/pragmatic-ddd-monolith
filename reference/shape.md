# shape

> 코드 작성 전 유스케이스 설계. 어느 BC·repository·service를 건드릴지, 트랜잭션 경계는 어디인지 미리 확정한다. 카테고리: Design.

## 사용 시점

- 신규 기능 시작 직전.
- 여러 BC를 묶는 오케스트레이션이 필요한 유스케이스.
- "이걸 어디에 두지?" 가 명확하지 않을 때.

## 입력

- 기능 한 줄 설명 (유저 스토리 또는 API 스펙).
- PROJECT.md의 BC 목록.

## 절차

1. (TBD) **Surface 확정**: HTTP 경로, 컨트롤러 위치(`app/<resource>/`).
2. (TBD) **Touch list**: 건드리는 BC, 호출할 Repository / Domain Service 나열.
3. (TBD) **트랜잭션 경계**: 한 UoW에 묶을 범위 결정.
4. (TBD) **검증 책임**: 구문은 DTO, 비즈니스는 Domain/Policy/Spec 분배.
5. (TBD) **읽기 vs 쓰기 분리**: Query Service 후보 여부.
6. (TBD) **에러 분류**: 발생 가능한 도메인 에러와 HTTP 매핑 미리 결정.
7. (TBD) 사용자에게 brief을 보여주고 합의 후 [[craft]]로 진입.

## 출력

- 유스케이스 brief (touch surface, 트랜잭션 경계, 검증 책임, 에러 목록).

## 관련

- [[craft]] - shape 결과를 받아 end-to-end 구현.
- [[transaction]] - 트랜잭션 경계 결정 상세.
- [[query]] - 읽기 분리 여부 판정.
- [[error-model]] - 에러 분류 체계.

## 프레임워크 노트

- NestJS: Controller / Service / Module 트리오로 매핑.
- Spring: `@RestController` / `@Service` / `@Configuration`로 매핑.
- ASP.NET Core: Minimal API or Controller / Service로 매핑.
