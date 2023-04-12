#### 타임리프 : 기본기능
- Object, user.username
- List, users[0].username
- Map, userMap['userA'].username

- 리터럴
  - 문자는 기본적으로 ' ' 으로 감싸야함.
  - 그러나 공백없는 한 단어는 ' ' 생략 가능
  - th:text="hello"
  - th:text="'hello World!'"

#### 타임리프 : 스프링 통합과 폼
- 부모태그에 th:object="${item}" 라고 선언하면 th:field="${item.name}" 이라고 하는 대신 th:field="*{name}" 만 해줘도 됨.
- th:field 선언하면 id, name, value 3가지를 한번에 만들어줌.
- form 태그에서 수정 못하게 할 항목에는 disabled 해주면됨.
- checkbox, radio 에서의 th:field와 th:value
  - th:value는 선택이 아니라 필수
  - th:field와 th:value가 일치하면 checked
- #ids.prev(), #ids.next()
  - label 태그에 주로 사용. 얘가 input 태그 앞에 위치하면 next, 뒤에 위치하면 prev
- @ModelAttribute가 메소드 레벨에 붙어있으면,
  - 해당 메소드가 반환하는 값이 addAttribute 됨.
  - 이는 해당 메소드를 가지고 있는 클래스 내부 전체에 다 기능함.
  - 성능상 다른 클래스로 빼서 static 으로 선언하면 좋겠지만, 별 차이 안나니까 그냥 써도 됨.
- Lombok 생성자 생성
  - @NoArgsConstructor : 인자 안받는 빈 생성자 생성
  - @AllArgsConstructor : 모든 필드를 인자로 받는 생성자 생성
  - @RequiredArgsConstructor : final 선언 혹은 @NonNull 선언된 필드를 인자로 받는 생성자 생성

#### 메시지, 국제화
- 메시지: 뷰에 입력하는 문자들을 하드코딩 하는게 아니라, message.properties 만들어서 메시지코드랑 메시지내용이랑 키 벨류로 매핑해둠.
- 타임리프에서는 #{}
- 스프링부트에서 해당 값을 가져오는 MessageSource 만들어줌. 그냥 Autowired만 해주면 됨. 근데 어차피 뷰로 바로 보낼 메시지들이라 자바코드내에서 확인할 필요가 있나. 아 테스트코드 작성할때.
- 국제화: message.properties를 국가별로 다르게 관리해서 http accept-lang 이나 locale에 따라 그에 맞는것을 보내주면 변경되겠지. 
- ms.getMessage(code, args, defaultMessage, locale) 호출하면, locale에 맞는 message.properites 찾아서 code에 매핑된 문자열 벨류값 반환함
- **Test 코드 예외처리**
  - Assertions.assertThatThrownBy( () -> function()).isInstanceOf(예외.class);


#### 검증1 : Validation
- BindingResult
  - @ModelAttribute로 잡은 객체에 요청값이 적절히 안들어왔을 경우 에러 사항들을 bindingResult에 담아줌.
  - 꼭! @ModelAttribute 매개변수 바로 다음에 선언되어야함.
  - bindingResult.addError() 를 통해 ObjectError와 FieldError 등을 수동으로 추가할 수 있음.
  - 해서 메소드 안쪽 본격 로직이 시작되기 전에 if(bindingResult.hasError()) 를 검사하며 에러가 있을 경우 실패 로직을 처리해버림.
  - 타임리프 스프링 검증 오류 통합 기능
    - #fields : #fields 로 BindingResult 가 제공하는 검증 오류에 접근할 수 있다.
    - th:errors : 해당 필드에 오류가 있는 경우에 태그를 출력한다. th:if 의 편의 버전이다. 
    - th:errorclass : th:field 에서 지정한 필드에 오류가 있으면 class 정보를 추가한다.
    - 글로벌 오류 처리
    ```
    <div th:if="${#fields.hasGlobalErrors()}">
      <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">전체 오류 메시지</p> 
    </div>
    ```
    - 필드 오류 처리
    ```
    <input type="text" id="itemName" th:field="*{itemName}" th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
    <div class="field-error" th:errors="*{itemName}">
        상품명 오류
    </div>
    ```
  - 그러나 에러를 bindingResult에 수동으로 등록하고 검증사항을 하나하나 자바 코드로 구현하고 이런것들이 너무너무너무 번거로워
  - 해서 나온게 Bean Validation

