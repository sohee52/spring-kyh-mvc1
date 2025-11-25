# 4. MVC 프레임워크 만들기

- 프론트 컨트롤러(Front Controller) 패턴
    - 프론트 컨트롤러는 하나의 서블릿만으로 모든 클라이언트 요청을 받아들이고, 적절한 컨트롤러를 찾아 호출합니다.
    - 이렇게 “입구를 하나로” 통합하면 공통 로직(인증, 로깅 등)을 한곳에서 처리할 수 있고, 개별 컨트롤러는 더 이상 서블릿 API에 의존하지 않아도 됩니다.

    ![img_3.png](img_3.png)
- 프론트 컨트롤러 패턴이 스프링 MVC와 무슨 관계가 있죠?
    - 스프링 MVC의 **DispatcherServlet**이 바로 이 FrontController 패턴을 구현한 핵심 컴포넌트입니다.
    - 따라서 우리가 단계별로 만든 예제는 DispatcherServlet의 축소판이라 볼 수 있습니다.
- 1단계(V1) 구조 및 코드

    ![img_4.png](img_4.png)
    - 서블릿과 비슷한 모양의 컨트롤러 인터페이스를 도입한다.

        ```java
        package hello.servlet.web.frontcontroller.v1;
        
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public interface ControllerV1 {
            void process(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException;
        }
        ```

        - 각 컨트롤러들은 이 인터페이스를 구현하면 된다. 프론트 컨트롤러는 이 인터페이스를 호출해서 구현과 관계없이 로직의 일관성을 가져갈 수 있다.
    - 회원 등록 컨트롤러

        ```java
        package hello.servlet.web.frontcontroller.v1.controller;
        
        import hello.servlet.web.frontcontroller.v1.ControllerV1;
        
        import javax.servlet.RequestDispatcher;
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public class MemberFormControllerV1 implements ControllerV1 {
        
            @Override
            public void process(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
                String viewPath = "/WEB-INF/views/new-form.jsp";
                RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
                dispatcher.forward(request, response);
            }
        }
        ```

    - 회원 저장 컨트롤러

        ```java
        package hello.servlet.web.frontcontroller.v1.controller;
        
        import hello.servlet.domain.member.Member;
        import hello.servlet.domain.member.MemberRepository;
        import hello.servlet.web.frontcontroller.v1.ControllerV1;
        
        import javax.servlet.RequestDispatcher;
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public class MemberSaveControllerV1 implements ControllerV1 {
        
            private final MemberRepository memberRepository = MemberRepository.getInstance();
        
            @Override
            public void process(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
        
                String username = request.getParameter("username");
                int age = Integer.parseInt(request.getParameter("age"));
        
                Member member = new Member(username, age);
                memberRepository.save(member);
        
                request.setAttribute("member", member);
        
                String viewPath = "/WEB-INF/views/save-result.jsp";
                RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
                dispatcher.forward(request, response);
            }
        }
        ```

    - 회원 목록 컨트롤러

        ```java
        package hello.servlet.web.frontcontroller.v1.controller;
        
        import hello.servlet.domain.member.Member;
        import hello.servlet.domain.member.MemberRepository;
        import hello.servlet.web.frontcontroller.v1.ControllerV1;
        
        import javax.servlet.RequestDispatcher;
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        import java.util.List;
        
        public class MemberListControllerV1 implements ControllerV1 {
        
            private final MemberRepository memberRepository = MemberRepository.getInstance();
        
            @Override
            public void process(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
        
                List<Member> members = memberRepository.findAll();
                request.setAttribute("members", members);
        
                String viewPath = "/WEB-INF/views/members.jsp";
                RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
                dispatcher.forward(request, response);
            }
        }
        ```

    - 프론트 컨트롤러

        ```java
        package hello.servlet.web.frontcontroller.v1;
        
        import hello.servlet.web.frontcontroller.v1.controller.MemberFormControllerV1;
        import hello.servlet.web.frontcontroller.v1.controller.MemberListControllerV1;
        import hello.servlet.web.frontcontroller.v1.controller.MemberSaveControllerV1;
        
        import javax.servlet.ServletException;
        import javax.servlet.annotation.WebServlet;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        import java.util.HashMap;
        import java.util.Map;
        
        @WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
        public class FrontControllerServletV1 extends HttpServlet {
        
            private final Map<String, ControllerV1> controllerMap = new HashMap<>();
        
            public FrontControllerServletV1() {
                controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
                controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
                controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
            }
        
            @Override
            protected void service(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
        
                System.out.println("FrontControllerServletV1.service");
        
                String requestURI = request.getRequestURI();
                ControllerV1 controller = controllerMap.get(requestURI);
        
                if (controller == null) {
                    response.setStatus(HttpServletResponse.SC_NOT_FOUND);
                    return;
                }
        
                controller.process(request, response);
            }
        }
        ```

        - urlPatterns
            - `urlPatterns = "/front-controller/v1/*"` : /front-controller/v1 를 포함한 하위 모든 요청은 이 서블릿에서 받아들인다.
            - 예) /front-controller/v1 , /front-controller/v1/a , /front-controller/v1/a/b
        - **controllerMap**
            - **key: 매핑 URL**
            - **value: 호출될 컨트롤러**
        - service()
            - 먼저 requestURI 를 조회해서 실제 호출할 컨트롤러를 controllerMap 에서 찾는다. 만약 없다면 404(SC_NOT_FOUND) 상태 코드를 반환한다.
            - 컨트롤러를 찾고 controller.process(request, response); 을 호출해서 해당 컨트롤러를 실행한다.
        - JSP
            - JSP는 이전 MVC에서 사용했던 것을 그대로 사용한다.
