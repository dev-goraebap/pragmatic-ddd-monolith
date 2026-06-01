# teach

> PROJECT.md(BC 목록·팀 컨벤션·도메인 용어집)를 인터뷰로 세팅한다. 카테고리: Setup.

## 사용 시점

- 프로젝트에 PROJECT.md가 없거나 placeholder만 있을 때.
- 신규 합류자가 BC 경계와 팀 컨벤션을 한눈에 보고 싶을 때.
- 도메인 용어 충돌(같은 단어 다른 의미)이 반복될 때.

## 입력

- 현재 디렉터리 구조 (`module/<bc>/` 목록).
- 사용자와의 라운드 인터뷰 (BC별 책임, 핵심 용어, 외부 시스템 연동 목록).

## 절차

1. (TBD) 현 `module/` 하위 BC 자동 스캔.
2. (TBD) BC별로 "이 BC는 무엇을 책임지고 무엇을 책임지지 않는가" 1줄 질문.
3. (TBD) 용어집(Ubiquitous Language) 시드 수집.
4. (TBD) 팀 컨벤션 확인(시간/ID 직접 호출 여부, audit 컬럼 처리, 검증 라이브러리 선택).
5. (TBD) PROJECT.md 작성·저장. 이후 모든 명령이 이 파일을 읽는다.

## 출력

- `PROJECT.md` (프로젝트 루트 또는 `docs/`).

## 관련

- [[document]] - 코드에서 ARCHITECTURE.md를 역생성하는 자매 명령.
- [[boundaries]] - PROJECT.md의 BC 목록을 검토하고 재분할할 때.

## 프레임워크 노트

프레임워크 무관. 본 명령은 메타데이터(파일)만 생성한다.
