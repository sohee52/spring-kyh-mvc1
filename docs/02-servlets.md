# 2. 서블릿

## 스프링 부트 서블릿 환경 구성

- 서블릿을 직접 등록해서 사용할 수 있도록 스프링 부트가 지원하는 애노테이션은?
    - `@ServletComponentScan`
    - `@SpringBootApplication` 위에 적는다.
- 서블릿 등록하는 코드
    - 전체 코드

        ```java
        package hello.servlet.basic;
        
        import javax.servlet.ServletException;
        import javax.servlet.annotation.WebServlet;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        @WebServlet(name = "helloServlet", urlPatterns = "/hello")
        public class HelloServlet extends HttpServlet {
        
            @Override
            protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
                System.out.println("HelloServlet.service");
                System.out.println("request = " + request);
                System.out.println("response = " + response);
        
                String username = request.getParameter("username");
                System.out.println("username = " + username);
        
                response.setContentType("text/plain");
                response.setCharacterEncoding("utf-8");
                response.getWriter().write("hello " + username);
        
            }
        }
        ```

    - 서블릿 애노테이션
        - `@WebServlet(name = "helloServlet", urlPatterns = "/hello")`
        - name : 서블릿 이름
        - urlPatterns : URL 매핑
    - HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 어떤 메서드를 실행?
        - `protected void service(HttpServletRequest request, HttpServletResponse response)`
    - 웹 브라우저 http://localhost:8080/hello?username=world 실행 결과는?
        - 결과: hello world
    - 콘솔 실행 결과

        ```java
        HelloServlet.service
        request = org.apache.catalina.connector.RequestFacade@5e4e72
        response = org.apache.catalina.connector.ResponseFacade@37d112b6
        username = world
        ```


## HttpServletRequest

- HttpServletRequest 역할
    - HTTP 요청 메시지를 개발자가 직접 파싱해서 사용해도 되지만, 매우 불편할 것이다.
    - **서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱한다.**
    - **그리고 그 결과를 HttpServletRequest 객체에 담아서 제공한다.**
    - **이 덕분에 개발자는 Start-line, Header, Remote, Local 등의 정보를 편리하게 조회할 수 있다.**
    - HttpServletRequest 객체는 추가로 **여러가지 부가 기능**도 함께 제공한다.
        - **임시 저장소 기능**
            - 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능
            - 저장: `request.setAttribute(name, value)`
            - 조회: `request.getAttribute(name)`
        - **세션 관리 기능**
            - `request.getSession(create: true)`
