---

layout: post
title: "인프런(김영한) filter 공부"
date: 2022-08-08

---

## 필터

#### 필터 흐름
- HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
모든 고객의 요청 로그를 남기는 요구사항이 있다면 필터를 사용.

필터는 특정 URL 패턴에 적용할 수있고, " /* " 이걸로 모든요청에 필터가 적용됨.

#### 필터 제한
- HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 	// 로그인 사용자
- HTTP 요청 -> WAS -> 필터 (적절하지 않은 요청이라 판단, 서블릿 호출 X)    // 비 로그인 사용자

필터에서 적정하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있어, 로그인 여부를 체크하기에는 딱 좋음.

#### 필터 체인
- HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러

필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가 가능 O, 로그를 남기는 필터를 먼저 적용하고, 
그 다음에 로그인 여부 체크하는 필터를 만들 수 있음

#### 필터 인터페이스

```
public interface Filter {
	public default void init(FilterConfig filterConfig) throws ServletException{}
	
	public void doFilter(ServletRequest request, ServletResponse response,
								FilterChain chain) throws IOException, ServletException;
								
	public default void destroy(){}
}

// doFilter 만 사용하면 됨 
// init, destroy는 default로 되어있어서 다른곳에서 사용할 필요가 없음
```

> Init(): 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출됨
> doFilter(): 고객의 요청이 올 때 마다 해당 메서드가 호출됨, 필터의 로직을 구현하는 메서드
> destroy(): 필터 종료메서드, 서블릿 컨테이너가 종료될 때 호출됨

#### LogFilter - 로그 필터

```
package hello.login.web.filter;
  import lombok.extern.slf4j.Slf4j;
  import javax.servlet.*;
  import javax.servlet.http.HttpServletRequest;
  import java.io.IOException;
  import java.util.UUID;
  @Slf4j
	/*
		필터를 사용하려면 필터 인터페이스를 구현해야함
	*/
  public class LogFilter implements Filter {
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
          log.info("log filter init");
      }
    	/**
      	- http 요청이 오면 doFilter가 호출이됨
      	- ServletRequest request는 http 요청이 아닌 겨우까지 고려해서 만든 인터페이스
      	  http를 사용하면 HttpServletRequest httpRequest = (HttpServletRequest)    
      	  request; 와 같이 다운 캐스팅 하면됨
      */
			@Override
      public void doFilter(ServletRequest request, ServletResponse response,
      FilterChain chain) throws IOException, ServletException {
              HttpServletRequest httpRequest = (HttpServletRequest) request;
              String requestURI = httpRequest.getRequestURI();
              //Http 요청을 구분하기 위해 요청당 임의의 uuid를 생성
        			String uuid = UUID.randomUUID().toString();
    
               try {
               // uuid와 requestURI를 출력
              log.info("REQUEST  [{}][{}]", uuid, requestURI);
              
              /**
              	이 부분이 가장중요, 다음 필터가 있으면 필터를 호출하는데 없으면
              	 서블릿을 호출함, 이 로직을 호출하지 않으면 다음 단계로 진행되지않음
              */
              chain.doFilter(request, response);
              
          } catch (Exception e) {
              throw e;
          } finally {
              log.info("RESPONSE [{}][{}]", uuid, requestURI);
} }
      @Override
      public void destroy() {
          log.info("log filter destroy");
      }
}

```

#### WebConfig - 필터 설정

```
package hello.login;
    import hello.login.web.filter.LogFilter;
    import org.springframework.boot.web.servlet.FilterRegistrationBean;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import javax.servlet.Filter;
    @Configuration
    public class WebConfig {
        @Bean
        public FilterRegistrationBean logFilter() {
            FilterRegistrationBean<Filter> filterRegistrationBean = new
    FilterRegistrationBean<>();
            filterRegistrationBean.setFilter(new LogFilter());
            filterRegistrationBean.setOrder(1);
            filterRegistrationBean.addUrlPatterns("/*");
            return filterRegistrationBean;
} }
```

필터를 등록하는 방법은 여러가지가 있지만, <span> 스프링 부트 </span>를 사용하면 FilterRegistrationBean을 사용해서 등록.

> setFilter(new LogFilter()): 등록할 필터를 지정
> setOrder(1): 필터는 체인으로 동작함. 따라서 순서가 필요하는데, 낮을수록 먼저 동작한다.
> addUrlPatterns("/*"): 필터를 적용할 URL 패턴을 지정한다. 한번엔 여러 패턴을 지정할 수 있다.

### 서블릿 필터 - <span style="font-weight:bold;"> 인증체크 </span>
( 비로그인자는 상품 관리 뿐만 아니라 미래에 개발될 페이지에도 접근하지 못하도록 개발 )

#### LoginCheckFilter - 인증 체크 필터

