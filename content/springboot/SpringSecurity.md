# 🔐 Spring Security 상세 개요 및 동작 원리

Spring Security는 스프링 기반 애플리케이션의 **인증(Authentication)과 인가(Authorization)**를 처리하는 강력한 보안 프레임워크이다.  
세션 기반 인증뿐만 아니라 **JWT, OAuth2, Basic Authentication 등 다양한 인증 방식**을 지원하며, 필터 체인(Filter Chain) 기반의 구조를 갖는다.

- 관련 코드: [스프링 시큐리티](code/SpringSecurityCodes.md)

---

## 📌 Spring Security의 핵심 개념

### 1️⃣ **인증(Authentication)과 인가(Authorization)**
- **인증 (Authentication)**: 사용자가 누구인지 확인하는 과정 (로그인)
- **인가 (Authorization)**: 인증된 사용자가 특정 리소스에 접근할 수 있는 권한이 있는지 확인하는 과정

### 2️⃣ **Spring Security의 주요 컴포넌트**
| 컴포넌트 | 설명 |
|---------|-------------------------|
| `SecurityFilterChain` | 요청을 가로채고 인증/인가 필터링을 수행 |
| `AuthenticationManager` | 인증 요청을 처리하고 `Authentication` 객체를 반환 |
| `AuthenticationProvider` | 인증 정보를 검증하고 결과를 반환 |
| `UserDetailsService` | 사용자 정보를 데이터베이스에서 조회 |
| `UserDetails` | 사용자 정보를 담는 인터페이스 |
| `GrantedAuthority` | 사용자 권한 정보를 담는 인터페이스 |
| `SecurityContext` | 인증 정보를 저장하는 컨텍스트 |

---

## 🔄 Spring Security의 **동작 원리**
Spring Security는 **Filter 기반의 보안 처리 방식**을 사용한다.  
인증과 인가는 `SecurityFilterChain`을 통해 순차적으로 처리된다.

### **1️⃣ 사용자가 인증 요청을 보냄**
사용자가 로그인(인증) 요청을 하면, 요청은 **Spring Security Filter Chain**을 통과한다.

### **2️⃣ `UsernamePasswordAuthenticationFilter` 가 요청을 가로챔**
- 로그인 요청이 들어오면 **`UsernamePasswordAuthenticationFilter`** 가 요청을 가로채고 `AuthenticationManager`에 인증을 위임한다.

### **3️⃣ `AuthenticationManager`가 `AuthenticationProvider`를 호출하여 인증 수행**
- `AuthenticationManager`는 실제 인증을 `AuthenticationProvider`에게 위임한다.
- `AuthenticationProvider`는 `UserDetailsService`를 호출하여 사용자 정보를 조회한다.

### **4️⃣ `UserDetailsService`가 DB에서 사용자 정보를 로드**
- `UserDetailsService`는 데이터베이스에서 사용자의 정보를 조회하고, `UserDetails` 객체를 생성하여 반환한다.

### **5️⃣ 비밀번호 검증 후 인증 성공 시 `SecurityContext`에 저장**
- 인증이 성공하면 `SecurityContext`에 인증 정보를 저장한다.
- `SecurityContextHolder`를 통해 인증 정보를 접근할 수 있다.

### **6️⃣ 인증 후 JWT 발급 (선택)**
- JWT 기반 인증이라면, 로그인 성공 후 **JWT를 생성하여 응답**한다.
- 이후 사용자는 이 JWT를 **Authorization 헤더에 포함하여 요청**을 보낸다.

### **7️⃣ 이후 요청은 `OncePerRequestFilter`를 통해 인증 검증**
- JWT 기반 인증이라면, 매 요청마다 `OncePerRequestFilter`를 통해 토큰을 검증하고 인증 정보를 `SecurityContext`에 설정한다.

### **8️⃣ `AccessDecisionManager`가 인가(Authorization) 수행**
- 사용자가 접근하려는 리소스가 **인가된 리소스인지 확인**한다.
- `@PreAuthorize`, `@Secured` 등의 애너테이션을 사용하여 접근 권한을 제한할 수 있다.

---

## 📌 Spring Security의 **Filter Chain 흐름**
Spring Security는 여러 개의 **Filter**를 체인 방식으로 연결하여 요청을 처리한다.

### 🔽 **Spring Security Filter Chain의 주요 필터**
| 필터 | 설명 |
|------|-------------------------------|
| `SecurityContextPersistenceFilter` | 요청마다 SecurityContext를 유지 |
| `UsernamePasswordAuthenticationFilter` | 로그인 요청을 처리 |
| `BasicAuthenticationFilter` | Basic Auth 요청을 처리 |
| `BearerTokenAuthenticationFilter` | JWT 인증을 처리 (OAuth2) |
| `ExceptionTranslationFilter` | 인증/인가 예외 처리 |
| `FilterSecurityInterceptor` | 최종 인가(Authorization) 처리 |

---