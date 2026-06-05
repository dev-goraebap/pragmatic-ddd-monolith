# backend-pragmatism-design

> 모놀리식 + 단일 DB 백엔드에서 "이 코드 어디 두지, 모듈 간 결합 어떻게 풀지"를 다루는 설계 스킬.
> 정답 하나를 강요하지 않고 **상황에 맞는 선택지**를 제시합니다. DDD는 "경계를 긋는 언어"만 실용적으로 빌립니다.

## 설치

```bash
npx skills add dev-goraebap/backend-pragmatism-design
```

## 무엇을 하는가

기능 기반 구조에서 출발해 결합을 점진적으로 풉니다.

1. 관련 모듈을 **바운디드 컨텍스트**로 묶고 (하위 도메인 Core/Generic/Support),
2. 컨텍스트 간 결합을 상황에 맞는 방법으로 푼다 — **폴더 레이어 분할 / DIP 인터페이스 / 동기 인-프로세스 이벤트**,
3. 조회는 **얕은 CQRS**로 분리한다 (화면용 쿼리 서비스 / 비즈니스 흐름용 읽기 전용 모델).

규정형이 아니라 **선택지 + 가이드**이며, 폴더명·파일 배치 등 **세부는 프로젝트 컨벤션에 위임**합니다.

- [SKILL.md](./SKILL.md) — 본체(전제·흐름·선택지).
- [reference/concepts.md](./reference/concepts.md) — 빌려온 DDD 어휘.
- [reference/decoupling.md](./reference/decoupling.md) — 결합 푸는 방법과 읽기 모델 상세.

## 왜 (배경)

설계 사상과 배경은 두 글에 정리돼 있습니다.

- [경계를 긋는 언어, 도메인 주도 설계](https://goraebap.xyz/posts/language-of-boundaries/) — DDD에서 무엇을, 왜 빌렸는가.
- [백엔드 실용주의 디자인](https://goraebap.xyz/posts/backend-pragmatism-design/) — 기능 사이 결합의 원인과, 점진적으로 푸는 선택지들.

## 적용 범위

모놀리식 + 단일 DB, OOP·DI 프레임워크(NestJS, Spring Boot, ASP.NET Core 등). MSA·다중 DB·분산(결과적 일관성)은 적용 범위 밖.

## 라이선스

Apache-2.0. [LICENSE](./LICENSE) 참조.