```

package hello.login.web.filter;
    import hello.login.web.SessionConst;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.util.PatternMatchUtils;
    import javax.servlet.*;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import javax.servlet.http.HttpSession;
    import java.io.IOException;
    @Slf4j
    public class LoginCheckFilter implements Filter {
    		/* 인증 필터를 적용해도 홈, 회원가입, 로그인 화면, css 같은 리소스에는 접근할 수 있어야한다.
    		   이렇게 화이트 리스트 경로는 인증과 무관하게 항상 허용한다. 화이트 리스트를 제외한 나머지 모든
    		*/ 경로에는 인증 체크 로직을 적용.
        private static final String[] whitelist = {"/", "/members/add", "/login",
    "/logout","/css/*"};
    
   @Override
   public void doFilter(ServletRequest request, ServletResponse response,
				FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        try {
          log.info("인증 체크 필터 시작 {}", requestURI); if (isLoginCheckPath(requestURI)) {
          log.info("인증 체크 로직 실행 {}", requestURI); HttpSession session = 			
          httpRequest.getSession(false); if (session == null ||
					session.getAttribute(SessionConst.LOGIN_MEMBER) == null) { log.info("미인증 사용자 요청 						{}", requestURI);
					
					/*
						미인증 사용자는 로그인 화면으로 다시 리다이렉트 해야함, 그런데 로그인 이후에 다시 홈으로 이동하면
						원하는 경로를 다시 찾아가야해서 고객들이 불편해함, 현재 요청한 경로인 requestURI를 /login에
						쿼리 파라미터로 함께 전달한다. 물론 /login 컨트롤러에서 로그인 성공시 해당 경로로 이동하는 기능은
						추가로 개발해야함
					*/
					//로그인으로 redirect 
          httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
          
          /* 
          	필터를 더는 진행하지 않는다, 이후 필터는 물론 서블릿, 컨트롤러가 더는 호출되지 않음
          	앞서 redirect를 사용했기 때문에 redirect가 응답으로 적용되고 요청이 끝남.
          */ 
          return; //여기가 중요, 미인증 사용자는 다음으로 진행하지 않고 끝! }
}
        chain.doFilter(request, response);
        
    } catch (Exception e) {
				throw e; //예외 로깅 가능 하지만, 톰캣까지 예외를 보내주어야 함 } finally {
				log.info("인증 체크 필터 종료 {}", requestURI); }
		}
    /**
    * 화이트 리스트의 경우 인증 체크X
    */
    private boolean isLoginCheckPath(String requestURI) {
        return !PatternMatchUtils.simpleMatch(whitelist, requestURI);
    }
    
}
```

#### WebConfig - loginCheckFilter() 추가

```
@Bean
  public FilterRegistrationBean loginCheckFilter() {
      FilterRegistrationBean<Filter> filterRegistrationBean = new
  FilterRegistrationBean<>();
      filterRegistrationBean.setFilter(new LoginCheckFilter());
      filterRegistrationBean.setOrder(2);
      filterRegistrationBean.addUrlPatterns("/*");
      return filterRegistrationBean;
}
```

> setFilter(new LoginCheckFilter()): 로그인 필터를 등록
> setOrder(2): 순서를 2번으로 잡았다. 로그 필터 다음에 로그인 필터가 적용된다.
> addUriPatterns("/*"): 모든 요청에 로그인 필터를 적용한다.

#### RedirectURL 처리
로그인에 성공하면 처음 ㅊ요청한 URL로 이동하는 기능을 개발

#### LoginController - loginV4()

```
/**
* 로그인 이후 redirect 처리
*/
  @PostMapping("/login")
  public String loginV4(
          @Valid @ModelAttribute LoginForm form, BindingResult bindingResult,
          @RequestParam(defaultValue = "/") String redirectURL,
          HttpServletRequest request) {
      if (bindingResult.hasErrors()) {
          return "login/loginForm";
}
      Member loginMember = loginService.login(form.getLoginId(),
  		form.getPassword());
      log.info("login? {}", loginMember);
      if (loginMember == null) {
			bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
          return "login/loginForm";
      }
//로그인 성공 처리
//세션이 있으면 있는 세션 반환, 없으면 신규 세션 생성
HttpSession session = request.getSession(); //세션에 로그인 회원 정보 보관
      session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
//redirectURL 적용
      return "redirect:" + redirectURL;
  }
 
```

로그인 체크 필터에서, 미인증 사용자는 요청 경로를 포함해서 <span style="font-weight:bold;">/login</span> 에 <span style="font-weight:bold;">redirectURL</span>> 요청 파라미터를 추가해서 요청했다. 이 값을 사용해서 로그인 성공시 해당 경로로 고객을 <span style="font-weight:bold;">redirect</span> 한다

*** 필터에는 스프링 인터셉터는 제공하지 않는 <span style="font-weight:bold;color:yellow;">chain.doFilter(request, response);</span> 를 호출할 때 request, response 를 다른 객체로 바꿀 수 있다. ServletRequest, ServletResponse 를 구현한 다른 객체를 만들어서 넘기면 해당 객체가 다음 필터 또는 서블릿에서 사용됨