#### 검증2 : Bean Validation
- 객체 필드 위에다가 에노테이션 명시하면 그에 맞게 검증해주고, 검증 안맞으면 bindingResult에 에러내용 넣어줌. 정말 편하네
- @NotBlank(message="공백이면 안돼요"), @NotNull, @Range(min=1000, max=100000), @Max(9999),, 등등 이메일, 신용카드번호 등등 웬만한 유형은 다 있음.
- 오류가 발생했을떄 에러메시지를 내가 따로 명시해줘도 되고, spring에서 해당 에노테이션에 맞는 기본 메시지도 등록되어있으며,
- error.properties에서 단계별로 메시지들을 관리해놔도 됨.
  - NotBlank.객체.필드명=상품의 이름이 공백이면 안됩니다
- 글로벌 오류는 그냥 자바 코드로 bindingResult에 수동으로 에러 입력해주는게 나음.
  ```        
  if (item.getPrice() != null && item.getQuantity() != null) {
      int resultPrice = item.getPrice() * item.getQuantity();
      if (resultPrice < 10000) {
          bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
      } 
  }
  ```
- 등록할때의 필드 검증 사항과 수정할떄의 필드 검증 사항이 다를 경우. (사실 다를 경우가 훨씬 많음)
  - ItemSaveForm, ItemUpdateForm, Item을 따로 분리.
  - @ModelAttribute("item") ItemSaveForm form
    - @ModelAttribute에 따로 명시해주는거 주의. 따로 명시 안해주면 클래스명 맨 앞글자만 소문자로 변경된 itemSaveForm을 키값으로 model에 담김.
- @ModelAttribute vs @RequestBody
  - @ModelAttribute 는 각각의 필드에 타입 변환 시도. 성공하면 다음으로. 실패하면 typeMismatch 로 FieldError 추가. 변환에 성공한 필드만 BeanValidation 적용.
  - @RequestBody는 일단 http body로 요청이 들어온것을 http 메시지 컨버터를 통해 컨버팅하고, 이 json을 객체로 바꾼후에, 검증이 진행되는데, json이 객체로 바뀌지 않으면 예를 들어 입력 타입 자체가 다를 경우 검증이 진행이 안됨. 애초에 컨트롤러가 실행이 안되는 상황이 되어버림. 때문에 이 상황에서는.. 어떻게 해야하는지 뒤에 예외처리 파트에서 나온대.
  
#### 로그인 처리1 : 쿠키, 세션
- Member 에 loginId, password 필드 추가
- MemberRepository에 Member finByLoginId(String loginId) 메소드 추가
  - return findAll().filter(m -> m.getLoginId().equals(loginId)).findFirst();
- LoginService
  - Member login(String loginId, String password) 
    - return repository.findByLoginId(loginId).filter(m -> m.getPassword().equals(password)).orElse(null);
- LoginController
  - loginService.login() 에서 null을 반환하면 bindingResult.reject("loginfail", "아이디 또는 비밀번호를 잘못입력하였습니다"). 해서 글로벌 오류 추가.
- 로그인 성공 처리
  - 1_ 쿠키 사용
    - Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
    - response.addCookie(idCookie);
    - 이리 하면 response로 set-cookie: memberId=1 이 넘어가고
    - 새로 고침 하면 request에 cooke:memberId=1이 계속 넘어옴
    - 로그아웃은 그냥 새로운 쿠키 만들어서 cookie.setMaxAge(0); 해서 response에 해당 쿠키를 저장시켜버리면 됨.
  - 2_ 세션 사용
    - 세션 메커니즘 <br> <img width="600" alt="스크린샷 2023-04-10 오후 5 14 45" src="https://user-images.githubusercontent.com/93418349/230858984-48d82ba6-3c05-40b4-b33d-e7e014e1d4b6.png"> <br> <img width="600" alt="스크린샷 2023-04-10 오후 5 14 50" src="https://user-images.githubusercontent.com/93418349/230859024-d08c3a99-5030-463e-a38a-054b84cc239f.png">
    - HttpSession session = request.getSession(); // 세션 없으면 새로 생성
    - session.setAttribute("loginMember", loginMember); // 세션 저장소에 세션 저장 + 쿠키 발행
      - 위에서 new Cookie() 하고 reponse에 addCookie 했던게 이걸로 다 진행됨. 
    - 로그 아웃
      - HttpSession session = request.getSession();
      - if(session != null) sesson invalidate();
      - 세션 저장소에서 세션 삭제함.
    - @SessionAttribute(name = "sessionId", required = false) Member loginMember
      - request.getSession() 해서 세션 가져오고
      - session.getAttribute(...) 해서 세션에 저장된 value 가져오고 하는걸 한번에 해줌.
      - 없으면 null 반환하고.
    - url에 지져분하게 sessionId가 표시되는거
      - application.properties에서 server.servlet.session.tracking-modes=cookie
    - 가장 마지막 세션 접근 시간을 기준으로 30분이 지나면 WAS에서 해당 세션을 만료시켜줌.
    - 세션저장소는 db에 두는게 아니라 메모리에 두기에 회원객체를 그대로 저장하는게 아니라, 정말 딱 로그인에 필요한 자주 쓰는 필드만 담은 객체를 핏하게 따로 두는게 좋긴함.
