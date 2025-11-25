# 6. 스프링 MVC - 기본 기능

## 로깅

- 로그 사용 방법
    1. Slf4j 사용
    2. Logger 라이브러리 사용

        ```java
        package hello.springmvc.basic;
        
        import lombok.extern.slf4j.Slf4j;
        import org.slf4j.Logger;
        import org.slf4j.LoggerFactory;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RestController;
        
        //@Slf4j
        @RestController
        public class LogTestController {
        
            private final Logger log = LoggerFactory.getLogger(getClass());
        
            @RequestMapping("/log-test")
            public String logTest() {
                String name = "Spring";
        
                log.trace("trace log={}", name);
                log.debug("debug log={}", name);
                log.info(" info log={}", name);
                log.warn(" warn log={}", name);
                log.error("error log={}", name);
        
                // 로그를 사용하지 않아도 a+b 계산 로직이 먼저 실행됨, 이런 방식으로 사용하면 X
                log.debug("String concat log=" + name);
        
                return "ok";
            }
        }
        ```


## 요청 매핑

- `@Controller` 와 `RestController` 의 차이
    - @Controller 는 반환 값이 String 이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.
    - @RestController 는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다.

- `@RequestMapping`에서 PathVariable는?

    ```java
    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data) {
     log.info("mappingPath userId={}", data);
     return "ok";
    }
    ```

    - @RequestMapping 은 URL 경로를 템플릿화 할 수 있는데, @PathVariable 을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.
    - @PathVariable 의 이름과 파라미터 이름이 같으면 생략할 수 있다.
        - @PathVariable("userId") String userId -> @PathVariable String userId
    - 다중으로도 사용 가능하다.

        ```java
        /**
         * PathVariable 사용 다중
         */
        @GetMapping("/mapping/users/{userId}/orders/{orderId}")
        public String mappingPath(@PathVariable String userId, @PathVariable Long
        orderId) {
         log.info("mappingPath userId={}, orderId={}", userId, orderId);
         return "ok";
        }
        ```

- 특정 파라미터 조건 매핑
    - 특정 파라미터가 있거나 없는 조건을 추가할 수 있다. 잘 사용하지는 않는다.

    ```java
    /**
    - 파라미터로 추가 매핑
    - params="mode",
    - params="!mode"
    - params="mode=debug"
    - params="mode!=debug" (! = )
    - params = {"mode=debug","data=good"}
    */
    @GetMapping(value = "/mapping-param", params = "mode=debug")
    public String mappingParam() {
    	[log.info](http://log.info/)("mappingParam");
    	return "ok";
    }
    ```

    - `@GetMapping(value = "/mapping-param", params = "mode=debug")`
        - 요청 URL에 `mode=debug`라는 **파라미터 조건**이 있어야만 이 메서드가 실행돼요.
        - 즉 단순히 `http://localhost:8080/mapping-param`을 호출하면 매핑되지 않고,
        - `http://localhost:8080/mapping-param?mode=debug`처럼 `mode=debug` 조건이 있어야 실행됩니다.

  스프링은 `params` 속성으로 다음과 같은 조건을 지정할 수 있어요:

    1. `params="mode"`
        - `mode`라는 파라미터가 **있으면** 매핑됨.
        - 예: `/mapping-param?mode=abc` → 매핑됨.
    2. `params="!mode"`
        - `mode`라는 파라미터가 **없으면** 매핑됨.
        - 예: `/mapping-param` (O), `/mapping-param?mode=abc` (X).
    3. `params="mode=debug"`
        - `mode`라는 파라미터가 있고 값이 `debug`일 때만 매핑됨.
        - 예: `/mapping-param?mode=debug` (O), `/mapping-param?mode=test` (X).
    4. `params="mode!=debug"`
        - `mode` 파라미터 값이 `debug`가 아닐 때 매핑됨.
        - 예: `/mapping-param?mode=test` (O), `/mapping-param?mode=debug` (X).
    5. `params = {"mode=debug","data=good"}`
        - `mode=debug` **그리고 동시에** `data=good`일 때만 매핑됨.
        - 예: `/mapping-param?mode=debug&data=good` (O).