- 1단계(V1)에서는 무엇을 했나요?
    - 기존 서블릿-기반 코드를 유지한 채 `FrontControllerServletV1`을 도입했습니다.
    - URL-패턴(`/front-controller/v1/*`)에 따라 **controllerMap**에서 컨트롤러를 찾아 `process()`를 실행하는 구조로 전환해 “단일 진입점”을 확보했습니다.
- 2단계(V2) 구조 및 코드

    ![img_5.png](img_5.png)
    - MyView

        ```java
        package hello.servlet.web.frontcontroller;
        
        import javax.servlet.RequestDispatcher;
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public class MyView {
        
            private final String viewPath;
        
            public MyView(String viewPath) {
                this.viewPath = viewPath;
            }
        
            public void render(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
                RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
                dispatcher.forward(request, response);
            }
        }
        ```

    - ControllerV2

        ```java
        package hello.servlet.web.frontcontroller.v2;
        
        import hello.servlet.web.frontcontroller.MyView;
        
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public interface ControllerV2 {
        
            MyView process(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException;
        }
        ```

    - MemberFormControllerV2 - 회원 등록 폼

        ```java
        package hello.servlet.web.frontcontroller.v2.controller;
        
        import hello.servlet.web.frontcontroller.MyView;
        import hello.servlet.web.frontcontroller.v2.ControllerV2;
        
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public class MemberFormControllerV2 implements ControllerV2 {
        
            @Override
            public MyView process(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
                return new MyView("/WEB-INF/views/new-form.jsp");
            }
        }
        ```

        - 이제 각 컨트롤러는 복잡한 dispatcher.forward() 를 직접 생성해서 호출하지 않아도 된다. 단순히 MyView 객체를 생성하고 거기에 뷰 이름만 넣고 반환하면 된다.
        - ControllerV1 을 구현한 클래스와 ControllerV2 를 구현한 클래스를 비교해보면, 이 부분의 중복이 확실하게 제거된 것을 확인할 수 있다.
    - MemberSaveControllerV2 - 회원 저장

        ```java
        package hello.servlet.web.frontcontroller.v2.controller;
        
        import hello.servlet.domain.member.Member;
        import hello.servlet.domain.member.MemberRepository;
        import hello.servlet.web.frontcontroller.MyView;
        import hello.servlet.web.frontcontroller.v2.ControllerV2;
        
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public class MemberSaveControllerV2 implements ControllerV2 {
        
            private final MemberRepository memberRepository = MemberRepository.getInstance();
        
            @Override
            public MyView process(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
        
                String username = request.getParameter("username");
                int age = Integer.parseInt(request.getParameter("age"));
        
                Member member = new Member(username, age);
                memberRepository.save(member);
        
                request.setAttribute("member", member);
        
                return new MyView("/WEB-INF/views/save-result.jsp");
            }
        }
        ```

    - MemberListControllerV2 - 회원 목록

        ```java
        package hello.servlet.web.frontcontroller.v2.controller;
        
        import hello.servlet.domain.member.Member;
        import hello.servlet.domain.member.MemberRepository;
        import hello.servlet.web.frontcontroller.MyView;
        import hello.servlet.web.frontcontroller.v2.ControllerV2;
        
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        import java.util.List;
        
        public class MemberListControllerV2 implements ControllerV2 {
        
            private final MemberRepository memberRepository = MemberRepository.getInstance();
        
            @Override
            public MyView process(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
        
                List<Member> members = memberRepository.findAll();
                request.setAttribute("members", members);
        
                return new MyView("/WEB-INF/views/members.jsp");
            }
        }
        ```

    - 프론트 컨트롤러 V2

        ```java
        package hello.servlet.web.frontcontroller.v2;
        
        import hello.servlet.web.frontcontroller.MyView;
        import hello.servlet.web.frontcontroller.v2.controller.MemberFormControllerV2;
        import hello.servlet.web.frontcontroller.v2.controller.MemberListControllerV2;
        import hello.servlet.web.frontcontroller.v2.controller.MemberSaveControllerV2;
        
        import javax.servlet.ServletException;
        import javax.servlet.annotation.WebServlet;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        import java.util.HashMap;
        import java.util.Map;
        
        @WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
        public class FrontControllerServletV2 extends HttpServlet {
        
            private final Map<String, ControllerV2> controllerMap = new HashMap<>();
        
            public FrontControllerServletV2() {
                controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
                controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
                controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
            }
        
            @Override
            protected void service(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
        
                String requestURI = request.getRequestURI();
                ControllerV2 controller = controllerMap.get(requestURI);
        
                if (controller == null) {
                    response.setStatus(HttpServletResponse.SC_NOT_FOUND);
                    return;
                }
        
                MyView view = controller.process(request, response);
                view.render(request, response);
            }
        }
        ```

        - ControllerV2의 반환 타입이 MyView 이므로 프론트 컨트롤러는 컨트롤러의 호출 결과로 MyView 를 반환 받는다.
        - 그리고 view.render() 를 호출하면 forward 로직을 수행해서 JSP가 실행된다.
        - 프론트 컨트롤러의 도입으로 MyView 객체의 render() 를 호출하는 부분을 모두 일관되게 처리할 수 있다. 각각의 컨트롤러는 MyView 객체를 생성만 해서 반환하면 된다.
