# 스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술
-------------
### 단원별 요점 정리

#### 타임리프 : 기본기능
- Object, user.username
- List, users[0].username
- Map, userMap['userA'].username

<!--
```
<div th:with="first=${users[0]}">
  <p>처음 사람의 이름은 <span th:text="${first.username}"></span></p> 
</div>
```
- HTTP 요청 파라미터 접근: param. ex) ${param.paramData}
- HTTP 세션 접근: session. ex) ${session.sessionData}
- 스프링 빈 접근: @ ex) ${@helloBean.hello('Spring!')}
```
@GetMapping("/date")
public String date(Model model) {
    model.addAttribute("localDateTime", LocalDateTime.now());
    return "basic/date";
}
<span th:text="${#temporals.format(localDateTime, 'yyyy-MM-dd HH:mm:ss')}">
</span>
```
-->

- 리터럴
  - 문자는 기본적으로 ' ' 으로 감싸야함.
  - 그러나 공백없는 한 단어는 ' ' 생략 가능
  - th:text="hello"
  - th:text="'hello World!'"

<!--
```
<span th:text="'미성년자'" th:if="${user.age lt 20}"></span>
```
- gt, ge, lt, le, eq, ne // 크다 크거나같다 작다 작거나같다 같다 같지않다
```
<th:block>
유일한 타임리프 자체 태그. block.
```
```
<script th:inline="javascript">
자바스크립트 인라인. js 쓸때는 왠만하면 이거 해주는게 편해.
```
```
<footer th:fragment="copy">
푸터 자리 입니다. 
</footer>

<footer th:fragment="copyParam (param1, param2)">
<p>파라미터 자리 입니다.</p>
<p th:text="${param1}"></p> <p th:text="${param2}"></p>
</footer>

<div th:replace="~{template/fragment/footer :: copy}"></div>
<div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터2')}"></
div>
```
-->
- 나머지 아주 다양한 타임리프 문법이 있는데, 필요하면 그냥 레퍼런스 문서 보거나, 챕터 마지막 정리 강의 보거나 하자

#### 타임리프 : 스프링 통합과 폼
- 부모태그에 th:object="${item}" 라고 선언하면 th:field="${item.name}" 이라고 하는 대신 th:field="*{name}" 만 해줘도 됨.
- th:field 선언하면 id, name, value 3가지를 한번에 만들어줌.
- form 태그에서 수정 못하게 할 항목에는 disabled 해주면됨.
- checkbox, radio 에서의 th:field와 th:value
  - th:value는 선택이 아니라 필수
  - th:field가 기본값일때, field와 value가 일치하면 checked
  - th:field가 리스트일때, field에 value가 있으면 checked
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
- 메시지: 뷰에 입력하는 문자들을 하드코딩 하는게 아니라, 마치 자바에서 오류메시지 enum 클래스 만들어서 담아두는것 처럼, 띄울 문자를 매핑해두면, 모든 페이지에서의 '상품이름'을 '상품명'으로 수정 하기 수월하겠지. message.properties 만들어서 메시지코드랑 메시지내용이랑 키 벨류로 매핑해둠.
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
- Validaion 라이브러리 있음. start.spring.io에서 처음 프로젝트 generate할때 추가하면됨.
- 객체 필드 위에다가 에노테이션 명시하면 그에 맞게 검증해주고, 검증 안맞으면 bindingResult에 에러내용 넣어줌. 정말 편하네
- @NotBlank(message="공백이면 안돼요"), @NotNull, @Range(min=1000, max=100000), @Max(9999),, 등등 이메일, 신용카드번호 등등 웬만한 유형은 다 있음.
- 오류가 발생했을떄 에러메시지를 내가 따로 명시해줘도 되고, spring에서 해당 에노테이션에 맞는 기본 메시지도 등록되어있으며,
- error.properties에서 단계별로 메시지들을 관리해놔도 됨.
  - NotBlank.객체.필드명=상품의 이름이 공백이면 안됩니다
- 다만, 필드 오류는 이렇게 필드마다 간편하게 검증 사항을 적용할 수 있는데, 오브젝트 오류는 방법이 없음. @ScriptAssert()를 가지고 할 수 있긴한데, 그냥 자바 코드로 bindingResult에 수동으로 에러 입력해주는게 나음.
  ```        
  if (item.getPrice() != null && item.getQuantity() != null) {
      int resultPrice = item.getPrice() * item.getQuantity();
      if (resultPrice < 10000) {
          bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
      } 
  }
  ```
