# 스프링 MVC 학습 내용 정리(#1서블릿편)
> 김영한 강사님의 스프링 MVC 1편의 내용을 정리한 문서입니다.

***
### 1. 서블릿
#### 1.1. 스프링 부트 서블릿 환경 구성
```java
@ServletComponentScan //서블릿 자동 등록
@SpringBootApplication
public class ServletApplication {
  public static void main(String[] args) {
    SpringApplication.run(ServletApplication.class, args);
  }
}
```
* 스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 `@ServletComponentScan` 을 지원한다

#### 1.2. 서블릿 등록
```java
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
* `@WebServlet` 서블릿 애노테이션
  + name: 서블릿 이름
  + urlPatterns: URL 매핑
* HTTP요청을 통해 매핑된 URL이 호출되면 `protected void service(HttpServletRequest request, HttpServletResponse response)`를 호출한다.

#### 1.3. 서블릿 컨테이너 동작 방식
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/13fe4f76-25ed-480c-9f32-4c2c40634cfd)
* 스프링 부트에서 내장 톰켓 서버 생성

![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/376762b0-95bc-47f2-a612-a18a96663fc5)
* HTTP 요청 메세지를 기반으로 Request 생성후 서블릿 컨테이너에 전달
* Response 객체 정보로 HTTP 응답

#### 1.4. HttpServletRequest-개요
* 기본적으로 HTTP 요청 메시지는 아래와 같이 되있다.
* **HTTP 요청 메시지**
```
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
username=kim&age=20
```
* 개발자는 이 정보들을 직접 파싱해서 사용하기 힘들기 때문에, 서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할수 있도록 HTTP 요청 메시지를 대신 파싱한다.
* HttpServletRequest 객체에 담아서 제공한다.

#### 1.5. HttpServletRequest-기본사용법
```java
@WebServlet(name = "requestHeaderServlet", urlPatterns = "/request-header")
public class RequestHeaderServlet extends HttpServlet {
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    printStartLine(request);
    printHeaders(request);
    printHeaderUtils(request);
    printEtc(request);
    response.getWriter().write("ok");
  }
}
```
* 기본틀

```java
//start line 정보
private void printStartLine(HttpServletRequest request) {
  System.out.println("--- REQUEST-LINE - start ---");
  System.out.println("request.getMethod() = " + request.getMethod()); //GET
  System.out.println("request.getProtocol() = " + request.getProtocol()); //HTTP/1.1
  System.out.println("request.getScheme() = " + request.getScheme()); //http // http://localhost:8080/request-header
  System.out.println("request.getRequestURL() = " + request.getRequestURL()); // /request-header
  System.out.println("request.getRequestURI() = " + request.getRequestURI()); //username=hi
  System.out.println("request.getQueryString() = " +request.getQueryString());
  System.out.println("request.isSecure() = " + request.isSecure()); //https 사용 유무
  System.out.println("--- REQUEST-LINE - end ---");
}
```
* start-line 정보 출력법

```java
//Header 모든 정보
private void printHeaders(HttpServletRequest request) {
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
```
* 헤더 정보 출력

```java
private void printHeaderUtils(HttpServletRequest request) {
  System.out.println("--- Header 편의 조회 start ---");

  System.out.println("[Host 편의 조회]");
  System.out.println("request.getServerName() = " + request.getServerName()); //Host 헤더
  System.out.println("request.getServerPort() = " + request.getServerPort()); //Host 헤더
  
  System.out.println("[Accept-Language 편의 조회]");
  request.getLocales().asIterator()
          .forEachRemaining(locale -> System.out.println("locale = " + locale));
  System.out.println("request.getLocale() = " + request.getLocale());
  
  System.out.println("[cookie 편의 조회]");
  if (request.getCookies() != null) {
    for (Cookie cookie : request.getCookies()) {
      System.out.println(cookie.getName() + ": " + cookie.getValue());
    }
  }
  
  System.out.println("[Content 편의 조회]");
  System.out.println("request.getContentType() = " + request.getContentType());
  System.out.println("request.getContentLength() = " + request.getContentLength());
  System.out.println("request.getCharacterEncoding() = " + request.getCharacterEncoding());

  System.out.println("--- Header 편의 조회 end ---");
}
```
* 헤더 편의 조회 출력

```java
//기타 정보
private void printEtc(HttpServletRequest request) {
  System.out.println("--- 기타 조회 start ---");

  System.out.println("[Remote 정보]");
  System.out.println("request.getRemoteHost() = " + request.getRemoteHost());
  System.out.println("request.getRemoteAddr() = " + request.getRemoteAddr());
  System.out.println("request.getRemotePort() = " + request.getRemotePort());
  
  System.out.println("[Local 정보]");
  System.out.println("request.getLocalName() = " + request.getLocalName());
  System.out.println("request.getLocalAddr() = " + request.getLocalAddr());
  System.out.println("request.getLocalPort() = " + request.getLocalPort());

  System.out.println("--- 기타 조회 end ---");
}
```
* HTTP 정보는 아니지만 기타 정보 또한 출력할 수 있다.
> * 로컬에서 테스트 하면 IPv6 정보가 나온다, IPv4의 정보를 보고 싶다면 아래 옵션을 VM options에 넣어주면 된다
> * -Djava.net.preferIPv4Stack=true


### 1.6. HTTP 요청 데이터-개요
* HTTP Request를 통해 클라이언트에서 서버로 데이터를 전달하는 방법은 주로 아래와 같은 3가지 방법을 이용한다
#### 1.6.1. GET - query parameter
* /url**?username=hello&age=20**
* 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
* 예) 검색, 필터, 페이징등에서 많이 사용하는 방식

#### 1.6.2. POST - HTML Form
* content-type: application/x-www-form-urlencoded
* 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
* 예) 회원 가입, 상품 주문, HTML Form 사용

#### 1.6.3. HTTP message body에 데이터를 직접 담아서 요청
* HTTP API에서 주로 사용, JSON, XML, TEXT
* 데이터 형식은 주로 JSON 사용
  + POST, PUT, PATCH

### 1.7. HTTP 요청 데이터 - GET Query Parameter
* 아래와 같이 URL와 ?를 시작으로 보내는 방법이다. 추가 파라미터는 &로 구분하면 된다.
* 예) `http://localhost:8080/request-param?username=hello&age=20`

```java
String username = request.getParameter("username"); //단일 파라미터 조회
Enumeration<String> parameterNames = request.getParameterNames(); //파라미터 이름들 모두 조회
Map<String, String[]> parameterMap = request.getParameterMap(); //파라미터를 Map으로 조회
String[] usernames = request.getParameterValues("username"); //복수 파라미터 조회
```
* Query Parameter 조회 메서드

### 1.8. HTTP 요청 데이터 - POST HTML Form
* content-type: `applicatioin/x-www-form-urlencoded`
* 메시지 바디에 Query Parameter 형식으로 데이터를 전달한다. 예)`username=hello&age=20`
```html
<!DOCTYPE html>
<html>
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
    <form action="/request-param" method="post">
      username: <input type="text" name="username" />
      age: <input type="text" name="age" />
      <button type="submit">전송</button>
    </form>
  </body>
</html>
```
* POST의 HTML Form을 전송하면 웹브라우저는 밑의 형식으로 HTTP 메시지를 만든다.
* **요청 URL**: http://localhost:8080/request-param
* **content-type**: `application/x-www-form-urlencoded`
* **message body**: `username=hello&age=20
* 고도로 발달한 POST의 HTML Form은 앞에서 본 GET 쿼리 파라미터 형식과 구분할 수없기 때문에 쿼리 파라미터 조회 메서드를 그대로 사용하면 된다.

### 1.9. HTTP 요청 데이터 - API 메세지 바디 - 단순 텍스트
* HTTP message body**에 데이터를 직접 담아서 요청
  + HTTP API에서 주로 사용, JSON, XML, TEXT
  + 데이터 형식은 주로 JSON 사용
  + POST, PUT, PATCH
* 먼저 가장 단순한 텍스트 메시지를 HTTP 메시지 바디에 담아서 전송하고, 읽어보자.
* HTTP 메시지 바디의 데이터를 InputStream을 사용해서 직접 읽을 수 있다.
```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-bodystring")
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

