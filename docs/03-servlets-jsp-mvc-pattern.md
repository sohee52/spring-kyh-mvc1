# 3. 서블릿, JSP, MVC 패턴

## 서블릿만으로 웹 어플리케이션 만들기

- 서블릿으로 웹 어플리케이션을 만들기 위한 코드
    - 회원 도메인 모델 `Member`
    - 회원 저장소 `MemberRepository`
    - 회원 등록 폼 `MemberFormServlet`
    - 회원 저장 `MemberSaveServlet`
    - 회원 목록 조회 `MemberListServlet`
- 서블릿과 자바 코드만으로 HTML을 만들면 매우 복잡하고 비효율적이다. 이로 인해 나온 것은?
    - 템플릿 엔진!
    - 예) JSP, Thymeleaf, Freemarker, Velocity 등

## JSP 코드 작성법

- `<%@ page contentType="text/html;charset=UTF-8" language="java" %>`
    - 첫 줄은 JSP문서라는 뜻이다. JSP 문서는 이렇게 시작해야 한다.
- `<%@ page import="hello.servlet.domain.member.MemberRepository" %>`
    - 자바의 import 문과 같다.
- `<% ~~ %>`
    - 이 부분에는 자바 코드를 입력할 수 있다.
- `<%= ~~ %>`
    - 이 부분에는 자바 코드를 출력할 수 있다.

## MVC 패턴

- MVC 패턴의 장점
    - 변경의 라이프 사이클이 다른 코드를 따로 관리할 수 있다.
        - 예를 들어서 UI를 일부 수정하는 일과 비즈니스 로직을 수정하는 일은 각각 다르게 발생할 가능성이 매우 높고 대부분 서로에게 영향을 주지 않는다. 이렇게 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수하기 좋지 않다.
    - 기능 특화
        - 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어 있기 때문에 이 부분의 업무만 담당하는 것이 가장 효과적이다.
- 컨트롤러
    - HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직이 있는 서비스를 호출한다.
    - 서비스는 데이터를 조회해서 모델에 담는다.
- 모델
    - 뷰에 출력할 데이터를 담아둔다.
    - 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링 하는 일에 집중할 수 있다.
- 뷰
    - 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다.
    - 여기서는 HTML을 생성하는 부분을 말한다.

## 서블릿+JSP인 MVC 패턴으로 웹 어플리케이션 만들기

### 회원 등록

- 컨트롤러

    ```java
    package hello.servlet.web.servletmvc;
    import javax.servlet.RequestDispatcher;
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    @WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
    public class MvcMemberFormServlet extends HttpServlet {
     @Override
     protected void service(HttpServletRequest request, HttpServletResponse
    response)
     throws ServletException, IOException {
    	 String viewPath = "/WEB-INF/views/new-form.jsp";
    	 RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    	 dispatcher.forward(request, response);
     }
    }
    ```

    - `dispatcher.forward()`
        - 다른 서블릿이나 JSP로 이동할 수 있는 기능이다.
        - 서버 내부에서 다시 호출이 발생한다.
        - 컨트롤러에서 뷰로 이동하게 해준다.
    - redirect vs forward
        - 리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다.
        - 반면에 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다
- 뷰

    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
     <meta charset="UTF-8">
     <title>Title</title>
    </head>
    <body>
    <!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
    <form action="save" method="post">
     username: <input type="text" name="username" />
     age: <input type="text" name="age" />
     <button type="submit">전송</button>
    </form>
    </body>
    </html>
    ```

    - 상대경로를 사용하면 폼 전송 시 무엇이 호출되나요?
        - 여기서 form의 action을 보면 절대 경로( / 로 시작)가 아니라 상대경로( / 로 시작X)인 것을 확인할 수 있다.
        - 이렇게 상대경로를 사용하면 폼 전송시 현재 URL이 속한 계층 경로 + save가 호출된다.
        - 현재 계층 경로: /servlet-mvc/members/
          결과: /servlet-mvc/members/save

### 회원 저장

- 컨트롤러

    ```java
    package hello.servlet.web.servletmvc;
    
    import hello.servlet.domain.member.Member;
    import hello.servlet.domain.member.MemberRepository;
    
    import javax.servlet.RequestDispatcher;
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    
    @WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
    public class MvcMemberSaveServlet extends HttpServlet {
    
        private MemberRepository memberRepository = MemberRepository.getInstance();
    
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response)
                throws ServletException, IOException {
    
            String username = request.getParameter("username");
            int age = Integer.parseInt(request.getParameter("age"));
    
            Member member = new Member(username, age);
            System.out.println("member = " + member);
    
            memberRepository.save(member);
    
            // Model에 데이터를 보관한다.
            request.setAttribute("member", member);
    
            String viewPath = "/WEB-INF/views/save-result.jsp";
            RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
            dispatcher.forward(request, response);
        }
    }
    
    ```

    - 객체 보관 및 꺼내는 방법
        - request가 제공하는 `setAttribute()` 를 사용하면 request 객체에 데이터를 보관해서 뷰에 전달할 수 있다.
        - 뷰는 `request.getAttribute()` 를 사용해서 데이터를 꺼내면 된다.
- 뷰

    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>회원 저장 결과</title>
    </head>
    <body>
        <h2>회원 저장 성공</h2>
        <ul>
            <li>id = ${member.id}</li>
            <li>username = ${member.username}</li>
            <li>age = ${member.age}</li>
        </ul>
        <a href="/index.html">메인으로</a>
    </body>
    </html>
    
    ```

    - <%= request.getAttribute("member")%> 로 모델에 저장한 member 객체를 꺼낼 수 있지만, 너무 복잡해진다.
    - JSP는 ${} 문법을 제공하는데, 이 문법을 사용하면 request의 attribute에 담긴 데이터를 편리하게 조회할 수 있다.

