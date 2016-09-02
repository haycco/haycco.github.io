---
title: 创建RESTful风格和基于token方式验证的Spring Boot应用
date: 2016-08-21 18:38:21
tags: 
- Spring Boot
- Spring Security
- REST API
- token
---

## RESTful是什么？
可能有些人还不知道RESTful是什么，这里引用百度百科的部分解说：

>*RESTful是一种软件架构风格，设计风格而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。
>REST（英文：Representational State Transfer，简称REST）描述了一个架构样式的网络系统，比如 web 应用程序。它首次出现在 2000 年 Roy Fielding 的博士论文中，他是 HTTP 规范的主要编写者之一。在目前主流的三种Web服务交互方案中，REST相比于SOAP（Simple Object Access protocol，简单对象访问协议）以及XML-RPC更加简单明了，无论是对URL的处理还是对Payload的编码，REST都倾向于用更加简单轻量的方法设计和实现。值得注意的是REST并没有一个明确的标准，而更像是一种设计的风格。*

上面说的东西看着似懂非懂的样子，我们换一种程序员的说法。
在 REST 样式的 Web 服务中，每个资源都有一个地址。资源本身都是方法调用的目标，方法列表对所有资源都是一样的。这些方法都是标准方法，包括 HTTP GET、POST、PUT、DELETE，还可能包括 HEADER 和 OPTIONS。
我们在开发和设计的API的URL地址要跟HTTP的请求方式对应起来，让我们来看看一个标准的URL地址 `http://example.com/products/` 对应的CRUD操作是什么：
<!--more-->

*请求方式* | *操作说明*
--- | ---
GET | 查询
POST | 新增
PUT | 修改
DELETE | 删除

## 基于Token的验证原理
基于Token的身份验证是无状态的，我们不将用户信息存在服务器或Session中。
这种概念解决了在服务端存储信息时的许多问题，NoSession意味着你的程序可以根据需要去增减机器，而不用去担心用户是否登录。

基于Token的身份验证的过程如下:
1. 用户通过用户名和密码发送请求。
2. 程序验证。
3. 程序返回一个签名的token 给客户端。
4. 客户端储存token,并且每次用于每次发送请求。
5. 服务端验证token并返回数据。

每一次请求都需要token。token应该在HTTP的头部发送从而保证了Http请求无状态。我们同样通过设置服务器属性Access-Control-Allow-Origin:\* ，让服务器能接受到来自所有域的请求。需要主要的是，在ACAO头部标明(designating)\*时，不得带有像HTTP认证，客户端SSL证书和cookies的证书。

## 配置Spring Security权限验证的实际应用
在这里分享一个Spring boot应用的设计，该应用带有Spring Security，RESTful API，Token，API Gateway，Mock Bean等相关特性
授权的流程主要是通过判断用户的HTTP URL请求进行过滤区分后台管理地址还是普通服务地址，同时用Spring Security进行权限验证，通过过滤器进行分流不同的授权策略。授权成功时通过API网关进行相关服务实例的返回控制，以此达到各个API的控制粒度细微化。
客户端用户拿到授权成功后的Token值才能继续往下其他的操作。
与此同时，后台管理员和普通域用户的Token后续还有一些差别，普通域用户授权成功后的Token进行缓存，而后台管理员的Token不缓存，也就是说在此处，后台管理员每一次请求都需要验证用户名和密码。
具体流程如下图所示：

![用户授权处理流程图](/images/oauth_token.jpg "用户授权处理流程图")

### pom.xml
我们的pom.xml文件只需加入以下两个starter依赖
``` xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-rest-webmvc</artifactId>
</dependency>
```
### 设计两个用户角色
* 普通域用户角色 - ROLE_DOMAIN_USER
* 后台管理员角色 - ROLE_HAYCCO_ADMIN
### 权限配置类
我们再来看看权限配置类 org.haycco.spring.infrastructure.security.SecurityConfig 的部分代码
``` java
@Configuration
@EnableScheduling
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
...
	@Override
	protected void configure(HttpSecurity http) throws Exception {
	  http
	    .csrf().disable()
	    .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
	    .and()
	    .authorizeRequests()
	    .antMatchers(actuatorEndpoints()).hasRole(hayccoAdminRole)
	    .anyRequest().authenticated()
	    .and()
	    .anonymous().disable()
	    .exceptionHandling()
	    .authenticationEntryPoint(unauthorizedEntryPoint());

	  http
	    .addFilterBefore(new AuthenticationFilter(authenticationManager()), BasicAuthenticationFilter.class)
	    .addFilterBefore(new ManagementEndpointAuthenticationFilter(authenticationManager()), BasicAuthenticationFilter.class);
	}
...
}
```
### 过滤器
为了拦截服务端对不同用户角色请求的URL地址，此处我们定义了两个过滤器：
* 普通域用户访问地址的URL过滤器 AuthenticationFilter.java
* 后台管理员访问地址的URL过滤器 ManagementEndpointAuthenticationFilter.java

