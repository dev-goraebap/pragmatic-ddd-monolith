# policy

> Policy 추출 시점·Entity와의 관계·Specification과의 차이. 카테고리: Topic.

## 사용 시점

- "이 값을 어떻게 계산할지"가 변형(strategy)을 가질 때 (할인 정책, 휴가 산정 정책).
- Entity 메서드나 Application Service에 결정 규칙이 누적되어 무거워질 때.

## 핵심 원칙

1. **무엇**: 상태 없는 도메인 규칙. "어떻게 결정/계산할 것인가"를 캡슐화.
2. **위치**: `module/<bc>/domain/` 안. 파일 명명은 팀 컨벤션. [원리는 A, 명명은 C]
3. **시그니처**: Entity/VO를 인자로 받아 값을 반환. **Policy → Entity 단방향**. Entity는 Policy를 import하지 않는다.
4. **호출 주체**: 대개 Application Service. 결과를 Entity 메서드 인자로 다시 넘겨 상태 전이.
5. **Specification과 구분**: Policy는 how/what value, Specification은 whether(boolean).

## 추출 절차

1. (TBD) Entity 메서드/Service에 분기와 계산이 누적된 영역 식별.
2. (TBD) 입력 타입과 출력 타입 명확화 (Entity/VO 입력, 값 출력).
3. (TBD) 별도 파일에 static 메서드로 추출. 변형이 명확해지면 인터페이스+구현체로 객체화.
4. (TBD) Policy 단위 테스트(의존성 0).
5. (TBD) 호출처를 Application Service로 옮기고 Entity는 결과만 받음.

## 관련

- [[specification]] - 판정 전담.
- [[model]] - 도메인 전체 설계의 일부.

## 프레임워크 노트

프레임워크 무관. 순수 도메인 계산.