- HttpServletRequest를 통해서 HTTP 메시지의 start-line, header 정보 조회 방법

    ```java
    package hello.servlet.basic.request;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.annotation.WebServlet;
    import jakarta.servlet.http.Cookie;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    
    @WebServlet(name = "requestHeaderServlet", urlPatterns = "/request-header")
    public class RequestHeaderServlet extends HttpServlet {
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            printStartLine(request);
            printHeaders(request);
            printHeaderUtils(request);
            printEtc(request);
        }
    
        //start line 정보
        private static void printStartLine(HttpServletRequest request) {
            System.out.println("--- REQUEST-LINE - start ---");
    
            System.out.println("request.getMethod() = " + request.getMethod()); //GET
            System.out.println("request.getProtocol() = " + request.getProtocol()); //HTTP/1.1
            System.out.println("request.getScheme() = " + request.getScheme()); //http
    
            // http://localhost:8080/request-header
            System.out.println("request.getRequestURL() = " + request.getRequestURL());
    
            // /request-header
            System.out.println("request.getRequestURI() = " + request.getRequestURI());
    
            //username=hi
            System.out.println("request.getQueryString() = " + request.getQueryString());
    
            System.out.println("request.isSecure() = " + request.isSecure()); //https 사용 유무
    
            System.out.println("--- REQUEST-LINE - end ---");
            System.out.println();
        }
    
        // header 정보
        private static void printHeaders(HttpServletRequest request) {
            System.out.println("--- Headers - start ---");
    /*
            Enumeration<String> headerNames = request.getHeaderNames();
            while (headerNames.hasMoreElements()) {
                String headerName = headerNames.nextElement();
                System.out.println(headerName + ": " + request.getHeader(headerName));
            }
    */
            request.getHeaderNames().asIterator()
                    .forEachRemaining(headerName -> System.out.println(headerName + ": " + request.getHeader(headerName)));
            System.out.println("--- Headers - end ---");
            System.out.println();
        }
    
        //Header 편리한 조회
        private void printHeaderUtils(HttpServletRequest request) {
            System.out.println("--- Header 편의 조회 start ---");
            System.out.println("[Host 편의 조회]");
            System.out.println("request.getServerName() = " + request.getServerName()); //Host 헤더
            System.out.println("request.getServerPort() = " + request.getServerPort()); //Host 헤더
            System.out.println();
            System.out.println("[Accept-Language 편의 조회]");
            request.getLocales().asIterator()
                    .forEachRemaining(locale -> System.out.println("locale = " + locale));
            System.out.println("request.getLocale() = " + request.getLocale());
            System.out.println();
            System.out.println("[cookie 편의 조회]");
            if (request.getCookies() != null) {
                for (Cookie cookie : request.getCookies()) {
                    System.out.println(cookie.getName() + ": " + cookie.getValue());
                }
            }
            System.out.println();
            System.out.println("[Content 편의 조회]");
            System.out.println("request.getContentType() = " + request.getContentType());
            System.out.println("request.getContentLength() = " + request.getContentLength());
            System.out.println("request.getCharacterEncoding() = " + request.getCharacterEncoding());
            System.out.println("--- Header 편의 조회 end ---");
            System.out.println();
        }
    
        //기타 정보
        private void printEtc(HttpServletRequest request) {
            System.out.println("--- 기타 조회 start ---");
            System.out.println("[Remote 정보]");
            System.out.println("request.getRemoteHost() = " + request.getRemoteHost()); //
            System.out.println("request.getRemoteAddr() = " + request.getRemoteAddr()); //
            System.out.println("request.getRemotePort() = " + request.getRemotePort()); //
            System.out.println();
            System.out.println("[Local 정보]");
            System.out.println("request.getLocalName() = " + request.getLocalName()); //
            System.out.println("request.getLocalAddr() = " + request.getLocalAddr()); //
            System.out.println("request.getLocalPort() = " + request.getLocalPort()); //
            System.out.println("--- 기타 조회 end ---");
            System.out.println();
        }
    }
    
    ```


## Http 요청 데이터

- HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법
    - **GET - 쿼리 파라미터**
        - 메시지 바디 없이, **URL의 쿼리 파라미터에 데이터를 포함해서 전달**
        - **검색, 필터, 페이징 등에서 많이 사용하는 방식**
        - 쿼리 파라미터는 URL에 다음과 같이 ? 를 시작으로 보낼 수 있다. 추가 파라미터는 & 로 구분하면 된다.
        - 예) http://localhost:8080/request-param?username=hello&age=20
    - **POST - HTML Form**
        - **content-type: `application/x-www-form-urlencoded`**
        - **메시지 바디에 쿼리 파리미터 형식으로 전달**
          message body: `username=hello&age=20`
        - **회원 가입, 상품 주문, HTML Form 사용**
    - **HTTP message body에 데이터를 직접 담아서 요청**
        - **HTTP API에서 주로 사용, JSON, XML, TEXT**
        - 데이터 형식은 주로 JSON 사용
        - POST, PUT, PATCH

### HTTP 요청 데이터 - GET 쿼리 파라미터

- 쿼리 파라미터 조회 메서드

    ```java
    String username = request.getParameter("username"); //단일 파라미터 조회
    Enumeration<String> parameterNames = request.getParameterNames(); //파라미터 이름들 모두 조회
    Map<String, String[]> parameterMap = request.getParameterMap(); //파라미터를 Map으로 조회
    String[] usernames = request.getParameterValues("username"); //복수 파라미터 조회
    ```

    ```java
    package hello.servlet.basic.request;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.annotation.WebServlet;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    
    @WebServlet(name = "requestParamServlet", urlPatterns = "/request-param")
    public class RequestParamServlet extends HttpServlet {
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            System.out.println("[전체 파라미터 조회] - start");
    /*
            Enumeration<String> parameterNames = request.getParameterNames();
            while (parameterNames.hasMoreElements()) {
                String paramName = parameterNames.nextElement();
                System.out.println(paramName + "=" + request.getParameter(paramName));
            }
    */
            request.getParameterNames().asIterator()
                    .forEachRemaining(paramName -> System.out.println(paramName + "=" + request.getParameter(paramName)));
            System.out.println("[전체 파라미터 조회] - end");
            System.out.println();
    
            System.out.println("[단일 파라미터 조회]");
            String username = request.getParameter("username");
            System.out.println("request.getParameter(username) = " + username);
            String age = request.getParameter("age");
            System.out.println("request.getParameter(age) = " + age);
            System.out.println();
    
            System.out.println("[이름이 같은 복수 파라미터 조회]");
            System.out.println("request.getParameterValues(username)");
            String[] usernames = request.getParameterValues("username");
            for (String name : usernames) {
                System.out.println("username=" + name);
            }
    
            response.getWriter().write("ok");
        }
    }
    
    ```