### 未授权的请求处理
在SecurityConfig里面我们对未授权的请求都统一设置HTTP状态为401，如下处理：
``` java
@Bean
public AuthenticationEntryPoint unauthorizedEntryPoint() {
  return new AuthenticationEntryPoint() {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
	    AuthenticationException authException) throws IOException, ServletException {
	response.sendError(HttpServletResponse.SC_UNAUTHORIZED);
    }
  };
}
```

### 授权适配器
#### 普通用户基于用户名和密码的授权
请求URL携带用户名和密码，我们的过滤器进行工作，流转至相关类 DomainUsernamePasswordAuthenticationProvider
``` java
package org.haycco.spring.infrastructure.security;

import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.util.StringUtils;

public class DomainUsernamePasswordAuthenticationProvider implements AuthenticationProvider {

    private TokenService tokenService;
    private ExternalServiceAuthenticator externalServiceAuthenticator;

    public DomainUsernamePasswordAuthenticationProvider(TokenService tokenService, ExternalServiceAuthenticator externalServiceAuthenticator) {
        this.tokenService = tokenService;
        this.externalServiceAuthenticator = externalServiceAuthenticator;
    }

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = String.valueOf(authentication.getPrincipal());
        String password = String.valueOf(authentication.getCredentials());

        if (StringUtils.isEmpty(username) || StringUtils.isEmpty(password)) {
            throw new BadCredentialsException("Invalid Domain User Credentials");
        }

        AuthenticationWithToken resultOfAuthentication = externalServiceAuthenticator.authenticate(username, password);
        String newToken = tokenService.generateNewToken();
        resultOfAuthentication.setToken(newToken);
        tokenService.store(newToken, resultOfAuthentication);

        return resultOfAuthentication;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```
#### 后台管理员基于用户名和密码的授权
请求URL携带用户名和密码，我们的过滤器进行工作，流转至相关类 HayccoAdminUsernamePasswordAuthenticationProvider
``` java
package org.haycco.spring.infrastructure.security;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.util.StringUtils;

public class HayccoAdminUsernamePasswordAuthenticationProvider implements AuthenticationProvider {

    public static final String INVALID_HAYCCO_ADMIN_CREDENTIALS = "Invalid Haycco Admin Credentials";

    @Value("${haycco.admin.username}")
    private String hayccoAdminUsername;

    @Value("${haycco.admin.password}")
    private String hayccoAdminPassword;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = String.valueOf(authentication.getPrincipal());
        String password = String.valueOf(authentication.getCredentials());

        if (credentialsMissing(username, password) || credentialsInvalid(username, password)) {
            throw new BadCredentialsException(INVALID_HAYCCO_ADMIN_CREDENTIALS);
        }

        return new UsernamePasswordAuthenticationToken(username, null,
                AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_HAYCCO_ADMIN"));
    }

    private boolean credentialsMissing(String username, String password) {
        return StringUtils.isEmpty(username) || StringUtils.isEmpty(password);
    }

    private boolean credentialsInvalid(String username, String password) {
        return !isHayccoAdmin(username) || !password.equals(hayccoAdminPassword);
    }

    private boolean isHayccoAdmin(String username) {
        return hayccoAdminUsername.equals(username);
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(HayccoAdminUsernamePasswordAuthenticationToken.class);
    }
}
```
#### 基于Token的授权
请求URL携带Token值，我们的过滤器进行工作，流转至相关类 TokenAuthenticationProvider
这里主要通过TokenService (EhCache)来缓存和移除相关超时的值
``` java
package org.haycco.spring.infrastructure.security;

import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.preauth.PreAuthenticatedAuthenticationToken;
import org.springframework.util.StringUtils;

public class TokenAuthenticationProvider implements AuthenticationProvider {

    private TokenService tokenService;

    public TokenAuthenticationProvider(TokenService tokenService) {
        this.tokenService = tokenService;
    }

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String token = String.valueOf(authentication.getPrincipal());
        if (StringUtils.isEmpty(token)) {
            throw new BadCredentialsException("Invalid token");
        }
        if (!tokenService.contains(token)) {
            throw new BadCredentialsException("Invalid token or token expired");
        }
        return tokenService.retrieve(token);
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(PreAuthenticatedAuthenticationToken.class);
    }
}
```
### API服务网关
如果我们要调用代理服务的话，我们需要获取相关网关接口的实现类，这里设计了一个适配器 AuthenticatedExternalWebService:
``` java
@Component
public class AuthenticatedExternalServiceProvider {

    public AuthenticatedExternalWebService provide() {
        return (AuthenticatedExternalWebService) SecurityContextHolder.getContext().getAuthentication();
    }
}
```
通过添加这个适配器对网关的实现并达到注入，进行相关控制
``` java
public abstract class ServiceGatewayBase {
    private AuthenticatedExternalServiceProvider authenticatedExternalServiceProvider;

    public ServiceGatewayBase(AuthenticatedExternalServiceProvider authenticatedExternalServiceProvider) {
        this.authenticatedExternalServiceProvider = authenticatedExternalServiceProvider;
    }

    protected ExternalWebServiceStub externalService() {
        return authenticatedExternalServiceProvider.provide().getExternalWebService();
    }
}
```