- DTO
  - Data Transfer Object: 계층간 데이터 전송을 위해 도메인 모델 대신 사용되는 객체
  - 순수하게 데이터를 저장. Getter, Setter가 있고 비즈니스 로직은 있어서는 안돼.
- DAO
  - Data Access Object: 실제 DB에 접근하는 객체. Repository와 비슷한 기능과 역할.
- @NotNull vs @NotEmpty vs @NotBlank
  - @NotNull: null만 허용하지 않음. "" 나 " "는 허용.
  - @NotEmpty: null과 "" 둘다 허용하지 않음. " "는 허용
  - @NotBlank: null, "", " " 모두 허용하지 않음. 가장 강도가 높네


#### 로그인 처리2 : 필터, 인터셉터
- 필터
  - http요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 
  - 필터 인터페이스를 구현, 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성 및 관리함.
    - 구현: implements Filter 
      - init(), doFilter(), destroy();
      - doFilter에 핵심 로직을 담음.
        - LoginCheckFilter 라면, 로그인이 필요한 페이지인가? 세션을 가지고 있는가? 해당 세션이 세션 저장소에 저장되어 있는가?
        - 를 담아서, 만약 로그인이 필요한 페이지인데, 세션이 유효하지 않다면 response.sendRedirect("/login?redirectURL=" + requestURI); 해서
        - 이전에 요청 했던 URI를 쿼리로 담아 리다이렉트 시킴. requestURI 얘는 request에서 가져온애.
        - 이후 컨트롤러에서 @RequestParam("/") String requestURL로 해당 쿼리 받아서, 
        - return "redirect:" + requestURL; 해주면 로그인 이후 원래 접근하려 했던 페이지로 리다이렉트 되겠지.
        - chain.doFilter(request, response); 등록된 다른 필터가 있으면 그 다음 필터 호출해주고, 아니면 서블릿으로 넘어감.
     - 등록: 스프링 빈으로 등록. @Configuration 클래스에 @Bean으로 잡아서 FilterRegistrationBean 에다가 .setFilter, .setOrder, .addUrlPattern("/*") 
- 인터셉터
  - http요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
  - 스프링 인터셉터 기본 흐름 <br> <img width="800" alt="스크린샷 2023-04-06 오전 9 19 16" src="https://user-images.githubusercontent.com/93418349/230241942-723d30a6-f9f2-4ddb-be58-56a8931a3836.png">
  
  - 구현: implements HandlerInterceptor
    ```
    public interface HandlerInterceptor {
        default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {}
        default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {}
        default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {}
     }
     ```
    - preHandle : 핸들러어댑터 호출 직전에 호출됨. 예외 있으면 핸들러어댑터 호출 안하고 끝냄
    - postHandle: 핸들러어댑터 다 처리되고 호출됨. 핸들러어댑터(핸들러)에서 예외 터지면 호출 안됨
    - afterCompletion: 뷰 랜더링 마치고 http 응답 보내고 나서 호출됨. 단 핸들러에서 예외 터져도 호출됨. 어떤 예외가 발생했는지 로그로 찍어볼 수 있음
    - request와 response 뿐만 아니라 handler와 modelAndView, 예외까지도 응답 정보로 받을 수 있음
    - 인자로 Object handler를 받는데, 이를 캐스팅 해서 사용함 
      - HandlerMethod가 @Controller, @RequestMapping에 쓰던 핸들러
      - ResourceHttpRequestHandler가 정정리소스때 사용되는 핸들러
    - 로그인 체크 로직은 preHandler() 에만 하면 되겠지. 필터에서 doFilter()에 넣은거랑 비슷
    - 다만, 필터에서는 화이트리스트 url 체크하고, try catch finally문 쓰고 했지만 얘는 등록과정중에 허용 url를 설정해버리니 페이지를 확인할 필요 없이 호출된 페이지를 타겟팅으로 핵심 로직만 진행시킬 수 있고, 예외 처리도 필터보다 훨씬 더 단일책임을 지게 깔끔하며 url 설정도 더 세심하게 할 수 있음.
  - 등록
    - 필터처럼 프링 빈에 등록하는 방식이 아니라, registry에 add하는 방식
    ```
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new LogInterceptor())
                    .order(1)
                    .addPathPatterns("/**")
                    .excludePathPatterns("/css/**", "/*.ico", "/error");
       }
       //...
    }
    ```
  <!--
  - ArgumentResolver
    - 이름 그대로 매개변수를 어찌 처리할것이냐.
    - 일반 객체가 놓이면 ArgumentResolver가 해당 객체에 @ModelAttrubute 적용시키는것처럼, 
    - 공통된 매개변수 처리 로직에 대해 내가 따로 새로운 애노테이션을 만들어서 적용시킬 수 있어. 
    - @SessionAttrubute Member loginMember(name = .... requied = false..) 어쩌구하는걸 그냥 @Login 같은걸로 만들어서 사용하면 좋은데..
    - 이건 일단 스킵.
  -->
