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

```
@{/hello/{param1}(param1=${param1}, param2=${param2})}
그 결과 /hello/data1?param2=data2
```
- 리터럴
  - 문자는 기본적으로 ' ' 으로 감싸야함.
  - 그러나 공백없는 한 단어는 ' ' 생략 가능
  - th:text="hello"
  - th:text-"'hello World!'"

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
- 나머지 아주 다양한것들이 있는데, 필요하면 그냥 레퍼런스 문서 보거나, 챕터 마지막 정리 강의 보거나 하자

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
- 스프링부트에서 해당 값을 가져오는 MessageSource 만들어줌. 그냥 Autowired만 해주면 됨.
- 국제화: message.properties를 국가별로 다르게 관리해서 http accept-lang 이나 locale에 따라 그에 맞는것을 보내주면 변경되겠지. 
- ms.getMessage(code, args, defaultMessage, locale) 호출하면, locale에 맞는 message.properites 찾아서 code에 매핑된 문자열 벨류값 반환함
- Test 코드 예외처리
  - Assertions.assertThatThrownBy( () -> function()).isIstanceOf(예외.class);


#### 검증1 : Validation

#### 검증2 : Bean Validation

#### 로그인 처리1 : 쿠키, 세션

#### 로그인 처리2 : 필터, 인터셉터

#### 예외 처리와 오류 페이지

#### API 예외 처리

#### 스프링 타입 컨버터

#### 파일 업로드
