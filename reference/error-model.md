# error-model

> 도메인 에러 분류 체계와 HTTP 매핑. 카테고리: Topic.

## 사용 시점

- 신규 BC의 에러 클래스 정의.
- 전역 예외 처리 지점 설계.
- API 에러 응답 스펙 정합성 확보.

## 핵심 원칙

1. **도메인 에러는 의미별 5종으로 분류한다.** 400 BadRequest / 401 Unauthorized / 403 Forbidden / 404 NotFound / 409 Conflict. [A]
2. **도메인 에러 클래스가 HTTP 상태로 자동 매핑되도록 전역 예외 처리 지점에서 연결한다.** Application Service나 Controller가 매번 HTTP 상태를 결정하지 않게 한다. 매핑 기법(필터 / advice / middleware / interceptor)은 프레임워크마다 다르다. [원리는 A, 기법은 B]
3. **메시지는 사용자 표시용이다.** 디버그 정보(스택 trace, 내부 식별자)는 별도 필드 또는 로그로 분리한다. [A]
4. **HTTP 프레임워크의 예외 타입(`HttpException`, `ResponseStatusException`, `BadRequestException` 등)을 도메인에서 throw하지 않는다.** 도메인은 HTTP를 모른다. [A]

## 분류 가이드

| HTTP | 베이스 | 의미 |
| --- | --- | --- |
| 400 | BadRequest | 입력 자체가 잘못됨 (포맷, 범위) |
| 401 | Unauthorized | 인증 누락 또는 실패 |
| 403 | Forbidden | 인증은 되었으나 권한 없음 |
| 404 | NotFound | 대상 리소스 부재 |
| 409 | Conflict | 비즈니스 불변식 위배, 상태 충돌, 동시성 충돌 |

## 팀이 별도로 결정할 항목 (PROJECT.md)

- **에러 코드 명명 규칙** (`<BC>_<CODE>` 대문자 스네이크 / `<bc>.<code>` 도트 / 그냥 SCREAMING_SNAKE 등).
- **wire 응답 포맷** (`{ code, message }` / `{ error: { code, message, details } }` / RFC 9457 Problem Details 등).
- **메시지 다국어 정책** (서버에서 i18n / 코드만 보내고 클라가 번역 / Accept-Language 기반).
- **도메인 에러 베이스 클래스의 물리적 위치 및 파일 명명**.

## 절차

1. (TBD) 발생 가능한 도메인 에러를 시나리오별 나열.
2. (TBD) 5종 베이스에 매핑.
3. (TBD) 팀 코드 명명 규약(PROJECT.md) 따라 코드 부여.
4. (TBD) 사용자 표시 메시지 작성.
5. (TBD) 도메인 단위 테스트에서 throw 검증.

## 관련

- [[model]] - 도메인 에러는 모델 설계의 일부.
- [[harden]] - Conflict / 동시성 충돌 자리 메우기.

## 프레임워크 노트

### 전역 매핑 기법

- **NestJS**: `ExceptionFilter`에서 `instanceof DomainError` 분기하여 `HttpException` 변환. `@Catch(DomainError)` + global filter 등록.
- **Spring Boot**: `@ControllerAdvice` + `@ExceptionHandler(DomainError.class)`. `ResponseEntity` 반환 또는 `ProblemDetail` 사용.
- **ASP.NET Core**: `IExceptionFilter` 또는 `UseExceptionHandler` 미들웨어. .NET 8+ `IExceptionHandler` 인터페이스.

### 도메인 에러 베이스 구현 예

- **NestJS / TS**: 추상 클래스 `DomainError`와 5종 subclass (`BadRequestError`, ...). HTTP 매핑은 필터가 담당.
- **Spring / Kotlin**: sealed class `DomainError`와 5종 subclass. `@ControllerAdvice`가 각 타입을 ResponseEntity로 변환.
- **C# / .NET**: abstract class `DomainException` 또는 record-based. 미들웨어가 ProblemDetails로 직렬화.

### HTTP 직접 throw 회피

각 프레임워크는 표준 HTTP 예외 타입을 제공하지만(`HttpException`, `ResponseStatusException`, `BadHttpRequestException` 등) 도메인에서는 사용하지 않는다. Application Service에서 외부 입력 검증 실패로 즉시 400을 던질 일이 생기면, 그것도 도메인 에러로 분류하거나 DTO 검증 단계로 끌어올린다.
