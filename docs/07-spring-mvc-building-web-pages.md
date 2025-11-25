# 7. 스프링 MVC - 웹 페이지 만들기

```java
package hello.itemservice.web.item.basic;

import hello.itemservice.domain.item.Item;
import hello.itemservice.domain.item.ItemRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import javax.annotation.PostConstruct;
import java.util.List;

@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "basic/items";
    }

    /**
     * 테스트용 데이터 추가
     */
    @PostConstruct
    public void init() {
        itemRepository.save(new Item("testA", 10000, 10));
        itemRepository.save(new Item("testB", 20000, 20));
    }
}
```

- `@RequiredArgsConstructor`
    - final 이 붙은 멤버변수만 사용해서 생성자를 자동으로 만들어준다.
    - final 키워드를 빼면 안된다! 그러면 ItemRepository 의존관계 주입이 안된다.
- `@PostConstruct`
    - 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출된다.

## 타임리프

- 타임리프 사용 선언

    <html xmlns:th="[http://www.thymeleaf.org](http://www.thymeleaf.org/)">

- th:xxx
    - **th:xxx 가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다.**
        - 렌더링(rendering) = 서버에서 HTML 템플릿(Thymeleaf 템플릿 파일)을 해석해서 최종 HTML 결과물을 만들어내는 과정
    - th:xxx 이 없으면 기존 html 의 xxx 속성이 그대로 사용된다.
        - 예) `href="../css/bootstrap.min.css”` `th:href="@{/css/bootstrap.min.css}"`
        - href=`"../css/bootstrap.min.css”` 을 th:href=`"@{/css/bootstrap.min.css}"` 의 값으로 변경한다.
        - HTML을 그대로 볼 때는 href 속성이 사용되고, 뷰 템플릿을 거치면 th:href 의 값이 href 로 대체되면서 동적으로 변경할 수 있다.
    - HTML을 파일로 직접 열었을 때, th:xxx 가 있어도 웹 브라우저는 th: 속성을 알지 못하므로 무시한다. 따라서 HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.
- @{...}
    - URL 링크 표현식이다.
    - 타임리프는 URL 링크를 사용하는 경우 @{...} 를 사용한다.
    - URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다.
        - 서블릿 컨텍스트란?

          여기서 말하는 **서블릿 컨텍스트(Servlet Context)** 는 쉽게 말하면

          👉 **웹 애플리케이션이 실행되는 기본 경로(context path)** 를 뜻해.
            
          ---

          ### 1. 서블릿 컨텍스트(Context Path)란?

            - 우리가 웹 애플리케이션을 서버에 올릴 때, 보통 도메인 뒤에 **하위 경로**가 붙어.
            - 그 하위 경로가 **컨텍스트 경로(context path)** 야.
            - 이 경로는 하나의 웹 애플리케이션을 구분하는 "루트 폴더" 같은 역할을 함.

          예시:

            - 애플리케이션을 `/myapp` 이라는 컨텍스트 경로로 배포했다고 해보자.
            - 서버 주소가 `http://localhost:8080` 이라면, 실제 애플리케이션 기본 경로는:

                ```
                http://localhost:8080/myapp
                
                ```

            - 여기서 `/myapp` 이 바로 **서블릿 컨텍스트 경로**임.

            ---

          ### 2. 타임리프 URL 링크 표현식과의 관계

          타임리프에서 `@{...}` 를 쓰면 **이 컨텍스트 경로를 자동으로 포함**해 줘.

          예시:

            ```html
            <a th:href="@{/items}">상품목록</a>
            
            ```

            - 만약 애플리케이션이 `/myapp` 에 배포돼 있다면,

              👉 최종 HTML 결과는

                ```html
                <a href="/myapp/items">상품목록</a>
                
                ```

              로 변환됨.

            - 즉, `@{/items}` → `/items` 가 아니라 `/myapp/items` 로 만들어 줘서,

              애플리케이션이 어디에 배포돼 있든 링크가 항상 올바르게 동작하게 해줌.


            ---
            
            ### 3. 왜 중요한가?
            
            - 개발 환경에서는 보통 루트(`/`)에 배포하지만, 운영 환경에서는 `/myapp` 같이 하위 폴더에 배포하는 경우가 많아.
            - 이때 `@{...}` 를 쓰면 URL 경로를 자동으로 맞춰주므로, 개발자가 배포 경로에 신경 쓸 필요가 없어져.
            
            ---
            
            ✅ **정리**
            
            - **서블릿 컨텍스트(Servlet Context)** = 애플리케이션의 기본 경로(context path).
            - 타임리프의 `@{...}` 는 이 경로를 자동으로 붙여서 URL을 생성해 줌.
    - 예) th:href="@{/css/bootstrap.min.css}"
    - URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다.
    - 경로 변수( {itemId} ) 뿐만 아니라 **쿼리 파라미터**도 생성한다.
    예) th:href="@{/basic/items/{itemId}(itemId=${[item.id](http://item.id/)}, query='test')}"
    생성 링크: http://localhost:8080/basic/items/1?query=test
    - th:href="@{|/basic/items/${[item.id](http://item.id/)}|}"
    리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.
- |...|
    - 리터럴을 대체한다.
    - 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.
        - 예) <span th:text="'Welcome to our application, ' + ${[user.name](http://user.name/)} + '!'">
        - `${...}` = 변수 표현식
    - 하지만 다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다.
      <span th:text="|Welcome to our application, ${[user.name](http://user.name/)}!|">
