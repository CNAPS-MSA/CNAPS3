## Business Logic 추가 전, Configuration 설정

1. 각 service를 gateway의 end-point에 등록시켜주어야한다.
   
    등록하지 않을 경우, Access policy Filter에 request가 걸려져, 추가한 로직이 실행되지않을 수 있기 때문이다.

    gateway directory -> resources -> config -> **application-dev.yml 과 application-prod.yml 모두 수정**

    <img width="738" alt="image" src="https://user-images.githubusercontent.com/18453570/82303226-a2778300-99f5-11ea-972d-3c122d1ae752.png">


    위 이미지처럼 빨간 박스 부분을 수정해주면 된다.

    위 이미지엔 Rental만 추가 되었는데, Book 서비스, Delivery, Payment등 새로운 서비스를 추가할 때마다 Rental을 추가한 형식으로 추가해야한다.

    ```yaml
    jhipster:
      gateway:
        rate-limiting:
          enabled: false
          limit: 100000
          duration-in-seconds: 3600
        authorized-microservices-endpoints: # Access Control Policy, if left empty for a route, all endpoints will be accessible
          rental: /api,/v2/api-docs # recommended dev configuration
          book: /api,/v2/api-docs 
    ```


2. Security 설정 변경
   
    기존의 Gateway, Rental, Book의 `SecurityConfiguration.java`를 살펴보면 아래와 같이 작성되어있다.

    ```java
    @Override
        public void configure(HttpSecurity http) throws Exception {
            // @formatter:off
            http
                .csrf()
                .disable()
                .addFilterBefore(corsFilter, UsernamePasswordAuthenticationFilter.class)
                .exceptionHandling()
                    .authenticationEntryPoint(problemSupport)
                    .accessDeniedHandler(problemSupport)
            .and()
                .headers()
                .contentSecurityPolicy("default-src 'self'; frame-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://storage.googleapis.com; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self' data:")
            .and()
                .referrerPolicy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN)
            .and()
                .featurePolicy("geolocation 'none'; midi 'none'; sync-xhr 'none'; microphone 'none'; camera 'none'; magnetometer 'none'; gyroscope 'none'; speaker 'none'; fullscreen 'self'; payment 'none'")
            .and()
                .frameOptions()
                .deny()
            .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
                .authorizeRequests()
                .antMatchers("/api/authenticate").permitAll()
                .antMatchers("/api/register").permitAll()
                .antMatchers("/api/activate").permitAll()
                .antMatchers("/api/account/reset-password/init").permitAll()
                .antMatchers("/api/account/reset-password/finish").permitAll()
                .antMatchers("/api/**").authenticated()
                .antMatchers("/management/health").permitAll()
                .antMatchers("/management/info").permitAll()
                .antMatchers("/management/prometheus").permitAll()
                .antMatchers("/management/**").hasAuthority(AuthoritiesConstants.ADMIN)
            .and()
                .apply(securityConfigurerAdapter());
            // @formatter:on
        }
    ```

    위의 코드에서 `.antMatchers("/api/**").authenticated()`부분을 `.antMatchers("/api/**").permitAll()`로 수정해준다.
    `.antMatchers("/api/**").authenticated()`는 해당 경로로 들어오는 요청이 인증된 요청인 경우에만 허용한다는 의미인데, Swagger로 테스트 하던 중 SecurityFilter에 오류가 생기는 현상이 있어, 로직 개발 중엔 `permitAll()`로 수정하여 개발 및 테스트를 진행하였다. 
    >Gateway, Rental, Book의 각 SecurityConfiguration.java을 모두 위와 같이 수정하였다.
    >>오직 개발 또는 테스트 단계일 때만 `permitAll()`로 설정해야하며, prod단계에선 `authenticated()`를 적용해야한다.


