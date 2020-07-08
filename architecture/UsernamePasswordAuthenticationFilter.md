# UsernamePasswordAuthenticationFilter 

실제 로그인을 하게 되면 UsernamePasswordAuthenticationFilter 를 통해 처리된다.

로그인 처리가 되는 과정은 다음과 같다.

- UsernamePasswordAuthenticationFilter 의 attemptAuthentication 메서드 호출
  - 해당 메서드에서 POST 요청인지 확인(로그인은 POST 요청만 가능)
  - username 과 password 를 이용해서 UsernamePasswordAuthenticationToken 을 생성
  - 그리고 ProviderManager 의 authenticate 메서드로 리턴한다.
- ProviderManager 의 authenticate 메서드 호출
  - result.provider.authenticate(authentication) 에서 AbstractUserDetailsAuthenticationProvider 의 authenticate 메서드를 호출한다.
- AbstractUserDetailsAuthenticationProvider 의 authenticate 메서드 호출
  - user = this.retrieveUser(username, (UsernamePasswordAuthenticationToken)authentication); 코드에서 DaoAuthenticationProvider 의 retrieveUser 메서드 호출
  - loadUser = this.getUserDetailsService().loadUserByUsername(username); 에서 UserDetails 를 구현한 구현체의 loadUserByUsername 메서드를 호출하여 User 클래스를 상속받은 클래스에서
  User 객체를 생성한 후에 UserDetails 객체를 리턴한다. 성공하면 AbstractUserDetailsAutenticationProvider 의 createSuccessAuthentication 메서드를 호출하여
  UsernamePasswordAuthenticationToken 에 인증된 객체를 저장하고 UsernamePasswordAuthenticationToken 안에 있는 WebAuthenticationDetails 에 remoteAddress 와 sessionId 를 저장한다.
- 만들어진 Authentication 객체를 ProviderManager 로 반환
  - 다시 result.provider.authenticate(authentication) 코드로 오게된다.
  - 그리고 최종적으로 AbstractAuthenticationProcessingFilter 의 successfulAuthentication 에서 SecurityContextHolder 의 SecurityContext 에 인증된 객체인 Authentication 을 저장한다.
  - 그리고 this.successHandler.onAuthenticationSuccess(request, response, authResult); 에서 successHandler 를 호출한다.
  - AuthenticationSuccessHandler 를 구현체가 으면 해당 구현체를 호출한다.