- 특정 헤더 조건 매핑

    ```java
    /**
    
    - 특정 헤더로 추가 매핑
    - headers="mode",
    - headers="!mode"
    - headers="mode=debug"
    - headers="mode!=debug" (! = )
    */
    @GetMapping(value = "/mapping-header", headers = "mode=debug")
    public String mappingHeader() {
    	[log.info](http://log.info/)("mappingHeader");
    	return "ok";
    }
    ```

    - 파라미터 매핑과 비슷하지만, HTTP 헤더를 사용한다.
    - Postman으로 테스트 해야 한다.
- 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume

    ```java
    /**
    
    - Content-Type 헤더 기반 추가 매핑 Media Type
    - consumes="application/json"
    - consumes="!application/json"
    - consumes="application/*"
    - consumes="*\/*"
    - MediaType.APPLICATION_JSON_VALUE
    */
    @PostMapping(value = "/mapping-consume", consumes = "application/json")
    public String mappingConsumes() {
    	[log.info](http://log.info/)("mappingConsumes");
    	return "ok";
    }
    ```

    - Postman으로 테스트 해야 한다.
    - HTTP 요청의 **Content-Type** 헤더를 기반으로 미디어 타입으로 매핑한다.
    - 만약 맞지 않으면 HTTP 415 상태코드(Unsupported Media Type)을 반환한다.
    - 예시) consumes
        - consumes = "text/plain"
          consumes = {"text/plain", "application/*"}
          consumes = MediaType.TEXT_PLAIN_VALUE
- 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce

    ```java
    /**
    - Accept 헤더 기반 Media Type
    - produces = "text/html"
    - produces = "!text/html"
    - produces = "text/*"
    - produces = "*\/*"
    */
    @PostMapping(value = "/mapping-produce", produces = "text/html")
    public String mappingProduces() {
    	[log.info](http://log.info/)("mappingProduces");
    	return "ok";
    }
    ```

    - HTTP 요청의 **Accept** 헤더를 기반으로 미디어 타입으로 매핑한다.
      만약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다.
    - 예시)
      produces = "text/plain"
      produces = {"text/plain", "application/*"}
      produces = MediaType.TEXT_PLAIN_VALUE
      produces = "text/plain;charset=UTF-8"
- Content-Type vs Accept

  ### **Content-Type 헤더**

    - **의미**: 클라이언트가 서버에 보내는 **HTTP 요청 본문(body)의 데이터 형식**을 알려줍니다.
    - **용도**: 서버가 들어온 데이터를 어떤 방식으로 파싱해야 하는지 결정할 수 있게 해줍니다.
    - **예시**:
        - `Content-Type: application/json` → 본문이 JSON 형식임을 의미
        - `Content-Type: application/x-www-form-urlencoded` → HTML Form 형식의 데이터
        - `Content-Type: multipart/form-data` → 파일 업로드 등 멀티파트 형식 데이터

    ---

  ### **Accept 헤더**

    - **의미**: 클라이언트가 **서버로부터 어떤 형식의 응답을 원하는지**를 알려줍니다.
    - **용도**: 서버가 여러 가지 응답 형식을 지원할 경우, 클라이언트가 지정한 형식을 우선적으로 선택할 수 있게 합니다.
    - **예시**:
        - `Accept: application/json` → JSON 형식의 응답을 원함
        - `Accept: text/html` → HTML 페이지 응답을 원함
        - `Accept: */*` → 아무 형식이나 받아들일 수 있음

## HTTP 요청 - 기본, 헤더 조회

- HTTP 헤더 정보를 조회하는 방법
    - `@RequestHeader MultiValueMap<String, String> headerMap`
        - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
    - `@RequestHeader("host") String host`
        - 특정 HTTP 헤더를 조회한다.
        - 필수 값 여부: required
          기본 값 속성: defaultValue
- `MultiValueMap`
    - MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
    - HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
    - 예) keyA=value1&keyA=value2
- 특정 쿠키 조회하는 방법
    - `@CookieValue(value = "myCookie", required = false) String cookie`
    - 특정 쿠키를 조회한다.
    - 필수 값 여부: required
      기본 값: defaultValue
    - `required=true` (기본값): 쿠키가 없으면 예외 발생
    - `required=false`: 쿠키가 없으면 `null`로 처리

## HTTP 요청 파라미터