- th:each
    - 반복 출력
    - 예) `<tr th:each="item : ${items}">`
    - 이렇게 하면 모델에 포함된 items 컬렉션 데이터가 item 변수에 하나씩 포함되고, 반복문 안에서 item 변수를 사용할 수 있다.
    - 컬렉션의 수 만큼 <tr>..</tr> 이 하위 태그를 포함해서 생성된다.
- ${...}
    - 예) `<td th:text="${item.price}">10000</td>`
    - 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다.
    - 프로퍼티 접근법을 사용한다. ( item.getPrice() )
- th:text
    - 내용의 값을 th:text 의 값으로 변경한다.
    - 예) `<td th:text="${item.price}">10000</td>`
    - 여기서는 10000을 ${item.price} 의 값으로 변경한다.
- 네츄럴 템플릿(natural templates)
    - = 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징
- th:value
    - 모델에 있는 item 정보를 획득하고 프로퍼티 접근법으로 출력한다. ( item.getId() )
    - value 속성을 th:value 속성으로 변경한다.
- th:action
    - HTML form에서 action 에 값이 없으면 현재 URL에 데이터를 전송한다.
    - 예) `<form th:action="@{/items}" method="post">`
    - 상품 등록 폼의 URL과 실제 상품 등록을 처리하는 URL을 똑같이 맞추고 HTTP 메서드로 두 기능을 구분한다.
        - 상품 등록 폼: GET /basic/items/add
        - 상품 등록 처리: POST /basic/items/add
    - 이렇게 하면 하나의 URL로 등록 폼과, 등록 처리를 깔끔하게 처리할 수 있다.

- POST - HTML Form 이 서버에 데이터를 전달하는 방식은?
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파리미터 형식으로 전달 → `itemName=itemA&price=10000&quantity=10`
    - 예) 회원 가입, 상품 주문, HTML Form 사용

- @RequestParam 으로 변수를 하나하나 받아서 Item 을 생성하는 과정은 불편하다. 어떻게 하면 더 편리하게 코딩할 수 있나?
    - `@ModelAttribute("item") Item item` 이를 파라미터 안에 넣으면 된다.
    - 그러면 Item 객체를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXxx)으로 입력해준다. 즉, `model.addAttribute("item", item)` 이 자동으로 일어난다.

- @ModelAttribute("hello") Item item → 이렇게 이름을 hello 로 지정하면 모델은 어떤 이름으로 저장되는가?
    - `model.addAttribute("hello", item);` 가 되어
    - 모델에 hello 이름으로 저장된다.
- @ModelAttribute 의 이름을 생략하면?
    - 모델에 저장될 때 클래스명을 사용한다.
    - 이때 클래스의 첫글자만 소문자로 변경해서 등록한다.
    - 예) `@ModelAttribute Item item`
    - model.addAttribute(item); 자동 추가, 생략 가능
    - 생략 시 model에 저장되는 name은 클래스명 첫글자만 소문자로 등록 Item → item
    - @ModelAttribute 자체도 생략가능하다. 대상 객체는 모델에 자동 등록된다.

- redirect:/…
    - 스프링은 `redirect:/…` 으로 편리하게 리다이렉트를 지원한다.
    - 컨트롤러에 매핑된 @PathVariable 의 값은 redirect 에도 사용 할 수 있다.
    - 예) `redirect:/basic/items/{itemId}`
    - → {itemId} 는 @PathVariable Long itemId 의 값을 그대로 사용한다

- 상품 등록을 완료하고 웹 브라우저의 새로고침 버튼을 클릭하였을 때 상품이 계속해서 중복 등록된다. 그 이유는? 이 문제를 해결하기 위한 방법도 말하시오.
    - 웹 브라우저의 새로 고침은 **마지막에 서버를 전송한 데이터를 다시 전송**하기 때문이다.
    - 해결 방법: PRG - Post, Redirect, Get
        - 상품 저장 후에 뷰 템플릿으로 이동하는 것이 아니라, 상품 상세 화면으로 리다이렉트를 호출해주면 된다.
        - 그러면 마지막에 호출한 내용이 상품 상세 화면인 GET /items/{id} 가 되는 것이다.
        - 이후 새로고침을 해도 상품 상세 화면으로 이동하게 되므로 새로 고침 문제를 해결할 수 있다.

- `"redirect:/basic/items/" + item.getId()` 의 문제점은? 해결 방법은?
    - URL에는 공백, 한글, 특수문자 같은 걸 그대로 넣으면 깨지거나 잘못 해석된다.
    - 브라우저와 서버가 이해할 수 있도록 안전한 형태(ASCII 문자)로 변환해주는 URL 인코딩을 해주어야 한다. → RedirectAttributes 사용

    ```java
    @PostMapping("/add")
    public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
     Item savedItem = itemRepository.save(item);
     redirectAttributes.addAttribute("itemId", savedItem.getId());
     redirectAttributes.addAttribute("status", true);
     // status=true 를 추가해보자. 그리고 뷰 템플릿에서 이 값이 있으면, 저장되었습니다. 라는 메시지를 출력
     return "redirect:/basic/items/{itemId}";
    }
    ```

- 실행해보면 다음과 같은 리다이렉트 결과가 나온다.
  http://localhost:8080/basic/items/3?status=true
- 즉, RedirectAttributes 를 사용하면 URL 인코딩도 해주고, pathVariable , 쿼리 파라미터까지 처리해준다.

```html
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 상세</h2>
    </div>
    <!-- 추가 -->
    <h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>
</div>
```

- th:if
    - 해당 조건이 참이면 실행
- ${param.status}
    - 타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능
    - 원래는 컨트롤러에서 모델에 직접 담고 값을 꺼내야 한다.
    - 그런데 쿼리 파라미터는 자주 사용해서 타임리프에서 직접 지원한다