- 2단계(V2)에서는 어떤 개선이 있었나요?
    - 모든 컨트롤러에서 뷰로 이동하는 부분에 중복이 있고, 깔끔하지 않았다.
    - 컨트롤러마다 반복되던 `dispatcher.forward()` 코드를 없애고, 별도로 뷰를 처리하는 객체인 **MyView** 객체에 `render()`를 위임했습니다.
    - 이로써 컨트롤러는 뷰의 경로만 넘기면 되고, 공통 뷰 처리 로직은 FrontController가 일관되게 수행합니다
- 3단계(V3) 구조 및 코드

    ![img_6.png](img_6.png)

    - ModelView

        ```java
        package hello.servlet.web.frontcontroller;
        
        import java.util.HashMap;
        import java.util.Map;
        
        public class ModelView {
        
            private String viewName;
            private Map<String, Object> model = new HashMap<>();
        
            public ModelView(String viewName) {
                this.viewName = viewName;
            }
        
            public String getViewName() {
                return viewName;
            }
        
            public void setViewName(String viewName) {
                this.viewName = viewName;
            }
        
            public Map<String, Object> getModel() {
                return model;
            }
        
            public void setModel(Map<String, Object> model) {
                this.model = model;
            }
        }
        ```

        - 뷰의 이름과 뷰를 렌더링할 때 필요한 model 객체를 가지고 있다. model은 단순히 map으로 되어 있으므로 컨트롤러에서 뷰에 필요한 데이터를 key, value로 넣어주면 된다.
    - ControllerV3

        ```java
        package hello.servlet.web.frontcontroller.v3;
        
        import hello.servlet.web.frontcontroller.ModelView;
        import java.util.Map;
        
        public interface ControllerV3 {
        
            ModelView process(Map<String, String> paramMap);
        }
        ```

        - 이 컨트롤러는 서블릿 기술을 전혀 사용하지 않는다. 따라서 구현이 매우 단순해지고, 테스트 코드 작성시 테스트 하기 쉽다.
        - HttpServletRequest가 제공하는 파라미터는 프론트 컨트롤러가 paramMap에 담아서 호출해주면 된다.
        - 응답 결과로 뷰 이름과 뷰에 전달할 Model 데이터를 포함하는 ModelView 객체를 반환하면 된다.
    - MemberFormControllerV3 - 회원 등록 폼

        ```java
        package hello.servlet.web.frontcontroller.v3.controller;
        
        import hello.servlet.web.frontcontroller.ModelView;
        import hello.servlet.web.frontcontroller.v3.ControllerV3;
        import java.util.Map;
        
        public class MemberFormControllerV3 implements ControllerV3 {
        
            @Override
            public ModelView process(Map<String, String> paramMap) {
                return new ModelView("new-form");
            }
        }
        ```

        - ModelView 를 생성할 때 new-form 이라는 view의 논리적인 이름을 지정한다.
        - 실제 물리적인 이름은 프론트 컨트롤러에서 처리한다.
    - MemberSaveControllerV3 - 회원 저장

        ```java
        package hello.servlet.web.frontcontroller.v3.controller;
        
        import hello.servlet.domain.member.Member;
        import hello.servlet.domain.member.MemberRepository;
        import hello.servlet.web.frontcontroller.ModelView;
        import hello.servlet.web.frontcontroller.v3.ControllerV3;
        
        import java.util.Map;
        
        public class MemberSaveControllerV3 implements ControllerV3 {
        
            private final MemberRepository memberRepository = MemberRepository.getInstance();
        
            @Override
            public ModelView process(Map<String, String> paramMap) {
                String username = paramMap.get("username");
                int age = Integer.parseInt(paramMap.get("age"));
        
                Member member = new Member(username, age);
                memberRepository.save(member);
        
                ModelView mv = new ModelView("save-result");
                mv.getModel().put("member", member);
        
                return mv;
            }
        }
        ```

        - `paramMap.get("username");`
            - 파라미터 정보는 map에 담겨있다. map에서 필요한 요청 파라미터를 조회하면 된다.
        - `mv.getModel().put("member", member);`
            - 모델은 단순한 map이므로 모델에 뷰에서 필요한 member 객체를 담고 반환한다.
    - MemberListControllerV3 - 회원 목록

        ```java
        package hello.servlet.web.frontcontroller.v3.controller;
        
        import hello.servlet.domain.member.Member;
        import hello.servlet.domain.member.MemberRepository;
        import hello.servlet.web.frontcontroller.ModelView;
        import hello.servlet.web.frontcontroller.v3.ControllerV3;
        
        import java.util.List;
        import java.util.Map;
        
        public class MemberListControllerV3 implements ControllerV3 {
        
            private final MemberRepository memberRepository = MemberRepository.getInstance();
        
            @Override
            public ModelView process(Map<String, String> paramMap) {
                List<Member> members = memberRepository.findAll();
        
                ModelView mv = new ModelView("members");
                mv.getModel().put("members", members);
        
                return mv;
            }
        }
        ```

    - FrontControllerServletV3

        ```c
        package hello.servlet.web.frontcontroller.v3;
        
        import hello.servlet.web.frontcontroller.ModelView;
        import hello.servlet.web.frontcontroller.MyView;
        import hello.servlet.web.frontcontroller.v3.controller.MemberFormControllerV3;
        import hello.servlet.web.frontcontroller.v3.controller.MemberListControllerV3;
        import hello.servlet.web.frontcontroller.v3.controller.MemberSaveControllerV3;
        
        import javax.servlet.ServletException;
        import javax.servlet.annotation.WebServlet;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        import java.util.HashMap;
        import java.util.Map;
        
        @WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
        public class FrontControllerServletV3 extends HttpServlet {
        
            private final Map<String, ControllerV3> controllerMap = new HashMap<>();
        
            public FrontControllerServletV3() {
                controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
                controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
                controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
            }
        
            @Override
            protected void service(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
        
                String requestURI = request.getRequestURI();
                ControllerV3 controller = controllerMap.get(requestURI);
        
                if (controller == null) {
                    response.setStatus(HttpServletResponse.SC_NOT_FOUND);
                    return;
                }
        
                Map<String, String> paramMap = createParamMap(request);
                ModelView mv = controller.process(paramMap);
        
                String viewName = mv.getViewName();
                MyView view = viewResolver(viewName);
        
                view.render(mv.getModel(), request, response);
            }
        
            private Map<String, String> createParamMap(HttpServletRequest request) {
                Map<String, String> paramMap = new HashMap<>();
                request.getParameterNames().asIterator()
                        .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
                return paramMap;
            }
        
            private MyView viewResolver(String viewName) {
                return new MyView("/WEB-INF/views/" + viewName + ".jsp");
            }
        }
        ```

        - `createParamMap()`
            - HttpServletRequest에서 파라미터 정보를 꺼내서 Map으로 변환한다. 그리고 해당 Map( paramMap )을 컨트롤러
              에 전달하면서 호출한다.
        - 뷰 리졸버
            - `MyView view = viewResolver(viewName)`
                - 컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경한다. 그리고 실제 물리 경로가 있는 MyView 객체를 반환한다.
                - 논리 뷰 이름: members
                  물리 뷰 경로: /WEB-INF/views/members.jsp
            - `view.render(mv.getModel(), request, response)`
                - 뷰 객체를 통해서 HTML 화면을 렌더링 한다.
                - 뷰 객체의 render() 는 모델 정보도 함께 받는다.
                - JSP는 request.getAttribute() 로 데이터를 조회하기 때문에, 모델의 데이터를 꺼내서 `request.setAttribute()` 로 담아둔다.
                - JSP로 포워드 해서 JSP를 렌더링 한다.
    - MyView

        ```java
        package hello.servlet.web.frontcontroller;
        
        import javax.servlet.RequestDispatcher;
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        import java.util.Map;
        
        public class MyView {
        
            private final String viewPath;
        
            public MyView(String viewPath) {
                this.viewPath = viewPath;
            }
        
            public void render(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
                RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
                dispatcher.forward(request, response);
            }
        
            public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
                modelToRequestAttribute(model, request);
                RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
                dispatcher.forward(request, response);
            }
        
            private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
                model.forEach(request::setAttribute);
            }
        }
        ```