- 복수 파라미터에서 단일 파라미터 조회
    - `username=hello&username=kim` 과 같이 파라미터 이름은 하나인데, 값이 중복이면 어떻게 될까?
    - `request.getParameter()` 는 하나의 파라미터 이름에 대해서 단 하나의 값만 있을 때 사용해야 한다.
    - 지금처럼 중복일 때는 `request.getParameterValues()` 를 사용해야 한다.
    - 참고로 이렇게 중복일 때 `request.getParameter()` 를 사용하면 `request.getParameterValues()` 의 첫번째 값을 반환한다
- GET URL 쿼리 파라미터 형식의 content-type
    - HTTP 메시지 바디를 사용하지 않기 때문에 content-type이 없다.

### HTTP 요청 데이터 - POST HTML Form

- POST HTML Form 형식의 content-type
    - HTTP 메시지 바디에 데이터 포함해서 보내기 때문에 content-type을 꼭 지정해야 한다. 이렇게 폼으로 데이터를 전송하는 형식을 application/x-www-form-urlencoded 라 한다.
    - application/x-www-form-urlencoded 형식은 앞서 GET에서 살펴본 쿼리 파라미터 형식과 같다. 따라서 쿼
      리 파라미터 조회 메서드를 그대로 사용하면 된다
- GET URL 쿼리 파라미터 형식과 POST HTML Form 형식의 공통점
    - 두 방식에 차이가 있지만 서버 입장에서는 둘의 형식이 동일하므로 request.getParameter() 로 편리하게 구분없이 조회할 수 있다.
- Postman을 사용한 테스트 방법은? 주의사항은?
    - 간단한 테스트에 HTML form을 만들기 귀찮을 때 쓰는 방법
    - Postman 테스트 주의 사항
        - POST 전송 시
        - Body x-www-form-urlencoded 선택
        - Headers에서 content-type: application/x-www-form-urlencoded 로 지정된 부분 꼭 확인

### HTTP 요청 데이터 - API 메시지 바디

- 단순 텍스트 메시지를 HTTP 메시지 바디에 담아서 전송하고, 읽는 방법
    - HTTP 메시지 바디의 데이터를 InputStream을 사용해서 직접 읽을 수 있다.

    ```java
    @WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-body-string")
    public class RequestBodyStringServlet extends HttpServlet {
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            ServletInputStream inputStream = request.getInputStream();
            String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
            System.out.println("messageBody = " + messageBody);
            response.getWriter().write("ok");
        }
    }
    ```

- JSON 형식으로 데이터를 전달하는 방법
    - **InputStream과 ObjectMapper 라이브러리**를 통해 조회할 수 있다.

    ```java
    @WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-body-json")
    public class RequestBodyJsonServlet extends HttpServlet {
    
        private ObjectMapper objectMapper = new ObjectMapper();
    
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            ServletInputStream inputStream = request.getInputStream();
            String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
            System.out.println("messageBody = " + messageBody);
    
            HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
            System.out.println("helloData.getUsername() = " + helloData.getUsername());
            System.out.println("helloData.getAge() = " + helloData.getAge());
    
            response.getWriter().write("ok");
        }
    }
    ```

    - JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면
        - Jackson, Gson 같은 JSON 변환 라이브러리를 추가해서 사용해야 한다.
        - 스프링 부트로 Spring MVC를 선택하면 기본으로 Jackson 라이브러리( `ObjectMapper` )를 함께 제공한다.

## HttpServletResponse

- HttpServletResponse 역할
    - HTTP 응답 메시지 생성
        - HTTP 응답코드 지정
        - 헤더 생성
        - 바디 생성
    - 편의 기능 제공
        - Content-Type, 쿠키, Redirect
