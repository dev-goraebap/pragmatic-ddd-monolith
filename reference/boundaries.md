# boundaries

> BC 분할·병합·이름 정합 검토. 한 BC가 부풀거나, 두 BC가 사실상 한 몸일 때. 카테고리: Design.

## 사용 시점

- 한 BC의 파일 수가 가파르게 증가.
- 두 BC 간 호출이 한 방향으로 집중되어 사실상 하위 BC처럼 보일 때.
- 같은 단어가 BC마다 다른 의미로 쓰이거나, 반대로 다른 단어가 같은 개념을 가리킬 때.

## 입력

- 현재 BC 인벤토리 ([[document]] 결과).
- BC별 변경 빈도(가능하면 git log).
- 용어 충돌 사례.

## 절차

1. (TBD) **변경 동조성 측정**: 어떤 BC들이 함께 변경되는가.
2. (TBD) **호출 방향 분석**: `app/`이 BC들을 어떤 비율로 묶는가.
3. (TBD) **용어 충돌 해결**: 분할 vs 이름 통일 결정.
4. (TBD) **분할 전략 선택**: 추출(extract sub-BC), 분리(split into two), 병합(merge), 이름 변경(rename).
5. (TBD) 마이그레이션 계획: 디렉터리 이동·import 갱신·테스트.

## 출력

- BC 재배치 plan과 영향 범위 표.

## 관련

- [[document]] - 인벤토리 선행.
- [[extract]] - 공통 요소 승격이 필요할 때.
- [[refactor]] - 실제 이동 작업.

## 프레임워크 노트

- NestJS: `@Module` 트리 재배치, providers/exports 갱신.
- Spring: 패키지 이동, `@ComponentScan` 경로 확인.
- ASP.NET Core: 어셈블리/네임스페이스 정리, DI 등록 갱신.
