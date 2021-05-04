# 5. 스프링 MVC - 구조 이해

## 스프링 MVC 전체 구조

---

- 직접 만든 MVC 프레임워크 구조

![5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20MVC%20-%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%20%E1%84%8B%E1%85%B5%E1%84%92%E1%85%A2%20d5ef6f2ac93b44fea33ee5a360220dc1/Untitled.png](5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20MVC%20-%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%20%E1%84%8B%E1%85%B5%E1%84%92%E1%85%A2%20d5ef6f2ac93b44fea33ee5a360220dc1/Untitled.png)

- SpringMVC 구조

![5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20MVC%20-%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%20%E1%84%8B%E1%85%B5%E1%84%92%E1%85%A2%20d5ef6f2ac93b44fea33ee5a360220dc1/Untitled%201.png](5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20MVC%20-%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%20%E1%84%8B%E1%85%B5%E1%84%92%E1%85%A2%20d5ef6f2ac93b44fea33ee5a360220dc1/Untitled%201.png)

- **직접 만든 프레임워크 스프링 MVC 비교**
    - FrontController → DispatcherServlet
    - handlerMappingMap → HandlerMapping
    - MyHandlerAdapter → HandlerAdapter
    - ModelView → ModelAndView
    - viewResolver → ViewResolver
    - MyView → View

### DispatcherServlet 구조 살펴보기

---

- `org.springframework.web.servlet.DispatcherServlet`
    - 스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다.
    - 프론트 컨트롤러가 바로 디스패처 서블릿(DispatcherServlet)
- DispacherServlet 서블릿 등록
    - `DispacherServlet`도 부모 클래스에서 `HttpServlet`을 상속받아서 사용하고, 서블릿으로 동작
        - DispatcherServlet → FrameworkServlet → HttpServletBean → HttpServlet
    - 스프링 부트는 DispacherServlet 을 서블릿으로 자동으로 등록하면서 모든 경로(`urlPatterns="/"`)에 대해서 매핑한다.
        - 참고: 더 자세한 경로가 우선순위가 높다. 그래서 기존에 등록한 서블릿도 함께 동작한다.
- 요청 흐름
    - 서블릿이 호출되면 `HttpServlet`이 제공하는 `serivce()`가 호출된다.
    - 스프링 MVC는 `DispatcherServlet`의 부모인 `FrameworkServlet`에서 `service()`를 오버라이드 해두었다.
    - `FrameworkServlet.service()`를 시작으로 여러 메서드가 호출되면서 `DispacherServlet.doDispatch()`가 호출된다.

- `DispacherServlet.doDispatch()`

    ```java
    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
      HttpServletRequest processedRequest = request;
      HandlerExecutionChain mappedHandler = null;
      ModelAndView mv = null;

    	// 1. 핸들러 조회
    	mappedHandler = getHandler(processedRequest);
      if (mappedHandler == null) {
    		noHandlerFound(processedRequest, response);
    		return;
    	}

    	//2.핸들러 어댑터 조회-핸들러를 처리할 수 있는 어댑터
    	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

    	// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
    	mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
      
    	processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);

    }

    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
    	// 뷰 렌더링 호출
    	render(mv, request, response);
    }

    protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    	View view;
    	String viewName = mv.getViewName();

    	//6. 뷰 리졸버를 통해서 뷰 찾기, 7.View 반환
    	view = resolveViewName(viewName, mv.getModelInternal(), locale, request);

    	// 8. 뷰 렌더링
    	view.render(mv.getModelInternal(), request, response);
    }
    ```

- SpringMVC 동작 순서
    1. **핸들러 조회**: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
    2. **핸들러 어댑터 조회**: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
    3. **핸들러 어댑터 실행**: 핸들러 어댑터를 실행한다.
    4. **핸들러 실행**: 핸들러 어댑터가 실제 핸들러를 실행한다.
    5. **ModelAndView 반환**: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 **변환**해서 반환한다.
    6. **viewResolver 호출**: 뷰 리졸버를 찾고 실행한다.

        JSP의 경우: `InternalResourceViewResolver`가 자동 등록되고, 사용된다.

    7. **View반환**: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.

        JSP의 경우 `InternalResourceView(JstlView)`를 반환하는데, 내부에 `forward()`로직이 있다.

    8. **뷰 렌더링**: 뷰를 통해서 뷰를 렌더링 한다.