### 회원 목록 조회

- 컨트롤러

    ```java
    package hello.servlet.web.servletmvc;
    
    import hello.servlet.domain.member.Member;
    import hello.servlet.domain.member.MemberRepository;
    
    import javax.servlet.RequestDispatcher;
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.util.List;
    
    @WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/members")
    public class MvcMemberListServlet extends HttpServlet {
    
        private MemberRepository memberRepository = MemberRepository.getInstance();
    
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response)
                throws ServletException, IOException {
    
            System.out.println("MvcMemberListServlet.service");
    
            List<Member> members = memberRepository.findAll();
            request.setAttribute("members", members);
    
            String viewPath = "/WEB-INF/views/members.jsp";
            RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
            dispatcher.forward(request, response);
        }
    }
    
    ```

    - request 객체를 사용해서 List<Member> members 를 모델에 보관했다.
- 뷰

    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>회원 목록</title>
        <style>
            table {
                border-collapse: collapse;
                width: 50%;
                margin-top: 20px;
            }
            th, td {
                border: 1px solid #ddd;
                padding: 8px;
                text-align: center;
            }
            th {
                background-color: #f2f2f2;
            }
            a {
                display: inline-block;
                margin-bottom: 10px;
            }
        </style>
    </head>
    <body>
        <h2>회원 목록</h2>
        <a href="/index.html">메인으로</a>
    
        <table>
            <thead>
                <tr>
                    <th>id</th>
                    <th>username</th>
                    <th>age</th>
                </tr>
            </thead>
            <tbody>
                <c:forEach var="item" items="${members}">
                    <tr>
                        <td>${item.id}</td>
                        <td>${item.username}</td>
                        <td>${item.age}</td>
                    </tr>
                </c:forEach>
            </tbody>
        </table>
    </body>
    </html>
    
    ```

    - 모델에 담아둔 members를 JSP가 제공하는 taglib기능을 사용해서 반복하면서 출력했다.
    - members 리스트에서 member 를 순서대로 꺼내서 item 변수에 담고, 출력하는 과정을 반복한다.
    - <c:forEach> 이 기능을 사용하려면 다음과 같이 선언해야 한다.
      `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>`
    - 해당 기능을 사용하지 않고, 다음과 같이 출력해도 되지만, 매우 지저분하다.

        ```html
        <%
         for (Member member : members) {
        	 out.write(" <tr>");
        	 out.write(" <td>" + member.getId() + "</td>");
        	 out.write(" <td>" + member.getUsername() + "</td>");
        	 out.write(" <td>" + member.getAge() + "</td>");
        	 out.write(" </tr>");
         }
        %>
        ```


## MVC 패턴 - 한계

- MVC 컨트롤러의 단점
    - 포워드 중복
        - View로 이동하는 코드가 항상 중복 호출되어야 한다. 물론 이 부분을 메서드로 공통화해도 되지만, 해당 메서드도 항상
          직접 호출해야 한다.
        - `java RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath); dispatcher.forward(request, response);`
    - ViewPath에 중복
        - `String viewPath = "/WEB-INF/views/new-form.jsp";`
        - prefix: /WEB-INF/views/
          suffix: .jsp
        - 그리고 만약 jsp가 아닌 thymeleaf 같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야 한다.
    - 사용하지 않는 코드
        - 다음 코드를 사용할 때도 있고, 사용하지 않을 때도 있다. 특히 response는 현재 코드에서 사용되지 않는다. `HttpServletRequest request, HttpServletResponse response`
        - 그리고 이런 HttpServletRequest , HttpServletResponse 를 사용하는 코드는 테스트 케이스를 작성하기도 어렵다.
    - 공통 처리가 어렵다.
        - 기능이 복잡해질 수 록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다.
        - 단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 해당 메서드를 항상 호출해야 하고, 실수로 호출하지 않으면 문제가 될 것이다. 그리고 호출하는 것 자체도 중복이다.
    - MVC 컨트롤러 단점을 해결하는 방법
        - 이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 수문장 역할을 하는 기능이 필요하다.
        - 프론트 컨트롤러(Front Controller) 패턴을 도입하면 이런 문제를 깔끔하게 해결할 수 있다.(입구를 하나로!)
        - 스프링 MVC의 핵심도 바로 이 프론트 컨트롤러에 있다.