- 3단계(V3)의 핵심 변화는?
    - **서블릿 종속성 제거**
        - 지금까지 컨트롤러에서 서블릿에 종속적인 HttpServletRequest를 사용했다. 그리고 Model도 request.setAttribute() 를 통해 데이터를 저장하고 뷰에 전달했다.
        - 컨트롤러 입장에서 HttpServletRequest, HttpServletResponse이 꼭 필요할까?
        - ㅊ
        - 서블릿의 종속성을 제거하기 위해 Model을 직접 만들고, 추가로 View 이름까지 전달하는 객체를 만들어서, 우리가 구현하는 컨트롤러가 서블릿 기술을 전혀 사용하지 않도록 변경해보자.
        - 이렇게 하면 구현 코드도 매우 단순해지고, 테스트 코드 작성이 쉽다.
    - **뷰 이름 중복 제거**
        - 컨트롤러에서 지정하는 뷰 이름에 중복이 있는 것을 확인할 수 있다.
        - **컨트롤러는 뷰의 논리 이름을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화 하자.**
        - 이렇게 해두면 향후 뷰의 폴더 위치가 함께 이동해도 프론트 컨트롤러만 고치면 된다.
        - /WEB-INF/views/new-form.jsp → new-form
          /WEB-INF/views/save-result.jsp → save-result
          /WEB-INF/views/members.jsp → members
    - 정리 : FrontController가 요청 파라미터를 `paramMap`에 담아 전달하고, 컨트롤러는 `ModelView`에 **논리 뷰 이름**과 모델 데이터를 채워 반환합니다. 이렇게 하면 테스트가 쉬워지고, 뷰 폴더가 바뀌어도 FrontController만 수정하면 됩니다.
