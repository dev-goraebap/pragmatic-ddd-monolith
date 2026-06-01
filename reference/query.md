# query

> 얕은 CQRS. 화면 조회 전용 Query Service 설계. 카테고리: Topic.

## 사용 시점

- 화면이 여러 BC의 데이터를 조합해 보여줘야 할 때.
- 응답 DTO가 도메인 Entity와 모양이 다를 때.
- N+1이 의심될 때.

## 핵심 원칙

1. **위치**: `app/<resource>/` 안의 Query Service. `module/` 안에 두지 않는다. 파일 명명은 팀 컨벤션. [원리는 A, 명명은 C]
2. **Domain / Repository를 거치지 않고 DB를 직접 쿼리**한다. 크로스 BC JOIN 허용.
3. **Response DTO를 직접 생성**한다. 중간 ViewModel 금지.
4. **CUD에서 호출 금지**. 화면 스펙 변경이 비즈니스를 깨뜨린다.
5. **읽기 전용 트랜잭션** 명시.

## 절차

1. (TBD) 화면 스펙(필드 목록·정렬·페이징) 확정.
2. (TBD) JOIN 또는 다단 쿼리 설계.
3. (TBD) N+1 검사 (배치 로딩 또는 단일 쿼리).
4. (TBD) Response DTO 매핑 직접 작성.
5. (TBD) 통합 테스트(선택)로 SQL 정합성 검증.

## 관련

- [[repository]] - 쓰기 측의 대비.
- [[transaction]] - 읽기 전용 트랜잭션 모드.

## 프레임워크 노트

- NestJS / TS: Drizzle / Kysely 같은 쿼리 빌더, 또는 raw SQL 권장. ORM 풀 그래프 로딩 회피.
- Spring: Spring Data Projections, JPQL DTO projection 또는 jOOQ.
- ASP.NET Core: Dapper 또는 EF Core projection (`Select(x => new ResponseDto { ... })`).