### RESTful的Controller示例
SampleController定义了可访问的普通用户角色，同时我们只要标注 @RestController 就能达到我们 @Controller 和 @ResponseBody 的效果
``` java
package org.haycco.spring.api.samplestuff;

import org.haycco.spring.api.ApiController;
import org.haycco.spring.domain.CurrentlyLoggedUser;
import org.haycco.spring.domain.DomainUser;
import org.haycco.spring.domain.Stuff;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@PreAuthorize("hasAuthority('ROLE_DOMAIN_USER')")
public class SampleController extends ApiController {
    private final ServiceGateway serviceGateway;

    @Autowired
    public SampleController(ServiceGateway serviceGateway) {
        this.serviceGateway = serviceGateway;
    }

    @RequestMapping(value = STUFF_URL, method = RequestMethod.GET)
    public List<Stuff> getSomeStuff() {
        return serviceGateway.getSomeStuff();
    }

    @RequestMapping(value = STUFF_URL, method = RequestMethod.POST)
    public void createStuff(@RequestBody Stuff newStuff, @CurrentlyLoggedUser DomainUser domainUser) {
        serviceGateway.createStuff(newStuff, domainUser);
    }
}
```

## 测试
这个程序其实很好测试，如果我们需要实施一个自动化回归测试来验证的话，Spring Boot还提供了很多种类型的测试特性的支持，在我的这个例子里，我创建了一个集成测试用例，讲嵌入我们的容器来部署相应的程序，并以最小的单位模拟完整的上下文，这个测试用例将会对AOP，过滤器和相关配置进行测试，虽然这样测试比较慢，但这样测试出来的结果比较有保证。
而且Spring Boot 1.4.0.RELEASE对测试套件进行了优化，让我们看看 SecurityTest 类
``` java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = { Application.class }, webEnvironment=WebEnvironment.DEFINED_PORT)
public class SecurityTest {
```
只要增加几个注解，就把JUnit环境和WEB容器运行环境需要的相关依赖进行了注入
而且我们要模拟相关的service bean的注入也是很简单
``` java
@MockBean
ExternalServiceAuthenticator someExternalServiceAuthenticator;

@MockBean
ServiceGateway serviceGateway;

@Autowired
ExternalServiceAuthenticator mockedExternalServiceAuthenticator;

@Autowired
ServiceGateway mockedServiceGateway;
```
我们找几个节点的测试用例，自己简单的看一下：
``` java
@Test
public void authenticate_withValidUsernameAndPassword_returnsToken() {
	authenticateByUsernameAndPasswordAndGetToken();
}

@Test
public void authenticate_withInvalidUsernameOrPassword_returnsUnauthorized() {
	String username = "test_user_2";
	String password = "InvalidPassword";

	BDDMockito.when(mockedExternalServiceAuthenticator.authenticate(anyString(), anyString())).
		thenThrow(new BadCredentialsException("Invalid Credentials"));

	given().header(X_AUTH_USERNAME, username).header(X_AUTH_PASSWORD, password).
		when().post(ApiController.AUTHENTICATE_URL).
		then().statusCode(HttpStatus.UNAUTHORIZED.value());
}

@Test
public void gettingStuff_withoutToken_returnsUnauthorized() {
	when().get(ApiController.STUFF_URL).
		then().statusCode(HttpStatus.UNAUTHORIZED.value());
}
```
利用模拟的Bean实例进行模式返回，获取我们需要的token值和未授权情况下的401处理

我们这次示例的运行，测试和部署在Maven里面都只需简单的几句命令就可完成
1. `mvn clean install`
2. `mvn spring-boot:run`
3. 打开浏览器访问 [https://localhost:8443/health](https://localhost:8443/health)

## 示例源代码
其它详细信息可以在我的Github进行查阅源代码   [请点我进行跳转](https://github.com/haycco/spring-boot-security-example)