- 요청 파라미터 조회란?
    - HttpServletRequest 의 request.getParameter() 를 사용하면 다음 두가지 요청 파라미터를 조회할 수 있다.
    - GET, 쿼리 파라미터 전송
        - 예시
          http://localhost:8080/request-param?username=hello&age=20
    - POST, HTML Form 전송
        - 예시

            ```
            POST /request-param ...
            content-type: application/x-www-form-urlencoded
            
            username=hello&age=20
            ```

    - GET 쿼리 파리미터 전송 방식이든, POST HTML Form 전송 방식이든 둘다 형식이 같으므로 구분 없이 조회할 수 있다. 이것을 간단히 요청 파라미터(request parameter) 조회라 한다

### 방법1: 쿼리 파라미터, HTML Form에서 요청 파라미터 조회하는 예시 코드

- RequestParamController

    ```java
    package hello.springmvc.basic.request;
    
    import hello.springmvc.basic.HelloData;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.*;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.util.Map;
    
    @Slf4j
    @Controller
    public class RequestParamController {
    
        /**
         * 반환 타입이 없으면서 이렇게 응답에 값을 직접 집어넣으면, view 조회 X
         */
        @RequestMapping("/request-param-v1")
        public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
            String username = request.getParameter("username");
            int age = Integer.parseInt(request.getParameter("age"));
    
            log.info("username={}, age={}", username, age);
            response.getWriter().write("ok");
        }
    }
    ```

    - `request.getParameter()`
        - 여기서는 단순히 HttpServletRequest가 제공하는 방식으로 요청 파라미터를 조회했다.
- GET 실행

  http://localhost:8080/request-param-v1?username=hello&age=20

- Post Form 페이지 생성
    - 리소스는 /resources/static 아래에 두면 스프링 부트가 자동으로 인식한다.
    - main/resources/static/basic/hello-form.html

        ```html
        <!DOCTYPE html>
        <html lang="ko">
        <head>
            <meta charset="UTF-8">
            <title>Request Param Test</title>
        </head>
        <body>
            <form action="/request-param-v1" method="post">
                <label>
                    username:
                    <input type="text" name="username" />
                </label>
                <br><br>
                <label>
                    age:
                    <input type="text" name="age" />
                </label>
                <br><br>
                <button type="submit">전송</button>
            </form>
        </body>
        </html>
        ```

    - Post Form 실행

      http://localhost:8080/basic/hello-form.html


### 방법2:  `@RequestParam`

- 스프링이 제공하는 @RequestParam 을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.
- version 1. HTTP 파라미터 이름이 변수 이름과 다를 때

    ```java
    @ResponseBody
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge) {
    
        log.info("username={}, age={}", memberName, memberAge);
        return "ok";
    }
    ```