- 4단계(V4) 구조 및 코드

    ![img_7.png](img_7.png)
    - 기본적인 구조는 V3와 같다. 대신에 컨트롤러가 ModelView 를 반환하지 않고, ViewName 만 반환한다.
    - ControllerV4

        ```java
        package hello.servlet.web.frontcontroller.v4;
        
        import java.util.Map;
        
        public interface ControllerV4 {
        
            /**
             * @param paramMap 요청 파라미터 맵
             * @param model    뷰에 전달할 데이터
             * @return viewName 논리적인 뷰 이름
             */
            String process(Map<String, String> paramMap, Map<String, Object> model);
        }
        ```

        - 이번 버전은 인터페이스에 ModelView가 없다. model 객체는 파라미터로 전달되기 때문에 그냥 사용하면 되고, 결과로 뷰의 이름만 반환해주면 된다.
    - MemberFormControllerV4

        ```java
        package hello.servlet.web.frontcontroller.v4.controller;
        
        import hello.servlet.web.frontcontroller.v4.ControllerV4;
        import java.util.Map;
        
        public class MemberFormControllerV4 implements ControllerV4 {
        
            @Override
            public String process(Map<String, String> paramMap, Map<String, Object> model) {
                return "new-form";
            }
        }
        ```

        - 정말 단순하게 new-form 이라는 뷰의 논리 이름만 반환하면 된다.
    - MemberSaveControllerV4

        ```java
        package hello.servlet.web.frontcontroller.v4.controller;
        
        import hello.servlet.domain.member.Member;
        import hello.servlet.domain.member.MemberRepository;
        import hello.servlet.web.frontcontroller.v4.ControllerV4;
        
        import java.util.Map;
        
        public class MemberSaveControllerV4 implements ControllerV4 {
        
            private final MemberRepository memberRepository = MemberRepository.getInstance();
        
            @Override
            public String process(Map<String, String> paramMap, Map<String, Object> model) {
                String username = paramMap.get("username");
                int age = Integer.parseInt(paramMap.get("age"));
        
                Member member = new Member(username, age);
                memberRepository.save(member);
        
                model.put("member", member);
        
                return "save-result";
            }
        }
        ```

        - `model.put("member", member)`
            - 모델이 파라미터로 전달되기 때문에, 모델을 직접 생성하지 않아도 된다.
    - MemberListControllerV4

        ```java
        package hello.servlet.web.frontcontroller.v4.controller;
        
        import hello.servlet.domain.member.Member;
        import hello.servlet.domain.member.MemberRepository;
        import hello.servlet.web.frontcontroller.v4.ControllerV4;
        
        import java.util.List;
        import java.util.Map;
        
        public class MemberListControllerV4 implements ControllerV4 {
        
            private final MemberRepository memberRepository = MemberRepository.getInstance();
        
            @Override
            public String process(Map<String, String> paramMap, Map<String, Object> model) {
                List<Member> members = memberRepository.findAll();
                model.put("members", members);
        
                return "members";
            }
        }
        ```

    - FrontControllerServletV4

        ```java
        package hello.servlet.web.frontcontroller.v4;
        
        import hello.servlet.web.frontcontroller.MyView;
        import hello.servlet.web.frontcontroller.v4.controller.MemberFormControllerV4;
        import hello.servlet.web.frontcontroller.v4.controller.MemberListControllerV4;
        import hello.servlet.web.frontcontroller.v4.controller.MemberSaveControllerV4;
        
        import javax.servlet.ServletException;
        import javax.servlet.annotation.WebServlet;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        import java.util.HashMap;
        import java.util.Map;
        
        @WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/v4/*")
        public class FrontControllerServletV4 extends HttpServlet {
        
            private final Map<String, ControllerV4> controllerMap = new HashMap<>();
        
            public FrontControllerServletV4() {
                controllerMap.put("/front-controller/v4/members/new-form", new MemberFormControllerV4());
                controllerMap.put("/front-controller/v4/members/save", new MemberSaveControllerV4());
                controllerMap.put("/front-controller/v4/members", new MemberListControllerV4());
            }
        
            @Override
            protected void service(HttpServletRequest request, HttpServletResponse response)
                    throws ServletException, IOException {
        
                String requestURI = request.getRequestURI();
                ControllerV4 controller = controllerMap.get(requestURI);
        
                if (controller == null) {
                    response.setStatus(HttpServletResponse.SC_NOT_FOUND);
                    return;
                }
        
                Map<String, String> paramMap = createParamMap(request);
                Map<String, Object> model = new HashMap<>();
        
                String viewName = controller.process(paramMap, model);
                MyView view = viewResolver(viewName);
        
                view.render(model, request, response);
            }
        
            private Map<String, String> createParamMap(HttpServletRequest request) {
                Map<String, String> paramMap = new HashMap<>();
                request.getParameterNames().asIterator()
                        .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
                return paramMap;
            }
        
            private MyView viewResolver(String viewName) {
                return new MyView("/WEB-INF/views/" + viewName + ".jsp");
            }
        }
        ```

        - 모델 객체 전달
            - `Map<String, Object> model = new HashMap<>(); //추가`
            - 모델 객체를 프론트 컨트롤러에서 생성해서 넘겨준다. 컨트롤러에서 모델 객체에 값을 담으면 여기에 그대로 담겨있게
              된다.
        - 뷰의 논리 이름을 직접 반환

            ```java
            java String viewName = controller.process(paramMap, model);
            MyView view = viewResolver(viewName);
            ```

            - 컨트롤러가 직접 뷰의 논리 이름을 반환하므로 이 값을 사용해서 실제 물리 뷰를 찾을 수 있다.
