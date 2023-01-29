## Jwt 코드 & Security 설정

- ### JWT 코드 , Security 설정 추가
    - ### application.yml
        - jwt 관련 설정정보 추가
        - `header: Authorization`
        - `secret:` 여기에는 임의의 문자열을 Base64인코더로 인코딩해서 넣어주기
        - `token-validity-in-seconds: 86400` 토큰 유효성 시간을 설정해준다.
    - ### build.grade

        ```
        implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
        implementation 'io.jsonwebtoken:jjwt-impl:0.11.5'
        implementation 'io.jsonwebtoken:jjwt-jackson:0.11.5'
        ```

        - jwt 개발을 위해서 의존성 추가해주기
    - ### TokenProvider
        - `private final Logger logger = LoggerFactory.*getLogger*(TokenProvider.class);`  해당 클래스에서 Log 기록을 출력하기 위해 Slf4j의 Logger 사용
        - `implements InitializingBean` → `afterPropertiesSet()` 을 오버라이딩 하여 빈이 생성되고 의존관계를 주입 받은 후에 secret값을 Base64 디코드하여 키 변수에 할당하기 위함이다.
        - `public String createToken(Authentication authentication)` Authentication 객체의 권한 정보를 이용해서 토큰을 생성하는 createToken 메소드이다.
        - `public Authentication getAuthentication(String token)` 토큰에 담겨있는 정보를 이용해 Authentication  객체를 리턴하는 메소드이다. ( 유저객체( 내가 엔티티로 생성한 유저가 아니다) , 토큰 , 권한들 )을 담은 Authentication 객체가 리턴된다.
        - `public boolean validateToken(String token)` 토큰을 파싱해보고 발생하는 예외들을 캐치하여 문제가 있으면 false를 , 정상이면 true를 반환하는 토큰의 유효성 검사하는 메소드이다.
    - ### JwtFilter
        - JWT를 위한 커스텀 필터이다.
        - `GenericFilterBean` 을 상속받은 뒤에 `public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)` 를 오버라이딩 한다. 실제 필터링 로직은 `doFilter` 메소드에 작성된다. → 토큰의 인증정보를 SecurityContext에 저장하는 역할을 수행한다.
        - `private String resolveToken(HttpServletRequest request)` HttpHeader에서 토큰 정보를 꺼내오는 메소드, `StringUtils.*hasText*(bearerToken) && bearerToken.startsWith("Bearer ")` 조건문을 통해서 이를 만족시키면 토큰 정보를 가져온다.
        - `public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException`
            - `resolveToken` 으로 토큰을 받아와서 유효성 검증을 하고 정상토큰이라면 TokenProvider의 Authentication을 받아오는 메소드를 이용해서 SecurityContext에 저장한다.
    - ### jwtSecurityConfig
        - TokenProvider , JwtFilter 를  SecurityConfig에 등록할 때 , 사용할 JwtSecurityConfig이다.
        - `SecurityConfigurerAdapter<DefaultSecurityFilterChain,HttpSecurity>` 를 상속받고 JwtFilter를 통해서 TokenProvider Security 로직에 필터를 등록한다.
    - ### JwtAuthenticationEntryPoint
        - 유효한 자격증명을 제공하지 않고 접근하려 할 때 401 Unauthorized 에러를 리턴하는 클래스
        - `AuthenticationEntryPoint` 인터페이스를 implements한다

            ```java
            
            @Override
            public void commence(HttpServletRequest request,
                                 HttpServletResponse response,
                                 AuthenticationException authException) throws IOException, ServletException {
                response.sendError(HttpServletResponse.SC_UNAUTHORIZED);
            }
            ```

    - ### JwtAccessDeniedHandler
        - 필요한 권한이 존재하지 않는 경우에 403 Forbidden 에러를 리턴하는 클래스
        - `AccessDeniedHandler` 인터페이스를 implements한다.
    - ### JwtProvider , JwtFilter , JwtSecurityConfig , JwtAuthenticationEntryPoint , JwtAccessDeniedHandler 들을 SecurityConfig에 추가한다. → SecurityConfig
        - `@EnableGlobalMethodSecurity(prePostEnabled = true)` 는 `@preAuthorize` 애노테이션을 메소드 단위로 작성하기 위해서 사용한다.
        - JwtProvider , JwtAuthenticationEntryPoint , JwtAccessDeniedHandler 는 생성자로 주입받는다.
        - `public PasswordEncoder passwordEncoder()`  패스워드 인코더로는 `BCryptPasswordEncoder()` 를 사용한다.
        - `.csrf().disable()` 토큰을 이용하기 때문에 csrf 설정은 꺼준다. → csrf 공격과 관련된 내용은 양이 많아서 다른 곳에서 정리하기로..
        - 401 에러와 403 에러에 대한 처리는 주입받았던 클래스를 넣어준다.

            ```java
            .exceptionHandling()
            .authenticationEntryPoint(jwtAuthenticationEntryPoint)
            .accessDeniedHandler(jwtAccessDeniedHandler)
            ```

        - x-frame-option 관련 설정 ( x-frame-option click jacking 공격 막기 관련 내용은 다른 곳에서.. ) 해당 페이지와 동일한 오리진의 프레임만 표시할 수 있다는 내용의 설정내용이다.

            ```java
            .and()
            .headers()
            .frameOptions()
            .sameOrigin()
            ```

        - 세션을 사용하지 않기 때문에 세션 관련 설정은 StateLess로 변경해준다. ( 아마도 비영속 상태를 말하는 것 일 듯)

            ```java
            .and()
            .sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            ```

        - 그 밖에 회원 가입 , 로그인 등의 api는 토큰이 없는 상태에서 요청이 발생하기 때문에 요청을 허가한다.

            ```java
            .authorizeRequests()
            .antMatchers("/api/hello").permitAll()
            .antMatchers("/api/authenticate").permitAll()
            .antMatchers("/api/signup").permitAll()
            .anyRequest().authenticated()
            ```

        - 마지막으로 customFilter로 만들었던 JwtFilter를 addFilterBefore로 등록했던 JwtSecurityConfig를 등록해주면 끝이다.

            ```java
            .and()
            .apply(new JwtSecurityConfig(tokenProvider));
            ```