* POST http://localhost:8080/request-body-string
* content-type: text/plain
* message body: `hello`
* 결과: `messageBody = hello`

### 1.10. HTTP 요청 데이터 - API 메시지 바디 - JSON

```java
@Getter @Setter
public class HelloData {
  private String username;
  private int age;
}
```
* JSON 형식 파싱을 위한 객체
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
      System.out.println("helloData.username = " + helloData.getUsername());
      System.out.println("helloData.age = " + helloData.getAge());
      response.getWriter().write("ok");
  }
}
```
* POST http://localhost:8080/request-body-json
* content-type: **application/json**
* message body: `{"username": "hello", "age": 20}`
> JSON 결과를 파싱하는 기능은 Jackson, Gson 같은 JSON 변환 라이브러리를 추가해서 사용해야하지만, 스프링 부트로 Spring MVC를 선택하면 Jackson 라이브러리 ObjectMapper를 함께 제공한다.

### 1.11. HttpServletResponse - 기본 사용법
#### 1.11.1. HttpServletResponse 역할
* HTTP 응답 메시지 생성
    + HTTP 응답 코드 지정
    + 헤더 생성
    + 바디 생성
* 편의 기능 제공
    + Content-Type, 쿠키, Redirect
#### 1.11.2. HttpServletResponse - 기본 사용법
```java
@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
public class ResponseHeaderServlet extends HttpServlet {
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //[status-line]
    response.setStatus(HttpServletResponse.SC_OK); //200

