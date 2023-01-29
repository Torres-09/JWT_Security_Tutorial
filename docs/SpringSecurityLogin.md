## Spring Security Login

- ### 시큐리티 로그인
    - `loginProcessingUrl("/login");`  SecurityConfig 추가 속성에 이것을 추가한다.
    - login 주소가 호출되면 시큐리티가 대신 로그인 로직을 진행해준다.
    - `.defaultSuccessUrl("/");` 로그인이 완료되면 메인페이지로 이동하도록 속성을 추가해준다.
- ### 시큐리티 Session
    - PrincipalDetail ( UserDetail )
        - 시큐리티 로그인이 완료되면 로그인이 완료되었다는 사실을 저장해야 한다.
        - 이를 시큐리티 세션에 저장한다. 시큐리티 세션에는 Authentication 객체가 들어갈 수 있고 Authentication 객체는 UserDetail 객체를 가지고 있을 수 있다.
        - PrincipalDetail 이라는 클래스를 만들어서 UserDetails 를 implements 한다. 그러면 Authentication 객체는 PrincipalDetail 객체를 가질 수 있다.
        - 이후에는 각 메소드들을 오버라이드해주어야 한다.
        - 유저가 가진 권한을 반환해주는 함수

            ```java
            @Override
            public Collection<? extends GrantedAuthority> getAuthorities() {
                Collection<GrantedAuthority> collect = new ArrayList<>();
                collect.add(new GrantedAuthority() {
                    @Override
                    public String getAuthority() {
                        return user.getRole();
                    }
                });
                return collect;
            }
            ```

            - 반환 타입이 정해져있기 때문에 반환 타입을 선언한 후에 그 안에 유저의 role을 반환하도록 오버라이드 한 후 return 한다.
        - 유저가 가진 패스워드를 반환해주는 함수

            ```java
            @Override
            public String getPassword() {
                return user.getPassword();
            }
            ```

            - 여기서는 String 타입을 반환하기 때문에 유저의 패스워드를 바로 반환해주면 된다.
        - 유저의 이름을 반환해주는 함수

            ```java
            @Override
            public String getUsername() {
                return user.getUsername();
            }
            ```

        - 계정이 만료되었는 지를 체크하는 함수

            ```java
            @Override
            public boolean isAccountNonExpired() {
                return true;
            }
            ```

        - 그 밖에도 계정 활성화, 계정 비밀번호 기간 만료, 계정 잠김 등의 함수들이 있다.
        - **예제이기 때문에 바로 true 혹은 false를 반환하지만 실제로 개발할 때는 예를 들어 사이트에 1년 간 로그인을 안하면 휴먼 계정이 되는 등의 처리를 할 수 있다.**
    - PrincipalDetailService
        - 시큐리티 설정에 .loginProcessingUrl("/login")를 추가했기 때문에 login 요청이 오면 자동으로 UserDetailsService 타입으로 IoC 되어 있는 loadUserByUsername 함수가 실행이 된다.
        - 그렇기 때문에 loadUserByUsername 를 오버라이딩 해주어야 로그인이 될 것이다.

            ```java
            @Override
            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
                User userEntity = userRepository.findByUsername(username);
                if (userEntity != null) {
                    return new PrincipalDetails(userEntity);
                }
                return null;
            }
            ```

        - 반환 타입이 UserDetails이다. 즉 로그인을 했을 때, username이 유효하다면 시큐리티 Session안에 Authentication안에 PrincipalDetails 객체를 넣게 된다. 로그인 완료!
        - **이전에 user로 접근했을 때 로그인 한 상태가 아니라면 로그인 페이지로 이동하도록 만들었다. 이 때 로그인을 하게 되면 default인 index 페이지가 아니라 접근하려고 했었던 user페이지로 이동하게 된다. 즉 default는 login 페이지에 접근하여 로그인했을 때 이동하는 페이지이다.**
- ### 시큐리티 권한처리
    - `@EnableGlobalMethodSecurity(securedEnabled = true)` 을 securityConfig에 추가하면 secured 어노테이션을 활성화하여 사용할 수 있다.

        ```java
        @Secured("ROLE_ADMIN")
        @GetMapping("/info")
        public @ResponseBody String info() {
            return "개인정보";
        }
        ```

    - `@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)`   prePostEnabled를 추가하여 preAuthorize , postAuthorize를 활성화할 수 있다.

        ```java
        @PreAuthorize("hasRole('ROLE_MANAGER') or hasRole('ROLE_ADMIN')")
        @GetMapping("/data")
        public @ResponseBody String data() {
            return "데이터정보";
        }
        ```

    - PreAuthorize는 해당 컨트롤러가 작동하기 전에 먼저 작동한다. 여러가지 권한을 설정할 수 있다. 만약 하나의 권한만 이용하고 싶다면 secured를 이용하는 것이 적절하다. 괄호안에 문법도 조금 다르다.
    - PostAuthorize도 있다. 컨트롤러가 끝나고 난 뒤 작동한다. 거의 사용하지 않는다.
    - 이런 방법들은 securityConfig 설정에 추가하는 게 아니라 메소드 하나에 지정하고 싶을 때 사용하면 좋은 방법이다.