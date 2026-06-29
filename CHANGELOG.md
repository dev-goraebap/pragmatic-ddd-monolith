# Changelog

All notable changes to this skill are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.7.0] - 2026-06-29

### Changed
- **규정형 → 원칙 전용(principle-only) 실용주의 아키텍처로 전환.** "패키지를 이렇게, 클래스 이름을 이렇게"라는 강제 구조를 걷어내고, 의존 방향·CQS·계약 경계·상하류·Conformist/ACL을 **원칙과 근거** 중심으로 재작성. 본문에서 구체 패키지 경로·클래스 명명을 전부 제거하고 역할·책임·관계로만 서술.
- `reference/collaboration.md`의 협력 개념(상하류·OHS/PL·Conformist/ACL·양방향 처리·리뷰 체크리스트·용어집)을 **SKILL.md 본문으로 흡수**.

### Added
- **`reference/adr-template.md`** — 구체 결정을 팀이 ADR로 남기도록 안내하는 템플릿·작성 가이드와, 이 아키텍처에서 ADR로 남길 결정 목록(명명·컨텍스트 경계·Conformist vs ACL·ORM 정책·계약 가시성·읽기 모델 등).
- SKILL.md에 **"결정은 ADR로"** 절 신설 — 스킬은 원칙까지만, 구체는 팀이 결정해 ADR로 기록.

### Removed
- **`reference/collaboration.md` 삭제** — 개념은 SKILL 흡수, Java 코드 예시는 통째 제거.
- SKILL.md/README.md에서 **모든 Java 코드, 구체 패키지·클래스 명명, 프레임워크 노트**(Spring Data/MyBatis/QueryDSL 등) 제거. 기존 "컨벤션 위임" 절은 "결정은 ADR로"로 흡수.

## [0.6.0] - 2026-06-19

### Changed
- **스킬 이름 변경: `backend-pragmatism-design` → `pragmatic-ddd-monolith`.** "단일 배포 모놀리식"을 명확히 드러내기 위함(모듈 별도 배포가 아님). 디렉터리·GitHub 레포명도 동일하게 변경 필요.
- **하나의 구체적 표준 아키텍처로 전환** — 블로그 기반 "선택지 메뉴"에서, 단일 배포 모놀리식 + 단일 DB(Spring Boot/Java)의 구체 아키텍처로 재작성. 컨텍스트=`module/<bc>` 패키지, 내부 레이어(`domain`/`application{command,query}`/`infrastructure/adapter`/`contract`), CQS, 상류/하류, contract(OHS+PL), Conformist vs ACL.
- 톤을 **Spring/Java 구체**로(JPA 애노테이션 허용, Spring Data/MyBatis/QueryDSL 등). 블로그 글 링크 전부 제거.

### Removed
- reference `concepts.md`·`decoupling.md` → **`collaboration.md` 1개로 통합**(contract 코드·Conformist/ACL·양방향 처리·PR 체크리스트·용어집).
- 블로그(goraebap.xyz) 첨부 링크 제거.

## [0.5.0] - 2026-06-07