<!--
- 다시 정리 
  - 로그인 처리는 '쿠키, 세션, 필터, 인터셉터' 이 네가지의 개념이 필요.
  - 나는 로그인 한 사용자입니다 라는것을 인증해줄 수 있는 기능이 쿠키.
  - 쿠키에 중요 정보를 그대로 가지고 다니면 안되니까 세션id를 쿠키에 넣어 가지고 다니고 서버 세션 저장소(메모리)에 id값과 중요정보를 저장해둠.
  - 페이지 이동할때마다 로그인 사용자인가 아닌가를 판단해야되는데 그게 필터와 인터셉터.
  - 즉, 적용된 페이지에 모든 요청때마다 필터와 인터셉터를 거쳐서 알맞은 로그인 사용자인지를 확인하는거야.
  - 필터는 was 와 서블릿 사이에서, 인터셉터는 서블릿과 컨트롤러 사이에서.
  - 인터셉터가 페이지를 include, exclude 하기 더 편하고 이래저래 강력하기에 필터보다는 얘를 더 많이 씀
  - 더 요약하면, 쿠키는 여권, 세션id는 여권 번호, 필터와 인터셉터는 출입국심사대 같은거야.
-->

#### 예외 처리와 오류 페이지
- 전체 흐름
  - 1_ WAS(/error-ex, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
  - 2_ WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
  - 3_ WAS 오류 페이지 확인
  - 4_ WAS(/error-page/500, dispatchType=ERROR) -> 필터(x) -> 서블릿 -> 인터셉터(x) -> 컨트롤러(/error-page/500) -> View
- BasicErrorController 의 처리 순서
- 1_ 뷰템플릿 
  - resources/templates/error/500.html 
  - resources/templates/error/5xx.html
- 2_ 정적리소스(static,public) 
  - resources/static/error/400.html
  - resources/static/error/404.html
  - resources/static/error/4xx.html 
- 3_ 적용 대상이 없을 때 뷰 이름(error)
  - resources/templates/error.html 


#### API 예외 처리
- 예외 상황일때 오류 페이지 보낼꺼면 BasicErrorController, api 보낼거면 ExceptionResolver.
- ExceptionResolver <br> <img width="800" alt="스크린샷 2023-04-05 오후 8 29 51" src="https://user-images.githubusercontent.com/93418349/230067794-84e5e2d3-c3b8-49bc-8c33-aa5c73ee42be.png">
- ExceptionResolver를 사용하면 컨트롤러에서 예외가 발생해도 ExceptionResolver 에서 예외를 처리해버린다. 따라서 예외가 발생해도 서블릿 컨테이너까지 예외가 전달되지 않고, 스프링 MVC에서 예외 처리는 끝이난다. 결과적으로 WAS 입장에서는 정상 처리가 된 것이다. 이렇게 예외를 이곳에서 모두 처리할 수 있다는 것이 핵심이다. 서블릿 컨테이너까지 예외가 올라가면 복잡하고 지저분하게 추가 프로세스가 실행된다. 반면에 ExceptionResolver 를 사용하면 예외처리가 상당히 깔끔해진다. 그런데 직접 ExceptionResolver 를 구현하려고 하니 상당히 복잡하다. 지금부터 스프링이 제공하는 ExceptionResolver 들을 알아보자.
- 이 ExceptionResolver 중에 스프링에서는 가장 높은 우선순위로 작동하는게 ExceptionHandlerExceptionResolver 인데 얘를 @ExceptionHandler를 통해 쉽게 가져다 사용할 수 있음
- @ExceptionHandler를 통한 예외 처리
  - IllegalArgumentException 처리
  ```
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  @ExceptionHandler(IllegalArgumentException.class)
  public ErrorResult illegalExHandle(IllegalArgumentException e) {
      log.error("[exceptionHandle] ex", e);
      return new ErrorResult("BAD", e.getMessage());
  }
  ```
 - 실행 흐름
- 컨트롤러를 호출한 결과 IllegalArgumentException 예외가 컨트롤러 밖으로 던져진다. 
- 예외가 발생했으로 ExceptionResolver 가 작동한다. 
- 가장 우선순위가 높은 ExceptionHandlerExceptionResolver 가 실행된다.
- ExceptionHandlerExceptionResolver 는 해당 컨트롤러에 IllegalArgumentException 을 처리할 수 있는 @ExceptionHandler 가 있는지 확인한다.
- illegalExHandle() 를 실행한다. 
- @RestController 이므로 illegalExHandle() 에도 @ResponseBody 가 적용된다. 
- 따라서 HTTP 컨버터가 사용되고, 응답이 JSON으로 반환된다.
- @ResponseStatus(HttpStatus.BAD_REQUEST) 를 지정했으므로 HTTP 상태 코드 400으로 응답한다.
```
// UserException 처리
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {
    log.error("[exceptionHandle] ex", e);
    ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
    return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
}
// Exception 처리
@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
@ExceptionHandler
public ErrorResult exHandle(Exception e) {
    log.error("[exceptionHandle] ex", e); 
    return new ErrorResult("EX", "내부 오류");
}
```
- @ExceptionHandler 는 마치 @RequestMapping 처럼 예외가 발생시 예외를 잡아다가 실행시키는 느낌.
- @ControllerAdvice
  - 해당 컨트롤러에서 발생한 예외에서만 처리가 제한되었는데, 이걸 조금 더 글로벌하게 설정하기 위한것.
  - 클래스 만들어서 클래스 레벨에 @ControllerAdvice 붙이고, api만 할거니까 @RestControllerAdvice 라 붙이면 더 좋고.
  - @ExceptionHandler 붙였던 메소드들 다 여기 클래스에 잘라 붙여넣기하고, 어느 컨트롤러에 타겟팅할것인지 설정해주면 됨.
  - @ControllerAdvice(여기) 에다가 대상 컨트롤러 명시해주는데, 아무것도 명시하지 않으면 모든 컨트롤러에 적용.
  - @ControllerAdvice(annotations = RestController.class) // 에노테이션 범위로 지정
  - @ControllerAdvice("org.example.controllers") // 패지키 범위로 지정 // 이걸 많이 쓰는듯
  - @ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class}) // 특정 클래스로 지정.
  
