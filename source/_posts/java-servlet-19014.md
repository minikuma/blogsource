---
title: '[java] Servlet'
tags: [java, servlet]
---
이번 포스팅은 Servlet에 대해 알아보려고 한다. 필자도 Servlet을 WAS(Web Application Server) 혹은 Container 등으로 막연하게 알고 있었다. 이번 기회에 Servlet에 대해 정리를 해 보려고 한다.   

## Goal   
1. Servlet Background & Definition
2. Servlet Process
3. Servlet Life Cycle
4. Servlet Example in java
5. Servlet Concurrency
6. Apply Servlet Annotation   

### Servlet Background & Definition   
Servlet은 Web Programming을 java를 이용하여 구현하기 위해 탄생하였다. 그래서 어떤 사람은 Servlet을 CGI(Common Gateway Interface) Program이라고 부르기도 한다. Servlet은 HTTP Protocol을 지원하는 `javax.servlet.http.HttpServlet` 클래스를 상속하여 기능을 구현할 수 있다.   

### Servlet Process
![](https://user-images.githubusercontent.com/20740884/51011860-a0014f80-159d-11e9-81ec-24ff54f66018.JPG)   
1) Web Server는 클라이언트로부터 받은 HTTP 요청을 Web Container(Servlet Container)에게 위임한다.    
- Web Container는 먼저 `web.xml` 파일을 확인한다.   
(어떤 URL로 Mapping해야 할지, 어떤 Servlet을 Container에 등록해야 할지)   

2) Web Container는 `web.xml`에 명시되어 있는 Servlet 객체를 찾아 해당 객체를 메모리에 로딩한다.   
- Web Container는 적당한 Servlet을 찾아 컴파일(.class)한다.   
- 메모리 로딩 시, `init()` 메소드를 실행하여 객체를 초기화 한다.   

3) Web Container는 클라이언트로부터 HTTP 요청을 받을 때마다 Thread를 생성한다.    
- 여러 개의 Thread가 생성된다.   
- 각 Thread는 단일 객체에 대해 Servlet `service()` 메소드를 실행한다.   
참고적으로 Web Container가 생성하는 Thread의 역할은 클라이언트로부터 요청된 HTTP 타입(GET/POST/PUT/DELETE 등)에 따라 (doGet/doPost/doPut/doDelete 등)을 호출한다. 메소드에 응답을 받게 되면 Thread는 소멸되거나, Thread Poll에 자원을 반납한다.   

**결론적으로 WAS(Web Application Server)는 Web Container + Servlet으로 볼 수 있다.**   

### Servlet Life Cycle
![](https://user-images.githubusercontent.com/20740884/51012604-5581d200-15a1-11e9-8256-67cff7c719d8.JPG)   
1) `init()` 메소드   
- 한번 만 실행된다.   
- 클라이언트 요청에 따라 적절한 Servlet이 생생되고 메모리에 로딩될 때 실행된다.   
- 객체를 초기화한다.
간혹 Servlet 객체가 초기화 되지 않았다는 에러를 만날수 있는데, 보통 Servlet이 초기화 되지 않았다는 의미는 1) 초기화 되는 중(생성자 실행 중, `init()`메소드 실행 중) 2) 소멸되는 중(`destroy()`메소드 실행 중) 3) 존재하지 않음(does not exist)으로 볼 수 있다.   

2) `service()` 메소드   
- Servlet이 수신한 모든 요청에 대해 실행된다.(각 Thread 당)   
- 클라이언트로부터 수신된 HTTP 타입(GET/GET/DELETE 등)에 따라 적절한 메소드(doGet/doPost/doDelete)가 호출된다.   
- 호출된 메소드에 대한 응답을 받으면 해당 Thread는 소멸 혹은 Thread Pool에 자원을 반납한다.   

3) `destroy()` 메소드   
- 한번 만 실행된다.   
- 메모리에 로딩된 Servlet 객체를 제거한다.   

### Servlet Example in java   
간단한 예제를 만들어 보려고 한다. 클라이언트로부터 요청받은 Address, Host, URL정보를 Web상에 출력하는 예제이다. 예제를 구현하기 위해 먼저 `javax.servlet.http.HttpServlet` 클래스를 상속받고 여러 메소드 중에 doGet() 메소드를 재 정의하여 사용하였다.   

``` java
ExampleServlet.java
public class ExampleServlet extends HttpServlet {
    //doGet 재 정의
    @Override protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html; charset=UTF-8");
        // Response 객체의 PrintWriter를 사용해 브라우저에 HTML을 출력한다.
        PrintWriter out = resp.getWriter();
        out.println("<HTML><HEAD><TITLE>HelloServlet</TITLE></HEAD>");
        out.println("<BODY>");
        out.println("<H2> Clinet IP: " + req.getRemoteAddr() + "</H2>");
        out.println("<H2> Client Host : " + req.getRemoteHost() + "</H2>");
        out.println("<H2> Request URI : " + req.getRequestURI() + "</H2>");
        out.println("</BODY></HTML>");
    }
}
```
```xml
web.xml
<servlet> <servlet-name>hello</servlet-name>
    <servlet-class>me.spring5.mvc.ExampleServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```   
Servlet name은 `hello`, Servlet URL Mapping은 `/` ROOT URL로 정의하였다.   

```java
http://localhost:8080/
```   
![](https://user-images.githubusercontent.com/20740884/51013619-ec04c200-15a6-11e9-9e9c-febad017e03f.JPG)   

### Servlet Concurrency   
![](https://user-images.githubusercontent.com/20740884/51014652-3a688f80-15ac-11e9-85c6-139c57b0b540.JPG)
보통 Web Application은 Multi-Thread 환경이고, 여러 Thread는 하나의 Servlet을 공유할 수 있기 때문에 병행성 제어(Concurrency Control)가 필요하다. 위 그림에서 보면 static, member 변수는 공유자원이기 때문에 `service()` 메소드안에서 사용 시에 상호 배제가 필요하다. 상호 배제란 공유를 하면 안되는 자원(Resource)의 동시 사용을 피하는 방법이다. 추후에 상호 배제에 대한 내용을 공부하면서 추가 포스팅을 할 예정이다.      

### Apply Servlet Annotation
기존 web.xml에서 정의한 servlet class, URL Mapping을 아래와 같이 `@WebServlet` Annotation으로도 구현이 가능하다.   

``` xml
web.xml
<servlet> <servlet-name>hello</servlet-name>
    <servlet-class>me.spring5.mvc.ExampleServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```   

``` java
ExampleServlet.java
@WebServlet("/")
public class ExampleServlet extends HttpServlet {
    //doGet 재 정의
    @Override protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html; charset=UTF-8");
        // Response 객체의 PrintWriter를 사용해 브라우저에 HTML을 출력한다.
        PrintWriter out = resp.getWriter();
        out.println("<HTML><HEAD><TITLE>HelloServlet</TITLE></HEAD>");
        out.println("<BODY>");
        out.println("<H2> Clinet IP: " + req.getRemoteAddr() + "</H2>");
        out.println("<H2> Client Host : " + req.getRemoteHost() + "</H2>");
        out.println("<H2> Request URI : " + req.getRequestURI() + "</H2>");
        out.println("</BODY></HTML>");
    }
}
```   
해당 annotation에 대해 부연 설명을 하자만 Servlet 3.0+ API에서 `javax.servlet.annotaion` Package를 도입하였고, 해당 Package는 Tomcat 7.0+에서 사용가능하다. Servlet을 선언하는 `@WebServlet`과 초기화 매개변수를 지정할 수 있는 `@WebInitParam` annotation type을 지원한다.   
