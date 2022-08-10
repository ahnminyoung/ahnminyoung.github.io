## 인터셉터

서블릿 필터가 서블릿이 제공하는 기술이라면, 스프링 인터셉터는  <span style="font-weight:bold;color:#CC9966;"> 스프링 MVC </span> 가 제공하는 기술. 웹과 관련된 공통 관심 사항을 처리하지만, 적용되는 순서와 범위, 그리고 사용방법은 다름

#### 스프링 인터셉터 흐름

> HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러

1. 스프링 인터셉터는 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출
2. 스프링 인터셉터는 <span style="font-weight:bold;color:#CC9966;"> 스프링 MVC </span>가 제공하는 기능이기 때문에 결국 디스패처 서블릿 이후에 등장하게 됨
3. **스프링 MVC**의 시작점이 **디스패처 서블릿**이라고 생각해보면 이해가 될 것.
4. 스프링 인터셉터에도 URL 패턴을 적용할 수 있는데, 서블릿 URL 패턴과는 다르고, 매우 정밀하게 설정 가능

#### 스프링 인터셉터 제한

> **HTTP** 요청 -> **WAS** -> **필터** -> **서블릿** -> **인터셉터** -> **컨트롤러**	// 로그인 사용자
>
> **HTTP** 요청 -> **WAS** -> **필터** -> **서블릿** -> **스프링 인터셉터**( 적절하지 않은 요청이라 판단, 컨트롤러 호출 X) //비로그인 사용자

#### 스프링 인터셉터 체인

> HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러

스프링 인터셉터는 체인으로 구성되어있고, 중간에 인터셉터를 자유롭게 추가할 수 있음. 예를 들어서 로그를 남기는 인터셉터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 인터셉터를 만들 수 있다.

서블릿 필터와 호출 되는 순서만 다르고 기능은 비슷함, 스프링 인터셉터가 서블릿 필터보다 편리하고, 정교하게 다양한 기능을 지원함.

#### 스프링 입터셉터 인터페이스
(스프링의 인터셉터를 사용하려면 HandlerInterceptor  인터페이스를 구현하면 됨)

```
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse
    response, Object handler) throws Exception {}
    
    default void postHandle(HttpServletRequest request, HttpServletResponse
    response,
    throws Exception {}
    
    Object handler, @Nullable ModelAndView modelAndView)
      default void afterCompletion(HttpServletRequest request, HttpServletResponse
    response,
    Exception {}
    Object handler, @Nullable Exception ex) throws
 }
```

- **서블릿 필터**의 경우 단순하게 **doFilter()** 하나만 제공되는데, **인터셉터**는 컨트롤러 호출 전 **( prehandle )**, 호출 후 **( postHandle )**, 요청 완료 이후**( aterCompletion )** 와 같이 단계적으로 잘 세분화 되어 있다.
- **서블릿 필터**의 경우 단순히 request, response 만 제공했지만, **인터셉터**는 어떤 컨트롤러 **( handler )** 가 호출 되는지 호출 정보도 받을 수 있다. 그리고 어떤 **modelAndView** 가 반환되는지 응답 정보도 받을 수 있다.

#### 스프링 인터셉터 호출 흐름

![](/Users/Ahnminyoung/Library/Application Support/typora-user-images/image-20220810143211232.png)

##### 정상 흐름

> - **preHandle**: 컨트롤러 호출전에 호출됨.( 더 정확히는 핸들러 어댑터 호출 전에 호출됨 )
>            **preHandle**의 응답값이 true **이면** 다음으로 진행하고, fals이면 더는 진행하지 않는다. **false** 경우 나머지 인터셉터는 물    론이고, 핸들러 어댑터도 호출되지 않음. 그림에서 1번에서 끝이 남
> - **postHandle**: 컨트롤러 호출 후에 호출된다. ( 더 정확히는 핸들러 어댑터 호출 후에 호출됨.)
> - **afterCompletion**: 뷰가 렌더링 된 이후에 호출된다.

#### 스프링 인터셉터 예외 상황

![](/Users/Ahnminyoung/Library/Application Support/typora-user-images/image-20220810143759699.png)

##### 예외 발생시

> - **preHandle**: 컨트롤러 호출 전에 호출된다.
> - **postHandle**: 컨트롤러에서 예외가 발생하면 **postHandle** 은 호출되지 않는다.
> - **afterCompletion**: **afterCompletion**은 항상 호출된다. 이 경우 예외 **( ex )** 를 파라미터로 받아서 어떤 예외가 발생 했는지 로그로 출력 할 수 있다.

##### afterCompletion은 예외가 발생해도 호출된다.

- **예외**가 발생하면 **postHandle()**는 호출되지 않으므로 예외와 무관하게 공통 처리를 하려면 **afterCompletion()** 을 사용해야 한다.
- **예외**가 발생하면 **afterCompletion()** 에 예외 정보 **( ex )** 를 포함해서 호출된다.

##### 정리

인터셉터는 스프링 MVC 구조에 특화된 필터 기능을 제공한다고 이해하면 된다. 스프링 MVC를 사용하고, 특별히 필터를 꼭 사용해야 하는 상황이 아니라면 인터셉터를 사용하는 것이 더 편리하다.

### 스프링 인터셉터 - 요청 로그

#### LogInterceptor - 요청 로그 인터셉터

