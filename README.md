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




#### 예외 처리와 오류 페이지

#### API 예외 처리

#### 스프링 타입 컨버터

#### 파일 업로드