- @RequestParam
    - 파라미터 이름으로 바인딩
    - @RequestParam의 name(value) 속성이 파라미터 이름으로 사용
    - 예) @RequestParam("username") String memberName
      → request.getParameter("usernam
- @ResponseBody
    - View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
- version 2. HTTP 파라미터 이름이 변수 이름과 같을 때

    ```java
    @ResponseBody
    @RequestMapping("/request-param-v3")
    public String requestParamV3(
     @RequestParam String username,
     @RequestParam int age) {
    	 log.info("username={}, age={}", username, age);
    	 return "ok";
    }
    ```

    - HTTP 파라미터 이름이 변수 이름과 같으면 `@RequestParam(name="xx")` 생략 가능
- version 3.  String, int 등의 단순 타입일 때 (사용하지 말기)

    ```java
    @ResponseBody
    @RequestMapping("/request-param-v4")
    public String requestParamV4(String username, int age) {
     log.info("username={}, age={}", username, age);
     return "ok";
    }
    ```

    - String , int , Integer 등의 단순 타입이면 @RequestParam 도 생략 가능
    - @RequestParam 애노테이션을 생략하면 스프링 MVC는 내부에서 required=false 를 적용한다.
    - 이렇게 애노테이션을 완전히 생략해도 되는데, 너무 없는 것도 약간 과하다는 주관적 생각이 있다. @RequestParam 이 있으면 명확하게 요청 파리미터에서 데이터를 읽는 다는 것을 알 수 있다.
    - 스프링 부트 3.2부터 자바 컴파일러에 -parameters 옵션을 넣어주어야 애노테이션에 적는 이름을 생략할 수 있다.
- 파라미터 필수 여부 - requestParamRequired

    ```java
    @ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
     @RequestParam(required = true) String username,
     @RequestParam(required = false) Integer age) {
    	 log.info("username={}, age={}", username, age);
    	 return "ok";
    }
    ```

    - `@RequestParam.required`
        - 파라미터 필수 여부
        - 기본값이 파라미터 필수( true )이다.
    - /request-param-required 요청
        - username 이 없으므로 400 예외가 발생한다
    - 주의! - 파라미터 이름만 사용
        - `/request-param-required?username=`
        - 파라미터 이름만 있고 값이 없는 경우 빈문자로 통과
    - 주의! - 기본형(primitive)에 null 입력
        - /request-param 요청
        - **`@RequestParam(required = false) int age`**
        - **null 을 int 에 입력하는 것은 불가능(500 예외 발생)**
        - **따라서 null 을 받을 수 있는 Integer 로 변경하거나, 또는 다음에 나오는 defaultValue 사용**
- 기본 값 적용 - requestParamDefault

    ```java
    @ResponseBody
    @RequestMapping("/request-param-default")
    public String requestParamDefault(
     @RequestParam(required = true, defaultValue = "guest") String username,
     @RequestParam(required = false, defaultValue = "-1") int age) {
    	 log.info("username={}, age={}", username, age);
    	 return "ok";
    }
    ```

    - 파라미터에 값이 없는 경우 defaultValue 를 사용하면 기본 값을 적용할 수 있다.
    - 이미 기본 값이 있기 때문에 required 는 의미가 없다.
    - defaultValue 는 빈 문자의 경우에도 설정한 기본 값이 적용된다.
      /request-param-default?username=
- 파라미터를 Map으로 조회하기 - requestParamMap

    ```java
    @ResponseBody
    @RequestMapping("/request-param-map")
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
     log.info("username={}, age={}", paramMap.get("username"),paramMap.get("age"));
     return "ok";
    }
    ```

    - 파라미터를 Map, MultiValueMap으로 조회할 수 있다.
        - @RequestParam Map
            - Map(key=value)
        - @RequestParam MultiValueMap
            - MultiValueMap(key=[value1, value2, ...] ex) (key=userIds, value=[id1, id2])
    - 파라미터의 값이 1개가 확실하다면 Map 을 사용해도 되지만, 그렇지 않다면 MultiValueMap 을 사용하자.

### 방법3 : `@ModelAttribute`

- `@ModelAttribute` 가 나오게 된 배경
    - 실제 개발을 하면 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주어야 한다. 보통 다음과 같이 코드를 작성할 것이다.

    ```java
    @RequestParam String username;
    @RequestParam int age;
    
    HelloData data = new HelloData();
    data.setUsername(username);
    data.setAge(age);
    ```

    - 스프링은 이 과정을 완전히 자동화해주는 @ModelAttribute 기능을 제공한다.
- 이를 사용하려면 먼저 요청 파라미터를 바인딩 받을 객체를 만들어야 한다.

    ```java
    package hello.springmvc.basic;
    import lombok.Data;
    @Data
    public class HelloData {
     private String username;
     private int age;
    }
    ```

    - 롬복 @Data
        - @Getter , @Setter , @ToString , @EqualsAndHashCode , @RequiredArgsConstructor 를 자동으로 적용해준다
- @ModelAttribute 적용 - modelAttributeV1 (생략X)

    ```java
    /**
     * @ModelAttribute 사용
     * 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨, 뒤에 model을 설명할 때 자세히
    설명
     */
    @ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData) {
     log.info("username={}, age={}", helloData.getUsername(),
    helloData.getAge());
     return "ok";
    }
    ```

- `@ModelAttribute` 동작 과정
    - 스프링MVC는 @ModelAttribute 가 있으면 다음을 실행한다.
        - HelloData 객체를 생성한다.
        - 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다.
        - 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다.
        - 예) 파라미터 이름이 username 이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다.
- 프로퍼티
    - 객체에 getUsername() , setUsername() 메서드가 있으면, 이 객체는 username 이라는 프로퍼티를 가지고 있다.
    - username 프로퍼티의 값을 변경하면 setUsername() 이 호출되고, 조회하면 getUsername() 이 호출된다.
