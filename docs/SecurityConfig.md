## Spring Security 설정  Data 설정

- ### Security 설정 , Data 설정
    - Security Config 설정 파일에 `@EnableWebSecurity` 어노테이션 추가 → 기본적인 웹 보안을 활성화 하겠다는 의미 , 추가적인 설정을 위해서는 WebSecurityConfigurer를 implements 하거나 WebSecurityConfigurerAdapter를 extends 하는 방법이 있다.

      [Spring Security without the WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

    - 기존 스프링 부트 버전에서는 이를 상속받아서 오버라이딩 하는 방식으로 구현했으나 이제는 스프링에서 SecurityFilterChain을 빈으로 등록하는 방식을 권장한다고 한다.
    - `.authorizeRequests()` 는 HttpServletReuqest를 사용하는 요청들에 대한 접근제한을 설정함
    - `.antMatchers("path").permitAll()` 은 path 에 대한 요청은 인증없이 접근을 허용
    - `.anyRequest()``.authenticated()` 나머지 요청들에 대해서는 모두 인증이 필수이다.
    - `httpBasic(withDefaults())` HTTP 기본 인증을 지원한다.
        - h2 console 접근 원활하게 하기 위한 Spring Security 설정 → h2-console 하위 모든 요청과 파비콘 관련 요청에서 spring security 로직을 실행하지 않음
        - `@Bean` `public WebSecurityCustomizer webSecurityCustomizer()`

            ```java
            return web -> web.ignoring().antMatchers(
                    "h2-console/**"
                    , "/favicon.ico"
            );
            ```

    - ### application.yml 설정
        - h2 DB 사용 , 메모리에 데이터 저장
        - JPA hibernate ddl-auto : create drop → SessionFactory가 시작될 때 Drop , Create , Alter 이후에 종료될 때 Drop
    - ### User Entity
        - 튜토리얼이기 때문에 롬복 기능을 적극적으로 사용 → 실무에서는 고려해야 함.
    - ### Authority Entity
        - User와 애노테이션은 동일 , 권한 이름을 저장할 PK
    - ### data.sql
        - 스프링 서버가 시작되면 자동으로 DB에 데이터를 넣어 줄 쿼리 저장. (create-drop이기 때문)
        - 왜 인지 모르겠지만 table 명을 user에서 user_tb로 변경했더니 오류 해결 → 아마도 user가 예약어로 설정되어있는 문제가 아닐까 추측
        - `defer-datasource-initialization: true` : 일정 스프링부트 버전부터는 하이버네이트가 초기화되기 이전에 data.sql을 실행하기 때문에 여기서 발생하는 에러를 없애기 위해서 해당 속성값을 application.yml에 추가한다. → 하이버네이트 초기화를 통해 생성된 스키마에 데이터 채우기