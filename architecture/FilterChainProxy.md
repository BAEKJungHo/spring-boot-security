# DelegatingFilterProxy 과 FilterChainProxy

![API](../images/s35.JPG)

Servlet Filter 는 Servlet 2.3 버전 부터 등장했다.

Servlet Filter 가 DelegatingFilterProxy 으로 요청을 보내면 DelegatingFilterProxy 을 통해 스프링 시큐리티가 Filter 기반으로 보안 처리를 할 수 있게 된다. 즉, DelegatingFilterProxy 은 Servlet Filter 이기 때문에 가장 먼저 요청을 받게 되고, 그 요청을 스프링에게 전달하는 것이다.

특정한 이름을 가진 빈을 찾아 그 빈에게 요청을 위임하는데 그게 바로 `DelegatingFilterProxy` 이다. 

localhost:8080 루트로 요청을 보낼때, 혹은 다른 요청이라도 가장 첫 번째로 반응하는 것은 DelegatingFilterProxy 클래스이다.

![API](../images/s36.JPG)

- AbstractAuthenticationProcessingFilter 

아래 메서드 디버깅을 통해 몇가지의 Filter 가 있는지 확인할 수 있다.

```java
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest)req;
        HttpServletResponse response = (HttpServletResponse)res;
        if (!this.requiresAuthentication(request, response)) {
            chain.doFilter(request, response);
        } else {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Request is to process authentication");
            }

            Authentication authResult;
            try {
                authResult = this.attemptAuthentication(request, response);
                if (authResult == null) {
                    return;
                }

                this.sessionStrategy.onAuthentication(authResult, request, response);
            } catch (InternalAuthenticationServiceException var8) {
                this.logger.error("An internal error occurred while trying to authenticate the user.", var8);
                this.unsuccessfulAuthentication(request, response, var8);
                return;
            } catch (AuthenticationException var9) {
                this.unsuccessfulAuthentication(request, response, var9);
                return;
            }

            if (this.continueChainBeforeSuccessfulAuthentication) {
                chain.doFilter(request, response);
            }

            this.successfulAuthentication(request, response, chain, authResult);
        }
    }
```  

위 총 15개의 필터에 대해서 처리가 완료되고나면 Servlet 에 접근할 수 있다.

![API](../images/s37.JPG)

DelegatingFilterProxy 가 springSecurityFilterChain 라는 이름을 빈으로 가지는 놈을 찾아서 요청에 대한 처리를 위임한다.

스프링에서 FilterChainProxy 를 빈으로 등록할 때 springSecurityFilterChain 이라는 이름으로 등록을 한다. 따라서 DelegatingFilterProxy 가 FilterChainProxy 에게 보안 처리를 위임하는 것이다. 보안처리가 완료되면 최종적으로 DispatcherServlet 으로 가서 남은 다른 요청들을 처리하게 된다.

## SecurityFilterAutoConfiguration

SecurityFilterAutoConfiguration 에서 DelegatingFilterProxy 를 등록하는 것을 볼 수 있고, springSecurityFilterChain 이름으로 등록하는 것을 볼 수 있다.

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@EnableConfigurationProperties({SecurityProperties.class})
@ConditionalOnClass({AbstractSecurityWebApplicationInitializer.class, SessionCreationPolicy.class})
@AutoConfigureAfter({SecurityAutoConfiguration.class})
public class SecurityFilterAutoConfiguration {
    private static final String DEFAULT_FILTER_NAME = "springSecurityFilterChain";

    public SecurityFilterAutoConfiguration() {
    }

    @Bean
    @ConditionalOnBean(
        name = {"springSecurityFilterChain"}
    )
    public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(SecurityProperties securityProperties) {
        DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean("springSecurityFilterChain", new ServletRegistrationBean[0]);
        registration.setOrder(securityProperties.getFilter().getOrder());
        registration.setDispatcherTypes(this.getDispatcherTypes(securityProperties));
        return registration;
    }

    private EnumSet<DispatcherType> getDispatcherTypes(SecurityProperties securityProperties) {
        return securityProperties.getFilter().getDispatcherTypes() == null ? null : (EnumSet)securityProperties.getFilter().getDispatcherTypes().stream().map((type) -> {
            return DispatcherType.valueOf(type.name());
        }).collect(Collectors.collectingAndThen(Collectors.toSet(), EnumSet::copyOf));
    }
}
```

## DelegatingFilterProxy

DelegatingFilterProxy 의 메서드 파라미터에 있는 targetBeanName 이 바로 springSecurityFilterChain 이다.

```java
public DelegatingFilterProxy(String targetBeanName, @Nullable WebApplicationContext wac) {
    this.targetFilterLifecycle = false;
    this.delegateMonitor = new Object();
    Assert.hasText(targetBeanName, "Target Filter bean name must not be null or empty");
    this.setTargetBeanName(targetBeanName);
    this.webApplicationContext = wac;
    if (wac != null) {
        this.setEnvironment(wac.getEnvironment());
    }
}
```    

## WebSecurityConfiguration

WebSecurityConfiguration 에서 springSecurityFilterChain 라는 이름으로 빈을 생성하는 것을 볼 수 있는데 this.webSecurity.build() 를 하면 FilterChainProxy 가 생성된다.

```java
    @Bean(name = {"springSecurityFilterChain"})
    public Filter springSecurityFilterChain() throws Exception {
        boolean hasConfigurers = this.webSecurityConfigurers != null && !this.webSecurityConfigurers.isEmpty();
        if (!hasConfigurers) {
            WebSecurityConfigurerAdapter adapter = (WebSecurityConfigurerAdapter)this.objectObjectPostProcessor.postProcess(new WebSecurityConfigurerAdapter() {
            });
            this.webSecurity.apply(adapter);
        }

        return (Filter)this.webSecurity.build();
    }
```

## WebSecurity

```java
 protected Filter performBuild() throws Exception {
        Assert.state(!this.securityFilterChainBuilders.isEmpty(), () -> {
            return "At least one SecurityBuilder<? extends SecurityFilterChain> needs to be specified. Typically this done by adding a @Configuration that extends WebSecurityConfigurerAdapter. More advanced users can invoke " + WebSecurity.class.getSimpleName() + ".addSecurityFilterChainBuilder directly";
        });
        int chainSize = this.ignoredRequests.size() + this.securityFilterChainBuilders.size();
        List<SecurityFilterChain> securityFilterChains = new ArrayList(chainSize);
        Iterator var3 = this.ignoredRequests.iterator();

        while(var3.hasNext()) {
            RequestMatcher ignoredRequest = (RequestMatcher)var3.next();
            securityFilterChains.add(new DefaultSecurityFilterChain(ignoredRequest, new Filter[0]));
        }

        var3 = this.securityFilterChainBuilders.iterator();

        while(var3.hasNext()) {
            SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder = (SecurityBuilder)var3.next();
            securityFilterChains.add(securityFilterChainBuilder.build());
        }

        // FilterChainProxy 생성
        FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);
        if (this.httpFirewall != null) {
            filterChainProxy.setFirewall(this.httpFirewall);
        }

        filterChainProxy.afterPropertiesSet();
        Filter result = filterChainProxy;
        if (this.debugEnabled) {
            this.logger.warn("\n\n********************************************************************\n**********        Security debugging is enabled.       *************\n**********    This may include sensitive information.  *************\n**********      Do not use in a production system!     *************\n********************************************************************\n\n");
            result = new DebugFilter(filterChainProxy);
        }

        this.postBuildAction.run();
        return (Filter)result;
    }
```    
