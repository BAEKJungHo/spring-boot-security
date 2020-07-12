# ThreadLocal

- Java.lang 패키지에서 제공하는 스레드 범위 변수. 즉, 스레드 수준의 데이터 저장소
  - 같은 스레드 내에서만 공유
  - 따라서 같은 스레드라면 해당 데이터를 메서드 매개변수로 넘겨줄 필요 없음
  - SecurityContextHolder 의 기본 전략 
    - SercurityContextHolder 내부에 감춰져있는 Principal 에서 로그인한 User 정보가 스레드 단위로 유지된다.
    - 서블릿은 클라이언트 요청 1개마다 하나의 스레드를 생성해서 요청을 처리하기 때문에, `1명 = 1스레드` 라고 생각하면 된다.
 
  
```java
public class AccountContext {

    private static final ThreadLocal<Account> ACCOUNT_THREAD_LOCAL = new ThreadLocal<>();

    public static void setAccount(Account account) {
        ACCOUNT_THREAD_LOCAL.set(account);
    }

    public static Account getAccount() {
        return ACCOUNT_THREAD_LOCAL.get();
    }

}
```

- Controller

```java
/**
 * 로그인이 필요한 페이지
 * @param model
 * @param principal
 * @return
 */
@GetMapping("/dashboard")
public String dashboard(Model model, Principal principal) {
    model.addAttribute("message", "Hello " + principal.getName());
    
    // ThreadLocal 에 account 정보 세팅
    AccountContext.setAccount(accountRepository.findByUsername(principal.getName()));
    
    sampleService.dashboard();
    return "dashboard";
}
```

- Service

```java
@Secured({"ROLE_USER", "ROLE_ADMIN"})
public void dashboard() {
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    UserDetails userDetails = (UserDetails) authentication.getPrincipal();
    
    // ThreadLocal 에 저장된 account 꺼내기
    Account account = AccountContext.getAccount();
    
    System.out.println("===============");
    System.out.println(authentication);
    System.out.println(userDetails.getUsername());
}
```

- SecurityContextHolder 의 경우 위 처럼 할 필요 없고 아래 처럼 그냥 가져다 쓰면 된다.

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
UserDetails userDetails = (UserDetails) authentication.getPrincipal();
 ```