- 바인딩 오류
    - age=abc 처럼 숫자가 들어가야 할 곳에 문자를 넣으면 BindException 이 발생한다. 이런 바인딩 오류를 처리하는 방법은 검증 부분에서 다룬다.
- @ModelAttribute 적용 - modelAttributeV2 (생략O)

    ```java
    @ResponseBody
    @RequestMapping("/model-attribute-v2")
    public String modelAttributeV2(HelloData helloData) {
     log.info("username={}, age={}", helloData.getUsername(),helloData.getAge());
     return "ok";
    }
    ```

    - @ModelAttribute 는 생략할 수 있다.
      그런데 @RequestParam 도 생략할 수 있으니 혼란이 발생할 수 있다.
    - 스프링은 해당 생략시 다음과 같은 규칙을 적용한다.
      String , int , Integer 같은 단순 타입 = @RequestParam
      나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)

## HTTP 요청 메시지

### HTTP 요청 메시지 - 단순 텍스트

- 요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 @RequestParam, @ModelAttribute 를 사용할 수 없다.
- RequestBodyStringController

    ```java
    @Slf4j
    @Controller
    public class RequestBodyStringController {
    
        @PostMapping("/request-body-string-v1")
        public void requestBodyString(HttpServletRequest request,
                                      HttpServletResponse response) throws IOException {
            ServletInputStream inputStream = request.getInputStream();
            String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    
            log.info("messageBody={}", messageBody);
    
            response.getWriter().write("ok");
        }
    }
    ```

    - 이 경우 HTTP 메시지 바디의 데이터를 InputStream 을 사용해서 직접 읽을 수 있다.
- Input, Output 스트림, Reader - requestBodyStringV2

    ```java
    @PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer responseWriter)
    throws IOException {
     String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
     log.info("messageBody={}", messageBody);
     responseWriter.write("ok");
    }
    ```

    - 스프링 MVC는 지원하는 InputStream(Reader), OutputStream(Writer)의 역할은?
        - InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
        - OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
- HttpEntity - requestBodyStringV3

    ```java
    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
     String messageBody = httpEntity.getBody();
     log.info("messageBody={}", messageBody);
     return new HttpEntity<>("ok");
    }
    ```

    - 스프링 MVC는 다음 파라미터를 지원한다.
    - HttpEntity: HTTP header, body 정보를 편리하게 조회
        - 메시지 바디 정보를 직접 조회
        - 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X
    - HttpEntity는 응답에도 사용 가능
        - 메시지 바디 정보 직접 반환
        - 헤더 정보 포함 가능
        - view 조회X
    - HttpEntity 를 상속받은 다음 객체들도 같은 기능을 제공한다.
        - RequestEntity
            - HttpMethod, url 정보가 추가, 요청에서 사용
        - ResponseEntity
            - HTTP 상태 코드 설정 가능, 응답에서 사용
            - `return new ResponseEntity<String>("Hello World", responseHeaders,
            HttpStatus.CREATED)`
- @RequestBody - requestBodyStringV4

    ```java
    @ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) {
     log.info("messageBody={}", messageBody);
     return "ok";
    }
    ```

    - @RequestBody
        - @RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 참고로 헤더 정보가 필요하다면 HttpEntity 를 사용하거나 @RequestHeader 를 사용하면 된다.
        - 이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam , @ModelAttribute 와는 전혀 관계가 없다.
- HttpEntity vs @RequestBody


    | 구분 | `HttpEntity<T>` | `@RequestBody` |
    | --- | --- | --- |
    | 접근 가능 | 요청 **헤더 + 본문** | 요청 **본문(body)만** |
    | 응답 활용 | `ResponseEntity`로 상태 코드/헤더 제어 가능 | `@ResponseBody`와 함께 주로 사용 |
    | JSON 변환 | 가능 (`HttpEntity<HelloData>`) | 더 많이 사용됨 (`@RequestBody HelloData`) |
    | 주 사용 목적 | HTTP 메시지를 전체적으로 다룰 때 | 본문만 간단하게 꺼내고 싶을 때 |
- 요청 파라미터 vs HTTP 메시지 바디
    - 요청 파라미터를 조회하는 기능: @RequestParam , @ModelAttribute
    - HTTP 메시지 바디를 직접 조회하는 기능: @RequestBody
    - @ResponseBody 를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다.
      물론 이 경우에도 view를 사용하지 않는다.