```
package hello.login.web.interceptor;
  import lombok.extern.slf4j.Slf4j;
  import org.springframework.web.method.HandlerMethod;
         
	import org.springframework.web.servlet.HandlerInterceptor;
  import org.springframework.web.servlet.ModelAndView;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.util.UUID;
  
  @Slf4j
  public class LogInterceptor implements HandlerInterceptor {
  public static final String LOG_ID = "logId";
	@Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse
													response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();
        request.setAttribute(LOG_ID, uuid);
        //@RequestMapping: HandlerMethod
				//정적 리소스: ResourceHttpRequestHandler if (handler instanceof HandlerMethod) {
				HandlerMethod hm = (HandlerMethod) handler; //호출할 컨트롤러 메서드의 모든 정보가 포함되어 있다.
        }
        log.info("REQUEST  [{}][{}][{}]", uuid, requestURI, handler);
        return true; //false 진행X }
	@Override
  public void postHandle(HttpServletRequest request, HttpServletResponse
						response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
  }
	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse
  response, Object handler, Exception ex) throws Exception {
          String requestURI = request.getRequestURI();
          String logId = (String)request.getAttribute(LOG_ID);
          log.info("RESPONSE [{}][{}]", logId, requestURI);
          if (ex != null) {
              log.error("afterCompletion error!!", ex);
          }
} }
```

- request.setAttribute(LOG_ID, uuid);
  - **서블릿 필터**의 경우 **<u>지역변수로 해결이 가능</u>**하지만, **스프링 인터셉터**는 **<u>호출 시점이 완전히 분리</u>**되어 있다. 따라서 preHandle 에서 지정한 값을 **postHandle**, **afterCompletion** 에서 함께 사용하려면 어딘가에 담아두어야 한다. **LogInterceptor**도 <u>**싱글톤**</u> 처럼 사용되기 때문에 <u>**맴버변수를**</u> 사용하면 위험. 따라서 **request** 에 담아두어야함. 이 값은 **afterCompletion** 에서 **request.getAttribute(LOG_ID)** 로 찾아서 사용한다.
- return true
  - **true**면 **정상 호출**이다. 다음 **인터셉터나 컨트롤러가 호출**됨.

```
if (handler instanceof HandlerMethod) {
HandlerMethod hm = (HandlerMethod) handler; //호출할 컨트롤러 메서드의 모든 정보가
포함되어 있다. }
```

#### HandlerMethod

**핸들러 정보**는 어**떤 핸들러 매핑을 사용하는가에 따라 달라짐**. 스프링을 사용하면 일반적으로 **@Controller**, **@RequestMapping**을 활용한 **핸들러 매핑을 사용**하는데, 이경우 **핸들러 정보로 HandlerMethod**가 넘어옴

#### ResourceHttpRequestHandler

**@Controller**가 아니라 **/resources/static** 와 같은 정적 리소스가 호출 되는 경우 **ResourceHttpRequestHandler**가 핸들러 정보로 넘어오기 때문에 **타입에 따라서 처리**가 필요함

#### postHandle, afterCompletion

**종료 로그**를 **postHandle**이 아니라 **afterCompletion** 에서 실행한 이유는, **예외가 발생한 경우** postHandle가 호출되지 않기 때문이다. **afterCompletion**은 예외가 발생해도 호출 되는 것을 보장

### WebConfig - 인터셉터 등록

```
@Configuration
  public class WebConfig implements WebMvcConfigurer {
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          registry.addInterceptor(new LogInterceptor())
                  .order(1)
     }
//...
}
```

필터와 인터셉터를 같이 사용하고 있으면 중복될 수 있음 그래서 인터셉터를 사용하고 싶으면 필터를 주석.

> WebMvcConfigurer 가 제공하는 addInterceptors() 를 사용해서 인터셉터를 등록할 수 있음
>
> - registry.addInterceptor(new LogInterceptor()): 인터셉터를 등록한다.
> - order(1): 인터셉터의 호출 순서를 지정한다. 낮을 수록 먼저 호출됨
> - addPathPatterns("/**"): 인터셉터를 적용할 URL 패턴을 지정함
> - excludePathPatterns("/css/**", "/*.ico", "/error"): 인터셉터에서 제외할 패턴을 지정함

필터와 비교해보면 인터셉터는 addPathPatterns, excludePathPatterns 로 매우 정밀하게 URL 패턴을 지정할 수 있다.

##### 실행 로그

```
 REQUEST  [6234a913-f24f-461f-a9e1-85f153b3c8b2][/members/add]
 [hello.login.web.member.MemberController#addForm(Member)]
  
 postHandle [ModelAndView [view="members/addMemberForm";
 model={member=Member(id=null, loginId=null, name=null, password=null),
 org.springframework.validation.BindingResult.member=org.springframework.validat
 ion.BeanPropertyBindingResult: 0 errors}]]
  
 RESPONSE [6234a913-f24f-461f-a9e1-85f153b3c8b2][/members/add]
```

#### 스프링의 URL 경로

스프링이 제공하는 URL 경로는 서블릿 기술이 제공하는 URL 경로와 완전히 다르다. 더욱 자세하고, 세밀하게 설정할 수 있다.

![](/Users/Ahnminyoung/Library/Application Support/typora-user-images/image-20220810151631218.png)