- 4단계(V4)는 왜 더 “실용적”이라고 하나요?
    - V3는 항상 ModelView 객체를 생성하고 반환해야 하는 부분이
      조금은 번거롭다
    - 개발자가 `ModelView`를 직접 만들 필요조차 없도록 인터페이스를 단순화했습니다.
    - 컨트롤러는 `process(paramMap, model)`을 호출받아 **model**에 값만 넣고 **뷰 논리 이름**을 문자열로 돌려주면 끝입니다.
    - FrontController가 나머지(Model→View Resolver→Render)를 처리해줘서 코드가 훨씬 간결해졌습니다.
- 5단계(V5) 구조 및 코드

    ![img_8.png](img_8.png)
    - 핸들러 어댑터
        - 중간에 어댑터 역할을 하는 어댑터가 추가되었는데 이름이 핸들러 어댑터이다.
        - 여기서 어댑터 역할을 해주는 덕분에 다양한 종류의 컨트롤러를 호출할 수 있다.
    - 핸들러
        - 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경했다.
        - 그 이유는 이제 어댑터가 있기 때문에 꼭 컨트롤러의 개념 뿐만 아니라 어떠한 것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있기 때문이다.
    - MyHandlerAdapter

        ```java
        package hello.servlet.web.frontcontroller.v5;
        
        import hello.servlet.web.frontcontroller.ModelView;
        
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public interface MyHandlerAdapter {
        
            boolean supports(Object handler);
        
            ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
                    throws ServletException, IOException;
        }
        ```

        - `boolean supports(Object handler)`
            - handler는 컨트롤러를 말한다.
            - 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드다.
        - `ModelView handle(HttpServletRequest request, HttpServletResponse response,
        Object handler)`
            - 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다.
            - 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 ModelView를 직접 생성해서라도 반환해야 한다.
            - 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만 이제는 이 어댑터를 통해서 실제 컨트롤러가 호출된다.
        - ControllerV3HandlerAdapter

            ```java
            package hello.servlet.web.frontcontroller.v5.adapter;
            
            import hello.servlet.web.frontcontroller.ModelView;
            import hello.servlet.web.frontcontroller.v3.ControllerV3;
            import hello.servlet.web.frontcontroller.v5.MyHandlerAdapter;
            
            import javax.servlet.http.HttpServletRequest;
            import javax.servlet.http.HttpServletResponse;
            import java.util.HashMap;
            import java.util.Map;
            
            public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
            
                @Override
                public boolean supports(Object handler) {
                    return (handler instanceof ControllerV3);
                }
            
                @Override
                public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) {
                    ControllerV3 controller = (ControllerV3) handler;
                    Map<String, String> paramMap = createParamMap(request);
                    return controller.process(paramMap);
                }
            
                private Map<String, String> createParamMap(HttpServletRequest request) {
                    Map<String, String> paramMap = new HashMap<>();
                    request.getParameterNames().asIterator()
                            .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
                    return paramMap;
                }
            }
            ```

            - ControllerV3 을 처리할 수 있는 어댑터

                ```java
                public boolean supports(Object handler) {
                 return (handler instanceof ControllerV3);
                }
                ```

            - handler를 컨트롤러 V3로 변환한 다음에 V3 형식에 맞도록 호출

                ```java
                ControllerV3 controller = (ControllerV3) handler;
                Map<String, String> paramMap = createParamMap(request);
                ModelView mv = controller.process(paramMap);
                return mv;
                ```

                - supports() 를 통해 ControllerV3 만 지원하기 때문에 타입 변환은 걱정없이 실행해도 된다.
                - ControllerV3는 ModelView를 반환하므로 그대로 ModelView를 반환하면 된다.
        - FrontControllerServletV5

            ```java
            package hello.servlet.web.frontcontroller.v5;
            
            import hello.servlet.web.frontcontroller.ModelView;
            import hello.servlet.web.frontcontroller.MyView;
            import hello.servlet.web.frontcontroller.v3.controller.MemberFormControllerV3;
            import hello.servlet.web.frontcontroller.v3.controller.MemberListControllerV3;
            import hello.servlet.web.frontcontroller.v3.controller.MemberSaveControllerV3;
            import hello.servlet.web.frontcontroller.v5.adapter.ControllerV3HandlerAdapter;
            import hello.servlet.web.frontcontroller.v5.adapter.ControllerV4HandlerAdapter;
            
            import javax.servlet.ServletException;
            import javax.servlet.annotation.WebServlet;
            import javax.servlet.http.HttpServlet;
            import javax.servlet.http.HttpServletRequest;
            import javax.servlet.http.HttpServletResponse;
            import java.io.IOException;
            import java.util.ArrayList;
            import java.util.HashMap;
            import java.util.List;
            import java.util.Map;
            
            @WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
            public class FrontControllerServletV5 extends HttpServlet {
            
                private final Map<String, Object> handlerMappingMap = new HashMap<>(); // Object로 변경
                private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();
            
                public FrontControllerServletV5() {
                    initHandlerMappingMap(); //핸들러 매핑 초기화
                    initHandlerAdapters(); //어댑터 초기화
                }
            
                private void initHandlerMappingMap() {
                    handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
                    handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
                    handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
                }
            
                private void initHandlerAdapters() {
                    handlerAdapters.add(new ControllerV3HandlerAdapter());
                    handlerAdapters.add(new ControllerV4HandlerAdapter());
                }
            
                @Override
                protected void service(HttpServletRequest request, HttpServletResponse response)
                        throws ServletException, IOException {
            
                    Object handler = getHandler(request);
                    if (handler == null) {
                        response.setStatus(HttpServletResponse.SC_NOT_FOUND);
                        return;
                    }
            
                    MyHandlerAdapter adapter = getHandlerAdapter(handler);
                    ModelView mv = adapter.handle(request, response, handler);
            
                    MyView view = viewResolver(mv.getViewName());
                    view.render(mv.getModel(), request, response);
                }
            
                private Object getHandler(HttpServletRequest request) {
                    String requestURI = request.getRequestURI();
                    return handlerMappingMap.get(requestURI);
                }
            
                private MyHandlerAdapter getHandlerAdapter(Object handler) {
                    for (MyHandlerAdapter adapter : handlerAdapters) {
                        if (adapter.supports(handler)) {
                            return adapter;
                        }
                    }
                    throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. handler=" + handler);
                }
            
                private MyView viewResolver(String viewName) {
                    return new MyView("/WEB-INF/views/" + viewName + ".jsp");
                }
            }
            ```

            - 생성자

                ```java
                public FrontControllerServletV5() {
                 initHandlerMappingMap(); //핸들러 매핑 초기화
                 initHandlerAdapters(); //어댑터 초기화
                }
                ```

                - 핸들러 매핑과 어댑터를 초기화(등록)한다.
            - 필드 매핑 정보

              `private final Map<String, Object> handlerMappingMap = new HashMap<>();`

                - 매핑 정보의 값이 ControllerV3 , ControllerV4 같은 인터페이스에서 아무 값이나 받을 수 있는 Object 로 변경되었다
            - 핸들러 매핑

                ```java
                private Object getHandler(HttpServletRequest request) {
                 String requestURI = request.getRequestURI();
                 return handlerMappingMap.get(requestURI);
                }
                ```

                - 핸들러 매핑 정보인 handlerMappingMap 에서 URL에 매핑된 핸들러(컨트롤러) 객체를 찾아서 반환한다.
            - 핸들러를 처리할 수 있는 어댑터 조회

                ```java
                for (MyHandlerAdapter adapter : handlerAdapters) {
                 if (adapter.supports(handler)) {
                 return adapter;
                 }
                }
                ```

                - handler 를 처리할 수 있는 어댑터를 adapter.supports(handler) 를 통해서 찾는다.
                  handler가 ControllerV3 인터페이스를 구현했다면, ControllerV3HandlerAdapter 객체가 반환된다.
            - 어댑터 호출

              `ModelView mv = adapter.handle(request, response, handler);`

                - 어댑터의 handle(request, response, handler) 메서드를 통해 실제 어댑터가 호출된다.
                - 어댑터는 handler(컨트롤러)를 호출하고 그 결과를 어댑터에 맞추어 반환한다. ControllerV3HandlerAdapter
                  의 경우 어댑터의 모양과 컨트롤러의 모양이 유사해서 변환 로직이 단순하다
        - FrontControllerServletV5 에 ControllerV4 기능도 추가해보자.

            ```java
            private void initHandlerMappingMap() {
                handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
                handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
                handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
            
                // V4 추가
                handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
                handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
                handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
            }
            
            private void initHandlerAdapters() {
                handlerAdapters.add(new ControllerV3HandlerAdapter());
                handlerAdapters.add(new ControllerV4HandlerAdapter()); // V4 추가
            }
            ```

        - ControllerV4HandlerAdapter

            ```java
            package hello.servlet.web.frontcontroller.v5.adapter;
            
            import hello.servlet.web.frontcontroller.ModelView;
            import hello.servlet.web.frontcontroller.v4.ControllerV4;
            import hello.servlet.web.frontcontroller.v5.MyHandlerAdapter;
            
            import javax.servlet.http.HttpServletRequest;
            import javax.servlet.http.HttpServletResponse;
            import java.util.HashMap;
            import java.util.Map;
            
            public class ControllerV4HandlerAdapter implements MyHandlerAdapter {
            
                @Override
                public boolean supports(Object handler) {
                    return (handler instanceof ControllerV4);
                }
            
                @Override
                public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) {
                    ControllerV4 controller = (ControllerV4) handler;
                    Map<String, String> paramMap = createParamMap(request);
                    Map<String, Object> model = new HashMap<>();
            
                    String viewName = controller.process(paramMap, model);
            
                    ModelView mv = new ModelView(viewName);
                    mv.setModel(model);
            
                    return mv;
                }
            
                private Map<String, String> createParamMap(HttpServletRequest request) {
                    Map<String, String> paramMap = new HashMap<>();
                    request.getParameterNames().asIterator()
                            .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
                    return paramMap;
                }
            }
            ```

            - handler 가 ControllerV4 인 경우에만 처리하는 어댑터

                ```java
                public boolean supports(Object handler) {
                 return (handler instanceof ControllerV4);
                }
                ```

            - 실행 로직

                ```java
                ControllerV4 controller = (ControllerV4) handler;
                Map<String, String> paramMap = createParamMap(request);
                Map<String, Object> model = new HashMap<>();
                String viewName = controller.process(paramMap, model);
                ```

                - handler를 ControllerV4로 케스팅 하고, paramMap, model을 만들어서 해당 컨트롤러를 호출한다.
                - 그리고 viewName을 반환 받는다.
            - 어댑터 변환

                ```java
                ModelView mv = new ModelView(viewName);
                mv.setModel(model);
                return mv;
                ```

                - 어댑터가 호출하는 ControllerV4 는 뷰의 이름을 반환한다. 그런데 어댑터는 뷰의 이름이 아니라 ModelView 를
                  만들어서 반환해야 한다. 여기서 어댑터가 꼭 필요한 이유가 나온다.
                - ControllerV4 는 뷰의 이름을 반환했지만, 어댑터는 이것을 ModelView로 만들어서 형식을 맞추어 반환한다. 마치
                  110v 전기 콘센트를 220v 전기 콘센트로 변경하듯이!
            - 어댑터와 ControllerV4

                ```java
                public interface ControllerV4 {
                 String process(Map<String, String> paramMap, Map<String, Object> model);
                }
                public interface MyHandlerAdapter {
                 ModelView handle(HttpServletRequest request, HttpServletResponse response,
                Object handler) throws ServletException, IOException;
                }
                ```

