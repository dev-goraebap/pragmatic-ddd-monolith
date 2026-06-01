# specification

> Specification 추출 시점·Policy와의 차이·조합. 카테고리: Topic.

## 사용 시점

- 같은 판정 분기("승진 대상인가", "연체 송장인가")가 여러 곳에서 반복.
- Application Service의 가드 분기가 "조건 + 조건 + 조건"으로 길어질 때.

## 핵심 원칙

1. **무엇**: 술어(predicate). `isSatisfiedBy(target): boolean`만 다룬다.
2. **위치**: `module/<bc>/domain/` 안. 파일 명명은 팀 컨벤션. [원리는 A, 명명은 C]
3. **시그니처**: Entity/VO를 인자로 받아 boolean 반환. **Specification → Entity 단방향**.
4. **조합**: 작게 정의하고 `and` / `or` / `not`으로 조합. 클래스 인스턴스 또는 단순할 경우 static 메서드.
5. **Policy와 구분**: Specification은 whether(boolean), Policy는 how/what value.
6. **사용처**: Application Service 가드 분기, Policy 내부 조건, Repository 쿼리 조건 변환.

## 추출 절차

1. (TBD) 반복되는 판정 분기 식별.
2. (TBD) 술어 1개 단위로 잘게 분리.
3. (TBD) 별도 파일에 정의. 조합 헬퍼(`AndSpec`, `OrSpec`) 활용.
4. (TBD) 단위 테스트.
5. (TBD) 호출처 분기를 `spec.isSatisfiedBy(entity)`로 치환.

## 관련

- [[policy]] - 값 계산 전담.
- [[model]] - 도메인 전체 설계의 일부.

## 프레임워크 노트

프레임워크 무관. 순수 도메인 술어.