- HttpServletResponse 활용 코드

    ```java
    package hello.servlet.basic.response;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.annotation.WebServlet;
    import jakarta.servlet.http.Cookie;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    import java.io.PrintWriter;
    
    @WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
    public class ResponseHeaderServlet extends HttpServlet {
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            //[status-line]
            response.setStatus(HttpServletResponse.SC_OK); // 200
    
            //[response-header]
            response.setHeader("Content-Type", "text/plain;charset=utf-8");
            response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate");
            response.setHeader("Pragma", "no-cache");
            response.setHeader("My-Header", "hello");
    
            //[header 편의 메서드]
            content(response);
            cookie(response);
            redirect(response);
    
            //[message body]
            PrintWriter writer = response.getWriter();
            writer.println("안녕하세요");
        }
    
        private void content(HttpServletResponse response) {
            //Content-Type: text/plain;charset=utf-8
            //Content-Length: 2
            //response.setHeader("Content-Type", "text/plain;charset=utf-8");
            response.setContentType("text/plain");
            response.setCharacterEncoding("utf-8");
            //response.setContentLength(2); //(생략시 자동 생성)
        }
    
        private void cookie(HttpServletResponse response) {
            //Set-Cookie: myCookie=good; Max-Age=600;
            //response.setHeader("Set-Cookie", "myCookie=good; Max-Age=600");
            Cookie cookie = new Cookie("myCookie", "good");
            cookie.setMaxAge(600); //600초
            response.addCookie(cookie);
        }
    
        private void redirect(HttpServletResponse response) throws IOException {
            //Status Code 302
            //Location: /basic/hello-form.html
            //response.setStatus(HttpServletResponse.SC_FOUND); //302
            //response.setHeader("Location", "/basic/hello-form.html");
            response.sendRedirect("/basic/hello-form.html");
        }
    }
    
    ```


## Http 응답 데이터

- HTTP 응답 메시지는 주로 다음 내용을 담아서 전달한다.
    - 단순 텍스트 응답
        - `writer.println("ok");`
    - HTML 응답
    - HTTP API - MessageBody JSON 응답
- HTTP 응답으로 HTML을 반환하는 방법

    ```java
    package hello.servlet.basic.response;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.annotation.WebServlet;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    import java.io.PrintWriter;
    
    @WebServlet(name = "responseHtmlServlet", urlPatterns = "/response-html")
    public class responseHtmlServlet extends HttpServlet {
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            //Content-Type: text/html;charset=utf-8
            response.setContentType("text/html");
            response.setCharacterEncoding("utf-8");
    
            PrintWriter writer = response.getWriter();
            writer.println("<html>");
            writer.println("<body>");
            writer.println("  <div>안녕하세요</div>");
            writer.println("  <div>반갑습니다</div>");
            writer.println("</body>");
            writer.println("</html>");
        }
    }
    
    ```

- HTTP 응답으로 JSON을 반환하는 방법

    ```java
    package hello.servlet.basic.response;
    
    import com.fasterxml.jackson.databind.ObjectMapper;
    import hello.servlet.basic.HelloData;
    import jakarta.servlet.ServletException;
    import jakarta.servlet.annotation.WebServlet;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    
    @WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")
    public class ResponseJsonServlet extends HttpServlet {
        private ObjectMapper objectMapper = new ObjectMapper();
    
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            //Content-Type: application/json
            response.setContentType("application/json");
            response.setCharacterEncoding("utf-8");
    
            // 객체 생성
            HelloData helloData = new HelloData();
            helloData.setUsername("홍길동");
            helloData.setAge(30);
    
            // 객체를 JSON으로 변환
            //{"username":"kim","age":20}
            String result = objectMapper.writeValueAsString(helloData);
    
            // 응답으로 JSON 데이터 전송
            response.getWriter().write(result);
        }
    }
    ```

- HTTP 응답으로 JSON을 반환할 때 content-type은? 객체를 JSON 문자로 변경하는 방법은?
    - HTTP 응답으로 JSON을 반환할 때는 content-type을 `application/json` 로 지정해야 한다.
    - Jackson 라이브러리가 제공하는 `objectMapper.writeValueAsString()` 를 사용하면 객체를 JSON 문자로 변경할 수 있다
- `application/json` 사용 시 주의 사항
    - application/json 은 스펙상 utf-8 형식을 사용하도록 정의되어 있다.
    - 따라서  `application/json`이라고만 사용해야지 `application/json;charset=utf-8` 이라고 전달하는 것은 의미 없는 파라미터를 추가한 것이 된다.
- `response.getWriter()` 사용 시 주의 사항
    - `response.getWriter()`를 사용하면 추가 파라미터를 자동으로 추가해버린다.
    - 이때는 `response.getOutputStream()`으로 출력하면 그런 문제가 없다