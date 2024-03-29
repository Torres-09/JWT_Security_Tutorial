## Jwt Introduction

- ### JWT 소개 , 프로젝트 생성
    - ### JTW
        - RFC 7519 웹 표준
        - JSON 객체를 사용해 토큰 자체에 정보를 저장하는 웹 토큰
        - 간편하고 쉽게 적용 가능하다 → 사이드 프로젝트에 유용함
        - 헤더 , 페이로드, 시그니처로 이루어짐
        - 헤더 : 시그니처를 해싱하는 알고리즘 정보 , 페이로드 : 서버와 클라이언트가 주고받는 시스템에서 실제로 사용될 정보에 대한 내용 , 시그니처 : 토큰의 유효성 검증을 위한 문자열
        - 장점 : 중앙의 인증서버 , 데이터 스토어에 대한 의존성이 없어 시스템 수평 확장 유리함.
        - 장점 : base64 url safe encoding을 사용해서 URL Cookie 헤더 모두 사용가능하다
        - 단점 : 페이로드의 정보가 많아지면 트래픽 사용량이 증가한다. → 데이터 설계 고려가 필요하다.
        - 단점 : 토큰이 클라이언트에 저장되기 때문에 서버에서 클라이언트의 토큰을 조작할 수 없다.
    - ### 프로젝트 설정
        - 기본 설정에 lombok , spring web , h2 , validation , spring data jpa 의존성 추가