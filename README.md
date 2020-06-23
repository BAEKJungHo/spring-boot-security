# 스프링 부트 기반 스프링 시큐리티 프로젝트

- Spring Boot
- JDK 1.8
- Postgresql
- Spring Data JPA
- Thymeleaf
- Spring MVC 
- Spring Security

## 강좌 소개

- 메소드 보안
  - AOP 기반
- URL 기반 보안
  - Filter 기반

## 프로젝트 생성

초기에는 spring initializr 를 이용하여 `spring web` 만 선택해서 스프링 부트 프로젝트를 생성한다.

> spring web 을 선택해서 프로젝트를 생성하면 spring-boot-starter-web 의존성이 자동으로 추가가 된다.

## IndexController 생성

이름은 뭐든 상관 없다. xxxApplication.java 와 동일한 위치에 WelcomeController(IndexController) 를 생성해준다.

그리고 톰캣이 제대로 구동이 됬는지 확인하기 위해 @RestController 어노테이션을 붙이고 `루트(/)` 로 접속할 수 있도록
핸들러 메서드를 생성해준다.

```java
@RestController
public class SecurityController {

    @GetMapping("/")
    public String index() {
        return "home";
    }

}
```

그리고 `localhost:8080/` 으로 접속하면 home 이라는 문자열이 찍히는 것을 확인할 수 있다.

## 시큐리티 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

시큐리티 의존성을 추가하고 톰캣을 시작하면 콘솔에 아래와 같은 문자열이 찍힌다.

 `Using generated security password: b6047e41-e261-1234-123a-9e6abc11b059` 	

스프링 시큐리티가 기본으로 제공하는 패스워드이다. 아이디는 `user` 라는 문자열을 기본으로 제공한다. 그리고 다시 `localhost:8080/` 으로 접속하면 로그인 페이지가 나오게된다. 
그리고 id 와 pw 를 입력하여 접속하게 되면 home 이라는 문자열이 정상적으로 나오게 된다.

> 즉, 시큐리티 의존성을 추가함으로써 인증을 받아야만 자원(리소스)에 접근 가능하게 되었다.

- 스프링 시큐리티 의존성 추가 시 일어나는 일들
	- 서버가 기동되면 스프링 시큐리티의 초기화 작업 및 보안 설정이 이루어진다.
	- 별도의 설정이나 구현을 하지 않아도 기본적인 웹 보안 기능이 현재 시스템에 연동되어 작동한다.
		+ 모든 요청은 인증이 되어야 자원에 접근이 가능하다.
		+ 인증 방식은 폼 로그인 방식과 httpBasic 로그인 방식을 제공한다.
		+ 기본 로그인 페이지를 제공한다.
		+ 기본 계정 한 개를 제공한다. (id : user / pw : 콘솔에 찍히는 랜덤 문자열)