### HTTP 요청 메시지 - JSON

- RequestBodyJsonController

    ```java
    package hello.springmvc.basic.request;
    
    import com.fasterxml.jackson.databind.ObjectMapper;
    import hello.springmvc.basic.HelloData;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.stereotype.Controller;
    import org.springframework.util.StreamUtils;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.ResponseBody;
    
    import javax.servlet.ServletInputStream;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.nio.charset.StandardCharsets;
    
    /**
     * {"username":"hello", "age":20}
     * content-type: application/json
     */
    @Slf4j
    @Controller
    public class RequestBodyJsonController {
    
        private final ObjectMapper objectMapper = new ObjectMapper();
    
        @PostMapping("/request-body-json-v1")
        public void requestBodyJsonV1(HttpServletRequest request,
                                      HttpServletResponse response) throws IOException {
            ServletInputStream inputStream = request.getInputStream();
            String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    
            log.info("messageBody={}", messageBody);
    
            HelloData data = objectMapper.readValue(messageBody, HelloData.class);
            log.info("username={}, age={}", data.getUsername(), data.getAge());
    
            response.getWriter().write("ok");
        }
    }
    
    ```

    - HttpServletRequest를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 문자로 변환한다.
    - 문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper 를 사용해서 자바 객체로 변환한다
- requestBodyJsonV2 - @RequestBody 문자 변환

    ```java
    @ResponseBody
    @PostMapping("/request-body-json-v2")
    public String requestBodyJsonV2(@RequestBody String messageBody) throws
    IOException {
     HelloData data = objectMapper.readValue(messageBody, HelloData.class);
     log.info("username={}, age={}", data.getUsername(), data.getAge());
     return "ok";
    }
    ```

    - 이전에 학습했던 @RequestBody 를 사용해서 HTTP 메시지에서 데이터를 꺼내고 messageBody에 저장한다.
    - 문자로 된 JSON 데이터인 messageBody 를 objectMapper 를 통해서 자바 객체로 변환한다
- requestBodyJsonV3 - @RequestBody 객체 변환

    ```java
    @ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData data) {
     log.info("username={}, age={}", data.getUsername(), data.getAge());
     return "ok";
    }
    ```

    - @RequestBody 객체 파라미터
        - @RequestBody HelloData data
        - @RequestBody 에 직접 만든 객체를 지정할 수 있다.
        - HttpEntity , @RequestBody 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
        - HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 우리가 방금 V2에서 했던 작업을 대신 처리해준다
    - @RequestBody는 생략 불가능
        - @ModelAttribute 에서 학습한 내용을 떠올려보자.
        - 스프링은 @ModelAttribute , @RequestParam 과 같은 해당 애노테이션을 생략시 다음과 같은 규칙을 적용한다.
            - String , int , Integer 같은 단순 타입 = @RequestParam
            - 나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)
        - 따라서 이 경우 HelloData에 @RequestBody 를 생략하면 @ModelAttribute 가 적용되어버린다.
        - HelloData data → @ModelAttribute HelloData data
        - 따라서 생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.
        - 주의
            - HTTP 요청시에 content-type이 application/json인지 꼭! 확인해야 한다. 그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.
    - 물론 앞서 배운 것과 같이 HttpEntity를 사용해도 된다.
      requestBodyJsonV4 - HttpEntity

        ```java
        @ResponseBody
        @PostMapping("/request-body-json-v4")
        public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
         HelloData data = httpEntity.getBody();
         log.info("username={}, age={}", data.getUsername(), data.getAge());
         return "ok";
        }
        ```

    - requestBodyJsonV5

        ```java
        @ResponseBody
        @PostMapping("/request-body-json-v5")
        public HelloData requestBodyJsonV5(@RequestBody HelloData data) {
         log.info("username={}, age={}", data.getUsername(), data.getAge());
         return data;
        }
        ```

        - @ResponseBody
            - 응답의 경우에도 @ResponseBody 를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.
              물론 이 경우에도 HttpEntity 를 사용해도 된다.
            - @RequestBody 요청
              JSON 요청 → HTTP 메시지 컨버터 → 객체
            - @ResponseBody 응답
              객체 → HTTP 메시지 컨버터 → JSON 응답


