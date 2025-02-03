# CSRF 공격
- CSRF는 사이트 간 요청 위조라는 의미로, 공격자가 희생자의 권한을 도용하여 특정 웹 사이트의 기능을 실행하게 할 수 있다.
- CSRF 공격을 하기 위해서는 희생자가 위조 요청을 보낼 사이트에 로그인 되어 있는 상태에서 공격자가 만든 피싱 사이트에 접속해야 한다.
- 특정 사이트에 자동 로그인이 되어 있다면 공격을 당하기 쉽다.

---

## 백엔드에서의 CSRF 공격 대응 수단.
### 1️⃣ Referer 검증
#### 요청이 들어올 때 request의 header에 담겨있는 referrer 값을 확인하여 같은 도메인에서 보낸 요청인지 검증하여 차단.
#### 구현: 별도 필터 구현 후 SecurityConfig에 추가

### 2️⃣ CSRF 토큰 사용
#### CSRF 토큰을 헤더에 담아 클라이언트에 전송 후 모든 요청마다 그 값을 포함하여 전송.
#### 백엔드에서 세선에 저장된 값과 요청으로 전송된 값이 일치하는지 확인.
#### 구현: CSRF 토큰을 헤더에 추가하는 로직 추가 + SecurityConfig에 아래 코드 추가

```
http.csrf(csrf -> csrf
                .ignoringRequestMatchers("/login", "/csrf-token", "/register/*")
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
        );
```

#### 1. 토큰이 없는 상태에서 POST 요청을 보내면 토큰이 발급되지 않음.
#### 2. Postman에서 아래 요청을 보내면 쿠키에 토큰이 발급되나 어찌된 영문인지 웹페이지에서 테스트 했을 때는 발급되지 않았음. 일단 아래 코드 추가해서 해결.
```
// 로그인 전 해당 경로로 요청을 보내 토큰 획득
    @GetMapping("/csrf-token")
    public String getCsrfToken(HttpServletRequest request) {
        CsrfToken csrfToken = (CsrfToken) request.getAttribute("_csrf");
        return csrfToken != null ? csrfToken.getToken() : "CSRF token not found";
    }
```