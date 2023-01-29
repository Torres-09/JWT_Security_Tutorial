## 사용 예제

- ### DTO , Repository , 로그인
    - ### LoginDto
        - `@NotNull` 애노테이션을 필드에 추가해서 validation 적용
    - ### SignUpDto
        - 회원가입 시 필요한 정보
    - ### TokenDto
        - token String 보유
    - ### UserRepository
        - `JpaRepository<User, Long>` 를 extends(상속) 하여 기본적인 findAll , save 등의 메소드를 기본적으로 사용 가능하다.
        - `@EntityGraph(attributePaths = "authorities")` → 쿼리가 수행이 될 때, Lazy 조회(지연 로딩)가 아니고 Eager조회(즉시 로딩)로 authorities 정보를 같이 가져온다.
        - `Optional<User> findOneWithAuthoritiesByUserName(String userName)` userName을 기준으로 유저 정보를 가져올 때, 권한 정보도 같이 가져온다.
    - ### CustomUserDetailsService
        - `UserDetailsService` 를 implements 한다. → `loadUserByUsername` 를 오버라이딩 해야 한다.
        - `loadUserByUsername` 는 로그인 시에 DB에서 유저정보와 권한정보를 가져오게 된다. 해당 정보를 기반으로 userDetails.User 객체를 생성해서 리턴한다.
    - ### AuthController
        - `TokenProvider` 와 `AuthenticationManagerBuilder` 를 주입받는다.
        - `public ResponseEntity<TokenDto> authorize(@Valid @RequestBody LoginDto loginDto)` Post 요청을 받는 로그인 api , loginDto 의 유저이름 , 패스워드를 파라미터로 받고, 이를 이용해서 `UsernamePasswordAuthenticationToken` 객체를 생성한다

            ```java
            UsernamePasswordAuthenticationToken authenticationToken =
                    new UsernamePasswordAuthenticationToken(loginDto.getUserName(), loginDto.getPassword());
            ```

        - 토큰을 이용해서 Authentication 객체를 생성하려고 authenticate 메소드가 실행이 될 때 , loadUserByUsername 메소드가 실행된다.
            - `Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);`
        - Authentication 객체를 SecurityContext에 저장하고 , 이 객체를 가지고 creatToken 메소드를 통해서 JWT Token을 생성한다. 이 토큰을 Response Header 에 넣어주고 api의 반환값인 TokenDto를 이용해서 ResponseBody에도 넣어서 리턴한다.