## HTTP 응답

### HTTP 응답 - 정적 리소스, 뷰 템플릿

### HTTP 응답- HTTP API, 메시지 바디에 직접 입력

```java
package hello.springmvc.basic.response;

import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Controller
//@RestController
public class ResponseBodyController {

    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
    }

    /**
     * HttpEntity, ResponseEntity(Http Status 추가)
     */
    @GetMapping("/response-body-string-v2")
    public ResponseEntity<String> responseBodyV2() {
        return new ResponseEntity<>("ok", HttpStatus.OK);
    }

    @ResponseBody
    @GetMapping("/response-body-string-v3")
    public String responseBodyV3() {
        return "ok";
    }

    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);

        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }

    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV2() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);

        return helloData;
    }
}
```

### responseBodyV1

- 서블릿을 직접 다룰 때처럼 `HttpServletResponse` 객체를 통해서 HTTP 메시지 바디에 직접 `ok` 응답 메시지를 전달한다.
- 코드 예시:

    ```java
    response.getWriter().write("ok");
    ```


### responseBodyV2

- `ResponseEntity` 엔티티는 `HttpEntity`를 상속받았는데, `HttpEntity`는 HTTP 메시지의 헤더와 바디 정보를 가진다.
- `ResponseEntity`는 여기에 더해서 **HTTP 응답 코드**를 설정할 수 있다.
- 예: `HttpStatus.CREATED`로 변경하면 `201` 응답이 나가는 것을 확인할 수 있다.

### responseBodyV3

- `@ResponseBody`를 사용하면 뷰(View)를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다.
- `ResponseEntity`도 동일한 방식으로 동작한다.

### responseBodyJsonV1

- `ResponseEntity`를 반환한다.
- HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어 반환된다.

### responseBodyJsonV2

- `ResponseEntity`는 HTTP 응답 코드를 설정할 수 있는데, `@ResponseBody`를 사용하면 이런 설정이 다소 까다롭다.
- `@ResponseStatus(HttpStatus.OK)` 애노테이션을 사용하면 **응답 코드**도 설정할 수 있다.
- 하지만 애노테이션이기 때문에 **응답 코드를 동적으로 변경할 수는 없다**.
- 프로그램 조건에 따라 동적으로 변경하려면 `ResponseEntity`를 사용해야 한다.

### @RestController

- `@Controller` 대신 `@RestController` 애노테이션을 사용하면, 해당 컨트롤러의 모든 메서드에 `@ResponseBody`가 적용된다.
- 따라서 뷰 템플릿을 사용하는 것이 아니라, **HTTP 메시지 바디에 직접 데이터**를 입력한다.
- 이름 그대로 **Rest API(HTTP API)**를 만들 때 사용하는 컨트롤러이다.
- 참고: `@ResponseBody`는 클래스 레벨에 두면 전체 메서드에 적용된다. `@RestController` 애노테이션 내부에 이미 `@ResponseBody`가 적용되어 있다.

## HTTP 메시지 컨버터

- @ResponseBody 사용 원리

  ![img_11.png](img_11.png)

    - @ResponseBody 를 사용
        - HTTP의 BODY에 문자 내용을 직접 반환
        - viewResolver 대신에 HttpMessageConverter 가 동작
        - 기본 문자처리: StringHttpMessageConverter
        - 기본 객체처리: MappingJackson2HttpMessageConverter
        - byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음
- 스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.
    - **HTTP 요청: @RequestBody , HttpEntity(RequestEntity)**
    - **HTTP 응답: @ResponseBody , HttpEntity(ResponseEntity)**
- HTTP 메시지 컨버터 인터페이스
  org.springframework.http.converter.HttpMessageConverter

    ```java
    package org.springframework.http.converter;
    
    public interface HttpMessageConverter<T> {
    
        boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
    
        boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
    
        List<MediaType> getSupportedMediaTypes();
    
        T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
                throws IOException, HttpMessageNotReadableException;
    
        void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
                throws IOException, HttpMessageNotWritableException;
    }
    ```

- HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다
    - canRead() , canWrite()

      메시지 컨버터가 해당 **클래스, 미디어타입**을 지원하는지 체크

    - read() , write()

      메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