    //[response-headers]
    response.setHeader("Content-Type", "text/plain;charset=utf-8");
    response.setHeader("Cache-Control", "no-cache, no-store, mustrevalidate");
    response.setHeader("Pragma", "no-cache");
    response.setHeader("my-header","hello");

    //[Header 편의 메서드] 밑에서 다시 설명
    content(response);
    cookie(response);
    redirect(response);

    //[message body]
    PrintWriter writer = response.getWriter();
    writer.println("ok");
  }
}
```
* status-line 설정
* 헤더 설정
* 메시지 바디 설정
```java
private void content(HttpServletResponse response) {
  //Content-Type: text/plain;charset=utf-8
  //Content-Length: 2
  //response.setHeader("Content-Type", "text/plain;charset=utf-8");
  response.setContentType("text/plain");
  response.setCharacterEncoding("utf-8");
  //response.setContentLength(2); //(생략시 자동 생성)
}
```
* Content 편의 메서드
```java
private void cookie(HttpServletResponse response) {
  //Set-Cookie: myCookie=good; Max-Age=600;
  //response.setHeader("Set-Cookie", "myCookie=good; Max-Age=600");
  Cookie cookie = new Cookie("myCookie", "good");
  cookie.setMaxAge(600); //600초
  response.addCookie(cookie);
}
```
* 쿠키 편의 메서드
```java
private void redirect(HttpServletResponse response) throws IOException {
  //Status Code 302
  //Location: /basic/hello-form.html
  //response.setStatus(HttpServletResponse.SC_FOUND); //302
  //response.setHeader("Location", "/basic/hello-form.html");
  response.sendRedirect("/basic/hello-form.html");
}
```
* redirect 편의 메서드

### 1.12. HTTP 응답 데이터 - 단순텍스트,HTML
```java
@WebServlet(name = "responseHtmlServlet", urlPatterns = "/response-html")
public class ResponseHtmlServlet extends HttpServlet {
@Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //Content-Type: text/html;charset=utf-8
    response.setContentType("text/html");
    response.setCharacterEncoding("utf-8");
    PrintWriter writer = response.getWriter();
    writer.println("<html>");
    writer.println("<body>");
    writer.println(" <div>안녕?</div>");
    writer.println("</body>");
    writer.println("</html>");
  }
}
```
* HttpServletResponse - HTML 응답
* HTTP 응답으로 HTML을 반환할 때는 content-type을 text/html로 지정해야한다.

### 1.13. HTTP 응답 데이터 - API Json
```java
@WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")
public class ResponseJsonServlet extends HttpServlet {
  private ObjectMapper objectMapper = new ObjectMapper();
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //Content-Type: application/json
    response.setHeader("content-type", "application/json");
    response.setCharacterEncoding("utf-8");

    HelloData data = new HelloData();
    data.setUsername("kim");
    data.setAge(20);

    //{"username":"kim","age":20}
    String result = objectMapper.writeValueAsString(data);
    response.getWriter().write(result);
  }
}
```
* HTTP 응답으로 JSON을 반환 할 때는 content-type을 application/json로 지정해야한다.
* Jackson 라이브러리가 제공하는 objectMapper.writeValueAsString()을 사용하면 객체를 JSON 문자로 변경할 수 있다.