### Changed
- **예제를 HR 도메인(tiny-hr)으로 교체** — blog(post/category…) 대신 사원·부서·직급(organization)·인증(iam)·휴가(leave)·결재(approval)·알림(notification). 정본 글과 예제 프로젝트 [tiny-hr](https://github.com/dev-goraebap/tiny-hr)에 정렬.
- **해결책 B를 "OHS + SPI(Service Provider Interface)"로 정식화** — 허브(결재)가 SPI 인터페이스를 소유하고 상대(휴가)가 구현·빈 등록, 허브는 `kind`로 디스패치. 휴직·근태 추가 시 결재 코드 수정 0.
- **"OHS의 API는 HTTP Web API가 아니라 코드 레벨 클래스/인터페이스"** 명시.

### Added
- 읽기 전용 모델 **caveat**: 내 컨텍스트 언어로 매끄럽지 않으면 억지로 감싸지 말고 외부 데이터로 인정.
- SKILL.md에 **의사결정 흐름 요약**(무질서 → 묶기 → 티어 → 결합 방법 선택).

## [0.4.0] - 2026-06-03

### Added
- **"컨텍스트 안을 어떻게 구성할까" 축 추가** — 선택지 A(모듈 그대로 플랫) vs B(`domain`/`application`/`adapter` 레이어)와 복잡도 기반 **차등 적용(1티어/2티어)**. "핵심이라서가 아니라 복잡해서 레이어를 준다"는 기준 반영.

### Changed
- 컨텍스트 간 결합 방법 중 "DIP 인터페이스"를 **"OpenHostService(OHS) + 의존 역전"** 으로 심화 — ACL 대신 OHS로 단방향 인정, 역참조는 전략 패턴+DIP로 차단(연차 결재 예시).
- 폴더 레이어 2분할을 **app/module** 명명으로 정리하고, 상·하위가 **1:1 대칭이 아님(N:M)** 을 명시.
- 컨텍스트 정의 보강: **컨텍스트 ≠ 크기**(모듈 하나여도 컨텍스트, 잣대는 "자기만의 언어와 책임"), **경계는 양방향으로 변한다**(묶이고 쪼개짐).
- 블로그 글 개선분 반영(`backend-pragmatism-design`).

## [0.3.0] - 2026-06-03

### Changed
- **블로그 정본을 반영해 규정형 → 선택지/가이드형으로 단순화.** "app/module/shared가 유일한 정답"이라는 규정적 서술을 걷어내고, 기능 기반 구조에서 출발해 결합을 점진적으로 푸는 흐름으로 재작성: 컨텍스트로 묶기 → 결합 풀기 메뉴(폴더 레이어 / DIP / 동기 이벤트) → 얕은 CQRS 읽기 모델.
- 폴더명·파일 배치·레이어 이름 등 **세부를 프로젝트 컨벤션에 위임**(권장만, 강제 X). 컨벤션 문서의 위치·이름도 강제하지 않음.
- README를 블로그 두 글(경계를 긋는 언어 / 백엔드 실용주의 디자인) 링크 중심으로 정리.

### Removed
- reference 5개(dependency-rules, app-layer, module-layer, migration, project-convention)를 **concepts.md + decoupling.md 2개**로 통합·축소.

## [0.2.0] - 2026-06-01

### Changed
- **Realigned into a structure-first skill (FSD-analog).** Dropped the 19-command (verb) router model in favor of a self-sufficient SKILL.md whose core is the three-layer one-directional reference graph (`app → module → shared`).
- Rewrote `SKILL.md` as a complete structural decision guide: layer graph, one-line placement heuristic, capability-first `app/<resource>` slices (channels as in-slice fragments), aggregate-organized `module/<bc>`, the three `shared` scopes, variance-based DIP, shallow CQRS, folder→test mapping, and anti-patterns.
- Rewrote `README.md` with the framework-neutral philosophy (the "why": why not pure layered/hexagonal/full-CQRS, variance-based reclassification, the orchestration trade-off).

### Added
- Five consolidated, knowledge-organized references replacing the 19 stubs: `dependency-rules.md`, `app-layer.md`, `module-layer.md`, `migration.md`, `project-convention.md`. Framework notes (NestJS / Spring Boot / ASP.NET Core) preserved per file.

### Removed
- The 19 per-command reference files (teach, document, shape, craft, model, boundaries, critique, audit, refactor, distill, harden, extract, transaction, query, repository, error-model, test-strategy, policy, specification).
- Error taxonomy and test-writing tactics are no longer dictated by the skill — delegated to per-project convention (`project-convention.md` / PROJECT.md). Only the structural test mapping is retained.

## [0.1.0]

### Added
- Initial skill skeleton with router-style SKILL.md and 19 reference command stubs.
- Framework-neutral body with NestJS / Spring Boot / ASP.NET Core mapping notes per reference.
- Apache-2.0 license, npm-installable package metadata.