- 스프링 부트 기본 메시지 컨버터 (일부 생략)
    - 0 = ByteArrayHttpMessageConverter
        - byte[] 데이터를 처리한다.
        - 클래스 타입: byte[] , 미디어타입: */* ,
        - 요청 예) @RequestBody byte[] data
          응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-stream
    - 1 = StringHttpMessageConverter
        - String 문자로 데이터를 처리한다.
        - 클래스 타입: String , 미디어타입: */*
        - 요청 예) @RequestBody String data
          응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain
    - 2 = MappingJackson2HttpMessageConverter
        - application/json
        - 클래스 타입: **객체 또는 HashMap , 미디어타입 application/json 관련**
        - 요청 예) @RequestBody HelloData data
          응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련
- HTTP 요청 데이터 읽기
    - HTTP 요청이 오고, 컨트롤러에서 @RequestBody , HttpEntity 파라미터를 사용한다.
    - 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead() 를 호출한다.
        - 대상 클래스 타입을 지원하는가.
        - 예) @RequestBody 의 대상 클래스 ( byte[] , String , HelloData )
    - HTTP 요청의 Content-Type 미디어 타입을 지원하는가.
        - 예) text/plain , application/json , */*
        - canRead() 조건을 만족하면 read() 를 호출해서 객체 생성하고, 반환한다.
- HTTP 응답 데이터 생성
    - 컨트롤러에서 @ResponseBody , HttpEntity 로 값이 반환된다.
    - 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite() 를 호출한다.
        - 대상 클래스 타입을 지원하는가.
            - 예) return의 대상 클래스 ( byte[] , String , HelloData )
        - HTTP 요청의 Accept 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping 의 produces )
            - 예) text/plain , application/json , */*
    - canWrite() 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

## 요청 매핑 헨들러 어뎁터 구조

- ArgumentResolver의 역할은?

    ![img_12.png](img_12.png)
    - HttpServletRequest , Model 은 물론이고, @RequestParam , @ModelAttribute 같은 애노테이션 그리고 @RequestBody , HttpEntity 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 보여주었다. 이렇게 **파라미터를 유연하게 처리**할 수 있는 이유가 바로 ArgumentResolver 덕분이다.
- ArgumentResolver
    - ArgumentResolver 의 supportsParameter() 를 호출해서 **해당 파라미터를 지원하는지 체크**하고,
    - 지원하면 resolveArgument() 를 호출해서 **실제 객체를 생성**한다.
    - 그리고 이렇게 생성된 객체가 **컨트롤러 호출시 넘어가는 것**이다.
- ReturnValueHandler의 역할
    - HandlerMethodReturnValueHandler 를 줄여서 ReturnValueHandler 라 부른다.
    - ArgumentResolver 와 비슷한데, 이것은 **응답 값을 변환하고 처리**한다.
    - **컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유**가 바로 ReturnValueHandler 덕분이다.

- HTTP 메시지 컨버터의 위치

  ![img_13.png](img_13.png)

    - 요청의 경우 @RequestBody 를 처리하는 ArgumentResolver 가 있고, HttpEntity 를 처리하는ArgumentResolver 가 있다. 이 ArgumentResolver 들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다.
    - 응답의 경우 @ResponseBody 와 HttpEntity 를 처리하는 ReturnValueHandler 가 있다. 그리고 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.
    - 스프링 MVC는 @RequestBody @ResponseBody 가 있으면
      RequestResponseBodyMethodProcessor(ArgumentResolver, ReturnValueHandler 둘다 구현)을 사용한다.
    - HttpEntity 가 있으면 HttpEntityMethodProcessor(ArgumentResolver, ReturnValueHandler 둘다 구현) 를 사용한다.
- 스프링은 다음을 모두 인터페이스로 제공한다. 따라서 필요하면 언제든지 기능을 확장할 수 있다
    - HandlerMethodArgumentResolver
    - HandlerMethodReturnValueHandler
    - HttpMessageConverter
- 기능 확장은 무엇을 상속 받아서 스프링 빈으로 등록하면 되는가?
    - WebMvcConfigurer

        ```java
        @Bean
        public WebMvcConfigurer webMvcConfigurer() {
            return new WebMvcConfigurer() {
                @Override
                public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
                    // ...
                }
        
                @Override
                public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                    // ...
                }
            };
        }
        
        ```