- 5단계(V5)는 어떤 문제를 해결하나요?
    - V3 방식과 V4 방식처럼 **다른 형태의 컨트롤러**를 한 FrontController가 처리하려면 **어댑터 패턴**이 필요합니다.
    - 어댑터 패턴을 사용하면 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 할 수 있다.
    - `MyHandlerAdapter`가 “컨트롤러-전용 코드”를 감싸 표준 `ModelView`를 돌려주기 때문에, 새로운 컨트롤러 타입(예: 애너테이션 기반)도 어댑터만 추가하면 동작합니다.
- 단계별 발전을 한눈에 요약하면?
    - v1: 프론트 컨트롤러를 도입
      기존 구조를 최대한 유지하면서 프론트 컨트롤러를 도입
    - v2: View 분류
      단순 반복 되는 뷰 로직 분리
    - v3: Model 추가
      서블릿 종속성 제거
      뷰 이름 중복 제거
    - v4: 단순하고 실용적인 컨트롤러
      v3와 거의 비슷
      구현 입장에서 ModelView를 직접 생성해서 반환하지 않도록 편리한 인터페이스 제공
    - v5: 유연한 컨트롤러
      어댑터 도입
      어댑터를 추가해서 프레임워크를 유연하고 확장성 있게 설계
- 여기에 애노테이션을 사용해서 컨트롤러를 더 편리하게 발전시킬 수도 있다. 만약 애노테이션을 사용해서 컨트롤러를 편리하게 사용할 수 있게 하려면 어떻게 해야할까? 바로 애노테이션을 지원하는 어댑터를 추가하면 된다!
- 지금까지 작성한 코드는 스프링 MVC 프레임워크의 핵심 코드의 축약 버전이고, 구조도 거의 같다.