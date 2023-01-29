## Spring Security SignUp

- 권한이 없는 상태에서 user manager admin으로 접근할 때 login하도록 유도 , loginForm 생성
- UserRepository
    - JpaRepository 상속받아서 기본적인 CRUD 사용
    - `@Repository`  가 없어도 IoC 적용된다. JpaRepository가 들고 있음.
- SecurityConfig

    ```java
    @Bean
    public BCryptPasswordEncoder encodePwd() {
        return new BCryptPasswordEncoder();
    }
    ```

    - 비밀번호를 바로 저장하는 것이 아니라 한 번 encode를 거친 후에 저장해준다.
- Controller

    ```java
    @PostMapping("/join")
    public String join(User user) {
        user.setRole("ROLE_USER");
        // 패스워드 암호화를 해주어야 한다.
        String rawPassword = user.getPassword();
        String encPassword = bCryptPasswordEncoder.encode(rawPassword);
        user.setPassword(encPassword);
    
        userRepository.save(user);
        return "redirect:/loginForm";
    }
    ```

    - 권한 지정은 직접해주고 패스워드는 받아서 한 번 변환한후에 넣어준다.