#### 스프링 타입 컨버터
- @ModelAttribute, @RequestParam, @PathVariable, 뷰 템플릿 등에서 이미 스프링이 알아서 문자를 숫자로 숫자를 문자로 타입 바꿔주고 있음.
- 이는 HttpMessageConverter와 관련 없는 서비스임. HttpMessageConverter 얘는 http body 메시지에 있는 json을 객체로 객체를 json으로 바꿔주는거야.
- 애노테이션을 활용하여 포맷터 적용하기. 양방향으로 변환됨. 타임리프에서는 중괄호를 두개 사용하면 ${{ }} 내가 설정한 포맷터가 적용됨. th:field는 자동으로 컨버팅 해주고.
  ```
  @NumberFormat(pattern = "###,###")
  private Integer number;
  @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
  private LocalDateTime localDateTime;
  ```

#### 파일 업로드
- @RequestParam MultipartFile file 로 파일 가져올 수 있고.
- ```
  if (!file.isEmpty()) {
      String fullPath = fileDir + file.getOriginalFilename(); 
      file.transferTo(new File(fullPath));
  }
  ```
- fileDir: 파일을 저장할 내 로컬 경로.
  - application.properties 에 file.dir=/Users/kimyounghan/study/file/
  - @Value("${file.dir}") private String fileDir; 라 하면 application.properties에 설정해놓은 값이 들어옴.
- file.getOriginalFilename() : 업로드 파일 명
- file.transferTo(...): 파일 저장
- 파일은 데이터 베이스에 저장하는게 아니라, 스토리지에 저장하는거. AWS를 이용하는 경우 S3에 저장. DB에는 파일 자체를 저장하는게 아니라 파일 경로를 저장함.
- html input 속성중 multiple="multiple" 해주면 파일을 다중 선택 할 수 있음.