- 등록할때의 필드 검증 사항과 수정할떄의 필드 검증 사항이 다를 경우. (사실 다를 경우가 훨씬 많음)
  - groups라는 개념으로 해결할 수 있는데 그것보다는 차라리 폼 전용 객체를 따로 만드는게 나은 판단
  - 현재 item 하나로만 controller에서 요청 받아서 repository로 넘겨서 저장하고 수정하고 다 했는데, 이는 별로 바람직하지는 않음.
  - 따라서 ItemSaveForm, ItemUpdateForm, Item을 따로 분리.
  - 검증 에노테이션을 각 상황에 맞춰 따로따로 줄 수 있음
  - 물론 controller에서든 어디에서든 ItemSaveForm을 Item으로 변환하는 과정이 필요하겠지.
  - @ModelAttribute("item") ItemSaveForm form
    - @ModelAttribute에 따로 명시해주는거 주의. 따로 명시 안해주면 클래스명 맨 앞글자만 소문자로 변경된 itemSaveForm을 키값으로 model에 담김.
- @ModelAttribute vs @RequestBody
  - @ModelAttribute 는 각각의 필드에 타입 변환 시도. 성공하면 다음으로. 실패하면 typeMismatch 로 FieldError 추가. 변환에 성공한 필드만 BeanValidation 적용.
  - @RequestBody는 일단 http body로 요청이 들어온것을 http 메시지 컨버터를 통해 컨버팅하고, 이 json을 객체로 바꾼후에, 검증이 진행되는데, json이 객체로 바뀌지 않으면 예를 들어 입력 타입 자체가 다를 경우 검증이 진행이 안됨. 애초에 컨트롤러가 실행이 안되는 상황이 되어버림. 때문에 이 상황에서는.. 어떻게 해야하는지 뒤에 예외처리 파트에서 나온대.
  
#### 로그인 처리1 : 쿠키, 세션
- 패키지 구조 설계
  - 일단 크게 domain과 web을 분리
  - 도메인 : 화면, UI, 기술 인프라 등등의 영역은 제외한 시스템이 구현해야 하는 핵심 비즈니스 업무 영역
  - 둘의 의존관계가 단방향. web은 domain을 알지만, domain은 web을 모르게.
  - web이 완전히 삭제되어도 domain에는 영향이 없게.
  - web에는 ItemSaveForm, ItemEditForm, ItemController
  - domain에는 Item, ItemRepository
  - 패키지 구조 설계에 대한 더 자세한 내용은 (https://cheese10yun.github.io/spring-guide-directory/)
- 적절한 쿠키를 가지고 있냐 없냐에 따라 return 해주는 뷰 페이지가 다르게.
- 쿠키만을 가지고 로그인 처리를 하면, 아주 쉽게 조작할 수 있어. 보안상 문제가 커. 때문에 나온 개념이 세션.
- 최대한 중요한 정보는 서버가 가지고 있고, 임의의 아주 긴 문자열(세션id)과 객체를 맵핑해둔 세션저장소를 서버가 가지고서 관리하여 세션값을 쿠키에 넣어서 주고받는거야.
- spring 에서 제공하는 세션의 경우 가장 마지막 세션 접근 시간을 기준으로 30분이 지나면 WAS에서 해당 세션을 만료시켜줌. 당연히 타임아웃값 다르게 설정해줄 수 있고.
- 실습에서는 세션저장소에 세션id와 회원객체를 그대로 맵핑해뒀는데, 보통 세션저장소는 db에 두는게 아니라 메모리에 두기에 메모리 관리도 해줘야돼. 때문에 회원객체를 그대로 두는게 아니라, 정말 딱 로그인에 필요한 자주 쓰는 필드만 담은 객체를 핏하게 따로 두는게 좋긴함.
- 직접 세션값을 만들고 저장소를 만들고 HttpSevletRequest랑 HttpSevletResponse로 addCookie 해줘서 주고받고 하면 되는데, 
- @SessionAttribute(name = "sessionId", required = false) Member loginMember 이런식으로 인자를 받으면 세션 다 뒤져서 있으면 loginMember에 넣어주고 없으면 null값 넣어줌.
- DTO
  - Data Transfer Object: 계층간 데이터 전송을 위해 도메인 모델 대신 사용되는 객체
  - 순수하게 데이터를 저장. Getter, Setter가 있고 비즈니스 로직은 있어서는 안돼.
  - JPA 로드맵 api2편에서 본격적으로 나옴
- DAO
  - Data Access Object: 실제 DB에 접근하는 객체. Repository와 비슷한 기능과 역할을 하는듯. 정확히는 모르겠네.
- @NotNull vs @NotBlank vs @NotEmpty
  - @NotNull: null만 허용하지 않음. "" 나 " "는 허용.
  - @NotEmpty: null과 "" 둘다 허용하지 않음. " "는 허용
  - @NotBlank: null, "", " " 모두 허용하지 않음. 가장 강도가 높네


#### 로그인 처리2 : 필터, 인터셉터
- 페이지마다 로그인 사용자만 접근 가능 하도록 만들려면, controller의 모든 핸들러 위에다가 로그인 여부를 체크하는 로직을 넣고, 로그인이 안되어있으면 리다이렉트 보내면 되겠지.
- 그러나 모든 핸들러마다 다 이걸 넣으면 너무 지져분해지겠지.
- 이는 공통관심사 로직이니 AOP로 처리할 수도 있지만, 필터와 인터셉터라는 더 강력한 기능이 있음
- 서블릿 필터는 서블릿에서 제공하는 기능, 스프링 인터셉터는 Spring MVC가 제공하는 기능. 필터랑 인터셉터라는 개념은 다른 프레임워크나 뭐 다 사용하는 범용적인 개념
- 필터
  - 사실 WAS는 바로 서블릿을 호출하는게 아니라, 서블릿을 호출하기전 필터를 거치게 된다. 여기서 말하는 서블릿은 디스팻쳐서블릿.
  - http요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
  - 때문에 이 필터를 등록해주고 여기에서 로그인 여부를 체크하는 로직을 넣으면 됨.
  - 로그인 안되어있는 사용자가 허가 되지 않은 페이지에 url 찍고 들어왔을때 로그인페이지로 리다이렉트 보냈을때, 쿼리로 처음 접근 url을 기억해뒀다가 로그인 후 해당 url로 보내주면 사용성이 좋겠지.
  - @Configuration 클래스에 @Bean 등록해서 필터 등록
- 인터셉터
  - http요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
  - spring mvc의 시작은 서블릿. 디스패처서블릿이야. 따라서 인터셉터도 서블릿 다음으로 오겠지.
  - preHandle, postHandle, afterCompletion이 있는데, 
  - preHandle : 핸들러어댑터 호출 직전에 호출됨. 예외 있으면 핸들러어댑터 호출 안하고 끝냄
  - postHandle: 핸들러어댑터 다 처리되고 호출됨. 핸들러어댑터(핸들러)에서 예외 터지면 호출 안됨
  - afterCompletion: 뷰 랜더링 마치고 http 응답 보내고 나서 호출됨. 단 핸들러에서 예외 터져도 호출됨. 어떤 예외가 발생했는지 로그로 찍어볼 수 있음
  - spring mvc 전용이기에 필터보다 인터셉터가 여러모로 더 좋아
  - @Configuration 클래스에 implements WebMvcConfigurer 구현해서 등록..
- 스프링 인터셉터 기본 흐름 <br> <img width="819" alt="스크린샷 2023-04-06 오전 9 19 16" src="https://user-images.githubusercontent.com/93418349/230241942-723d30a6-f9f2-4ddb-be58-56a8931a3836.png">
- 다시 정리 
  - 로그인 처리는 '쿠키, 세션, 필터, 인터셉터' 이 네가지의 개념이 필요.
  - 나는 로그인 한 사용자입니다 라는것을 인증해줄 수 있는 기능이 쿠키.
  - 쿠키에 중요 정보를 그대로 가지고 다니면 안되니까 세션id를 쿠키에 넣어 가지고 다니고 서버 세션 저장소(메모리)에 id값과 중요정보를 저장해둠.
  - 페이지 이동할때마다 로그인 사용자인가 아닌가를 판단해야되는데 그게 필터와 인터셉터.
  - 즉, 적용된 페이지에 모든 요청때마다 필터와 인터셉터를 거쳐서 알맞은 로그인 사용자인지를 확인하는거야.
  - 필터는 was 와 서블릿 사이에서, 인터셉터는 서블릿과 컨트롤러 사이에서.
  - 인터셉터가 페이지를 include, exclude 하기 더 편하고 이래저래 강력하기에 필터보다는 얘를 더 많이 씀
  - 더 요약하면, 쿠키는 여권, 세션id는 여권 번호, 필터와 인터셉터는 출입국심사대 같은거야.


#### 예외 처리와 오류 페이지
- try catch로 중간에 에러를 잡아주지 않으면 핸들러에서 WAS까지 에러가 돌라감
- 서블릿 예외 처리 2가지
  - throw new RuntimeExcepion(); : 500 에러 페이지 보여줌
  - response.sendError(에러코드) : 입력한 에러코드에 맞는 에러 페이지 보여줌.
- 서블릿 컨테이너가 제공하는 기본 오류 페이지는 너무 구림. 서비스 망한것처럼 보임.
- WAS까지 올라온 에러는 해당 에러에 맞는 등록된 페이지가 있는지 확인하고, 없으면 기본 그 오류 페이지 보여주고, 커스텀한 페이지가 등록되어있다면, 그 페이지를 다시 부르게 됨.
- 마치 http 요청이 온것처럼 was서부터 핸들러까지 쭉 해당 페이지를 부르러 호출하며 내려감. 물론 실제로 http 요청은 아니고 클라이언트는 모르게 서버내부에서만 진행되는 프로세스임.
- 다만, 다시 핸들러까지 호출하러 내려갈때, 필터와 인터셉터는 또 호출할 필요 없긴함.
- 필터는 DispatchType 으로 중복 호출 제거 ( dispatchType=REQUEST )
- 인터셉터는 경로 정보로 중복 호출 제거( excludePathPatterns("/error-page/**") )
- 전체 흐름
  - 1_ WAS(/error-ex, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
  - 2_ WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
  - 3_ WAS 오류 페이지 확인
  - 4_ WAS(/error-page/500, dispatchType=ERROR) -> 필터(x) -> 서블릿 -> 인터셉터(x) -> 컨트롤러(/error-page/500) -> View
- 서블릿에서 처리하는건 너무너무 번거로워. 에러마다 오류페이지 등록하고 뭐하고...아이고.. 스프링 부트는 자동화 가능... 그니까 전체 흐름만 파악하고 전체 구현 방법은 스프리부트 활용한는것만 기억
- 이미 BasicErrorController에 기본 로직이 다 구현되어있어. 결국 아래 처리순서에 따라 페이지만 만들면 알아서 컨트롤러가 저 페이지 부르고 다 함.
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
- 진짜 이 단원은 설명이 존나게 길었지만 결론은 그냥 에러페이지 /error/xxx.html,,,등록하면 돼...


#### API 예외 처리
- 뷰를 보내는게 아니라 api를 보내야할 경우, 에러가 발생하면 이전에 설정해둔 그 오류 페이지가 전달되면 안되겠지.
- 어..이것도 BasicErrorController가 알아서,, json으로 요청 보내면 json으로, html 요청 보내면 html 오류페이지 알아서 다 해주네..
- @Requestmapping(produces=MediaType..) 미디어타입을 json으로 한거, text로 한거.. 같은 url에 다른 미디어타입을 매핑해두어서 그에 맞는 타입을 반환해주는 원리.
- 그러나, api 오류처리는 html 페이지 보여주는거랑 다르게 더 세밀하게 처리해줘야돼. api 전송하는 곳마다 서로 약속한 규약이 다를 수도 있고, 도메인마다 전달해야될것들도 다르기 때문에.
- 때문에, BasicErrorController로는 오류 페이지 뿌리는것만 하고 api는 ExceptionResolver를 활용.
- ExceptionResolver <br> <img width="814" alt="스크린샷 2023-04-05 오후 8 29 51" src="https://user-images.githubusercontent.com/93418349/230067794-84e5e2d3-c3b8-49bc-8c33-aa5c73ee42be.png">
- ExceptionResolver를 사용하면 컨트롤러에서 예외가 발생해도 ExceptionResolver 에서 예외를 처리해버린다. 따라서 예외가 발생해도 서블릿 컨테이너까지 예외가 전달되지 않고, 스프링 MVC에서 예외 처리는 끝이난다. 결과적으로 WAS 입장에서는 정상 처리가 된 것이다. 이렇게 예외를 이곳에서 모두 처리할 수 있다는 것이 핵심이다. 서블릿 컨테이너까지 예외가 올라가면 복잡하고 지저분하게 추가 프로세스가 실행된다. 반면에 ExceptionResolver 를 사용하면 예외처리가 상당히 깔끔해진다. 그런데 직접 ExceptionResolver 를 구현하려고 하니 상당히 복잡하다. 지금부터 스프링이 제공하는 ExceptionResolver 들을 알아보자.
- 정리하면, 예외 상황일때 오류 페이지 보낼꺼면 BasicErrorController, api 보낼거면 ExceptionResolver.
- 그리고 ExceptionResolver는 예외 발생시 WAS까지 다시 왔다 갔다 하는게 아니라 위에 이미지처럼 중간에 잘 해결해줌.

#### 스프링 타입 컨버터

#### 파일 업로드
