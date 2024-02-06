# 스프링 MVC 1편 학습 내용 정리
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


#### 1.6. HTTP 요청 데이터-개요
* HTTP Request를 통해 클라이언트에서 서버로 데이터를 전달하는 방법은 주로 아래와 같은 3가지 방법을 이용한다
##### 1.6.1. GET - query parameter
* /url**?username=hello&age=20**
* 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
* 예) 검색, 필터, 페이징등에서 많이 사용하는 방식

##### 1.6.2. POST - HTML Form
* content-type: application/x-www-form-urlencoded
* 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
* 예) 회원 가입, 상품 주문, HTML Form 사용

##### 1.6.3. HTTP message body에 데이터를 직접 담아서 요청
* HTTP API에서 주로 사용, JSON, XML, TEXT
* 데이터 형식은 주로 JSON 사용
  + POST, PUT, PATCH

#### 1.7. HTTP 요청 데이터 - GET Query Parameter
* 아래와 같이 URL와 ?를 시작으로 보내는 방법이다. 추가 파라미터는 &로 구분하면 된다.
* 예) `http://localhost:8080/request-param?username=hello&age=20`

```java
String username = request.getParameter("username"); //단일 파라미터 조회
Enumeration<String> parameterNames = request.getParameterNames(); //파라미터 이름들 모두 조회
Map<String, String[]> parameterMap = request.getParameterMap(); //파라미터를 Map으로 조회
String[] usernames = request.getParameterValues("username"); //복수 파라미터 조회
```
* Query Parameter 조회 메서드

#### 1.8. HTTP 요청 데이터 - POST HTML Form
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

#### 1.9. HTTP 요청 데이터 - API 메세지 바디 - 단순 텍스트
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

#### 1.10. HTTP 요청 데이터 - API 메시지 바디 - JSON

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

#### 1.11. HttpServletResponse - 기본 사용법
##### 1.11.1. HttpServletResponse 역할
* HTTP 응답 메시지 생성
    + HTTP 응답 코드 지정
    + 헤더 생성
    + 바디 생성
* 편의 기능 제공
    + Content-Type, 쿠키, Redirect
##### 1.11.2. HttpServletResponse - 기본 사용법
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

#### 1.12. HTTP 응답 데이터 - 단순텍스트,HTML
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

#### 1.13. HTTP 응답 데이터 - API Json
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

***
### 2. 서블릿->JSP->MVC 패턴 변화
#### 2.1. 간단한 회원 관리 웹 애플리케이션
* **회원 정보**
  + 이름: `username`
  + 나이: `age`
* **기능 요구사항**
  + 회원 저장
  + 회원 목록 조회
```java
@Getter @Setter
@AlrgConstructor
public class Member {
  private Long id;
  private String username;
  private int age;
}
```
```java
public class MemberRepository {
  private static Map<Long, Member> store = new HashMap<>(); //static 사용
  private static long sequence = 0L; //static 사용
  private static final MemberRepository instance = new MemberRepository();
  // 중략
  public Member save(Member member) {
    member.setId(++sequence);
    store.put(member.getId(), member);
    return member;
  }
  public Member findById(Long id) {
    return store.get(id);
  }
  public List<Member> findAll() {
    return new ArrayList<>(store.values());
  }
}
```

#### 2.2. 서블릿으로 코드 만들기
```java
@WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/newform")
public class MemberFormServlet extends HttpServlet {
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    response.setContentType("text/html");
    response.setCharacterEncoding("utf-8");

    PrintWriter w = response.getWriter();
    w.write("<!DOCTYPE html>\n" +
            "<html>\n" +
            "<head>\n" +
            " <meta charset=\"UTF-8\">\n" +
            " <title>Title</title>\n" +
            "</head>\n" +
            "<body>\n" +
            "<form action=\"/servlet/members/save\" method=\"post\">\n" +
            " username: <input type=\"text\" name=\"username\" />\n" +
            " age: <input type=\"text\" name=\"age\" />\n" +
            " <button type=\"submit\">전송</button>\n" +
            "</form>\n" +
            "</body>\n" +
            "</html>\n");
  }
}
```
* localhost:8080/servlet/members/newform 요청시 회원 정보를 입력할 수 있는 HTML Form을 만들어서 응답한다.
```java
@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    System.out.println("MemberSaveServlet.service");
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    Member member = new Member(username, age);
    System.out.println("member = " + member);
    memberRepository.save(member);

    response.setContentType("text/html");
    response.setCharacterEncoding("utf-8");

    PrintWriter w = response.getWriter();
    w.write("<html>\n" +
            "<head>\n" +
            " <meta charset=\"UTF-8\">\n" +
            "</head>\n" +
            "<body>\n" +
            "성공\n" +
            "<ul>\n" +
            " <li>id="+member.getId()+"</li>\n" +
            " <li>username="+member.getUsername()+"</li>\n" +
            " <li>age="+member.getAge()+"</li>\n" +
            "</ul>\n" +
            "<a href=\"/index.html\">메인</a>\n" +
            "</body>\n" +
            "</html>");
  }
}
```
* localhost:8080/servlet/members/save?usernmae=dd&?age=30 요청시 데이터를 가져와 Member 객체를 만들고 MemberRepository에 저장한다.

```java
@WebServlet(name = "memberListServlet", urlPatterns = "/servlet/members")
public class MemberListServlet extends HttpServlet {
private MemberRepository memberRepository = MemberRepository.getInstance();
@Override
protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  response.setContentType("text/html");
  response.setCharacterEncoding("utf-8");
  List<Member> members = memberRepository.findAll();
  PrintWriter w = response.getWriter();
  w.write("<html>");
  w.write("<head>");
  w.write(" <meta charset=\"UTF-8\">");
  w.write(" <title>Title</title>");
  w.write("</head>");
  w.write("<body>");
  w.write("<a href=\"/index.html\">메인</a>");
  w.write("<table>");
  w.write(" <thead>");
  w.write(" <th>id</th>");
  w.write(" <th>username</th>");
  w.write(" <th>age</th>");
  w.write(" </thead>");
  w.write(" <tbody>");
  for (Member member : members) {
    w.write(" <tr>");
    w.write(" <td>" + member.getId() + "</td>");
    w.write(" <td>" + member.getUsername() + "</td>");
    w.write(" <td>" + member.getAge() + "</td>");
    w.write(" </tr>");
  }
  w.write(" </tbody>");
  w.write("</table>");
  w.write("</body>");
  w.write("</html>");
  }
}
```
* localhost:8080/servlet/members 요청시 for 루프를 통해 회원 수 만큼 동적으로 생성하고 응답한다.

#### 2.3. JSP로 변경
##### 2.3.1 JSP 코드 작성성
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>Title</title>
  </head>
  <body>
    <form action="/jsp/members/save.jsp" method="post">
    username: <input type="text" name="username" />
    age: <input type="text" name="age" />
    <button type="submit">전송</button>
    </form>
  </body>
</html>
```
* main/webapp/jsp/members/new-form.jsp 파일 위와 같이 생생성
* `<%@ page contentType="text/html;charset=UTF-8" language="java" %>`
  + JSP문서라는 표시

```html
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
  // request, response 사용 가능
  MemberRepository memberRepository = MemberRepository.getInstance();
  System.out.println("save.jsp");
  String username = request.getParameter("username");
  int age = Integer.parseInt(request.getParameter("age"));
  Member member = new Member(username, age);
  System.out.println("member = " + member);
  memberRepository.save(member);
%>
<html>
  <head>
    <meta charset="UTF-8">
  </head>
  <body>
    <ul>
        <li>id=<%=member.getId()%></li>
        <li>username=<%=member.getUsername()%></li>
        <li>age=<%=member.getAge()%></li>
    </ul>
    <a href="/index.html">메인</a>
  </body>
</html>
```
* `main/webapp/jsp/members/save.jsp` 생성 , 회원 저장 기능
* `<%@ page import="hello.servlet.domain.member.MemberRepository" %>`
  + 자바의 import 문과 같다.
* `<% ~~ %>`
  + 이 부분에는 자바 코드를 입력할 수 있다.
* `<%= ~~ %>`
  + 이부분에는 자바 코드를 출력할 수 있다.
 
```html
<%@ page import="java.util.List" %>
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
MemberRepository memberRepository = MemberRepository.getInstance();
List<Member> members = memberRepository.findAll();
%>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
  </head>
  <body>
    <a href="/index.html">메인</a>
    <table>
      <thead>
        <th>id</th>
        <th>username</th>
        <th>age</th>
      </thead>
      <tbody>
      <%
        for (Member member : members) {
          out.write(" <tr>");
          out.write(" <td>" + member.getId() + "</td>");
          out.write(" <td>" + member.getUsername() + "</td>");
          out.write(" <td>" + member.getAge() + "</td>");
          out.write(" </tr>");
        }
      %>
      </tbody>
    </table>
  </body>
</html>
```
* `main/webapp/jsp/members.jsp` 회원 목록 출력
#### 2.4. 서블릿과 JSP의 한계
* 서블릿으로 개발 : 뷰 화면을 위한 HTML 작업이 자바 코드와 섞여서 지저분함
* JSP 또한 동적으로 변경이 필요한 부분에는 자바 코드를 적용해야만함
* JSP는 비즈니스 로직과 HTML 뷰 영역이 뒤섞여있다. 역할이 모여있다.
* 이러한 난잡함은 유지보수가 굉장히 힘들어진다.

#### 2.5. MVC 패턴 - 개요
##### 2.5.1. MVC 패턴의 등장
* 비즈니스 로직은 서블릿 처럼 다른 곳에서 처리, JSP는 목적에 맞게 HTML로 화면(view)를 그리는 일에 집중하게하는 MVC 패턴이 등장.

##### 2.5.2. Model View Controller
* 컨트롤러 : HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.
* 모델 : 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링하는 일에 집중할 수 있다.
* 뷰 : 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성하는 부분을 말한다.
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/bde159c6-e918-4da6-9483-276d415214ea)

#### 2.6. MVC 패턴 - 적용
##### 2.6.1. 회원 등록 폼 - 컨트롤러
```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String viewPath = "/WEB-INF/views/new-form.jsp";
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
  }
}
```
* `dispatcher.forward()` : 다른 서블릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서 다시 호출이 발생한다.
> `/WEB-INF`<br/> 이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다. 우리가 기대하는 것은 항상 컨트롤러를 통해서
JSP를 호출하는 것이다. <br/>
> `redirect vs forward`<br/> 리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청한다.
따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다. 반면에 포워드는 서버 내부에서 일어나는 호
출이기 때문에 클라이언트가 전혀 인지하지 못한다.

##### 2.6.2 회원 등록 폼 - 뷰
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
* 예전 코드와다르게 절대 경로가 아닌 상대경로로 설정해둔다.

##### 2.6.3. 회원 저장 - 컨트롤러
```java
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/
save")
public class MvcMemberSaveServlet extends HttpServlet {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));

    Member member = new Member(username, age);
    System.out.println("member = " + member);
    memberRepository.save(member);

    //Model에 데이터를 보관한다.
    request.setAttribute("member", member);
    String viewPath = "/WEB-INF/views/save-result.jsp";
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
  }
}
```
* HttpServletRequest를 Model로 사용한다.
* request가 제공하는 `setAttribute()` 를 사용하면 request 객체에 데이터를 보관해서 뷰에 전달할 수 있다.
* 뷰는 `request.getAttribute()` 를 사용해서 데이터를 꺼내면 된다.

#### 2.6.4. 회원 저장 - 뷰
```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
  <meta charset="UTF-8">
  </head>
    <body>
    성공
    <ul>
      <li>id=${member.id}</li>
      <li>username=${member.username}</li>
      <li>age=${member.age}</li>
    </ul>
    <a href="/index.html">메인</a>
  </body>
</html>
```
* `<%= request.getAttribute("member")%>` 로 모델에 저장한 member 객체를 꺼낼 수 있지만, 너무 복잡해진다. 자바코드가 안 섞이도록 노력한다.
* JSP는 `${}` 문법을 제공하는데, 이 문법을 사용하면 request의 attribute에 담긴 데이터를 편리하게 조회할 수 있다.

##### 2.6.5. 회원 목록 조회 - 컨트롤러
```java
@WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/members")
public class MvcMemberListServlet extends HttpServlet {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    System.out.println("MvcMemberListServlet.service");
    List<Member> members = memberRepository.findAll();
    request.setAttribute("members", members);
    String viewPath = "/WEB-INF/views/members.jsp";
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
  }
}
```
* request 객체를 사용해서 `List<Member> members` 를 모델에 보관했다.

##### 2.6.6. 회원 목록 조회 - 뷰
```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
  </head>
  <body>
    <a href="/index.html">메인</a>
    <table>
    <thead>
      <th>id</th>
      <th>username</th>
      <th>age</th>
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
* `<c:forEach>` 이 기능을 사용하려면 다음과 같이 선언해야 한다. `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
* 아래와 같이 출력해도되지만 자바 코드가 섞이게 되서 지양한다.
```java
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

#### 2.7. MVC 패턴의 한계
* 포워드 중복
  + View로 이동하는 코드가 항상 중복 호출되고 있다.
```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```
* ViewPath 중복
    + `String viewPath = "/WEB-INF/views/new-form.jsp";`
* 사용하지 않는 코드
    + `HttpServletRequest request, HttpServletResponse response` 테스트 케이스 작성도 어렵다.
* 공통 처리가 어렵다
    + 프론트 컨트롤러 패턴을 도입하는 것으로 해결할수있다.

***
### 3. MVC 프레임워크 만들어보기
#### 3.1. 프론트 컨트롤러 도입
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/caea8ab6-5f92-46e9-b7c8-591f73d396ac)
* 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음
* 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
* 입구를 하나로
* 공통처리 가능
* 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨
* 스프링 MVC의 핵심도 FrontController : DispathcerServlet이 FrontController로 구현되어 있음

#### 3.2. 프론트 컨트롤러 도입 v1
##### 3.2.1. 구조
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/d1884c7e-5f74-49e3-a008-556c1a285639)
##### 3.2.2. Controller V1
```java
public class MemberFormControllerV1 implements ControllerV1 {
  @Override
  public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String viewPath = "/WEB-INF/views/new-form.jsp";
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
  }
}
```
* 회원 등록 컨트롤러
```java
public class MemberSaveControllerV1 implements ControllerV1 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @Override
  public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
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
* 회원 저장 컨트롤러
```java
public class MemberListControllerV1 implements ControllerV1 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @Override
  public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    List<Member> members = memberRepository.findAll();
    request.setAttribute("members", members);
    String viewPath = "/WEB-INF/views/members.jsp";
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
  }
}
```
* 회원 목록 컨트롤러
```java
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {
  private Map<String, ControllerV1> controllerMap = new HashMap<>();
  public FrontControllerServletV1() {
    controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
    controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
    controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
  }
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
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
* urlPatterns = "/front-controller/v1/*"` : `/front-controller/v1` 를 포함한 하위 모든 요청은 이 서블릿에서 받아들인다.
* Service()를 통해 `controller.process(request, response);` 을 호출해서 해당 컨트롤러를 실행한다.

##### 3.2.3. V1의 단점
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```
위 코드와 같이 뷰로 이동하는 부분에 중복이 있고 깔끔하지 않다.

#### 3.3. View 분리 - V2
##### 3.3.1 V2 구조
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/d9868b1f-7f79-4637-ac27-aa3ed10dc603)
##### 3.3.2 MyView
```java
public class MyView {
  private String viewPath;
  public MyView(String viewPath) {
    this.viewPath = viewPath;
  }
  public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
  }
}
```
* 중복되는 부분을 따로 함수화

##### 3.3.3 Controller V2
```java
public class MemberFormControllerV2 implements ControllerV2 {
@Override
  public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    return new MyView("/WEB-INF/views/new-form.jsp");
  }
}
```
* 회원 등록 폼 ControllerV2
```java
public class MemberSaveControllerV2 implements ControllerV2 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @Override
  public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    Member member = new Member(username, age);
    memberRepository.save(member);
    request.setAttribute("member", member);
    return new MyView("/WEB-INF/views/save-result.jsp");
  }
}
```
* 회원 저장 폼 Controller V2
```java
public class MemberListControllerV2 implements ControllerV2 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @Override
  public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    List<Member> members = memberRepository.findAll();
    request.setAttribute("members", members);
    return new MyView("/WEB-INF/views/members.jsp");
  }
}
```
* 회원 목록 Controllter V2
```java
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {
  private Map<String, ControllerV2> controllerMap = new HashMap<>();
  public FrontControllerServletV2() {
    controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
    controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
    controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
  }
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
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
* Front Controller V2
* ControllerV2의 반환 타입이 `MyView` 이므로 프론트 컨트롤러는 컨트롤러의 호출 결과로 `MyView` 를 반환 받는다. 그리고 `view.render()` 를 호출하면 `forward` 로직을 수행해서 JSP가 실행된다.
* 프론트 컨트롤러의 도입으로 `MyView` 객체의 `render()` 를 호출하는 부분을 모두 일관되게 처리할 수 있다. 각각의 컨트롤러는 `MyView` 객체를 생성만 해서 반환하면 된다.

##### 3.3.4. Controller V2 단점
* 서블릿 종속성 제거 : HttpServletRequest, HttpServletResponse의 불필요함
* 뷰 이름 중복 제거

#### 3.4. Model 추가 - V3
##### 3.4.1. V3 구조
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/cd806da9-51d2-4613-9dec-f4e41aaedb73)
##### 3.4.2. ModelView
```java
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
* 뷰의 이름과 뷰를 렌더링할 때 필요한 Model 객체

##### 3.4.3. Controller V3
```java
public class MemberFormControllerV3 implements ControllerV3 {
@Override
  public ModelView process(Map<String, String> paramMap) {
    return new ModelView("new-form");
  }
}
```
* ModelView를 생성할 때 new-form이라는 view의 논리적 이름을 지정. 실제 경로는 프론트 컨트롤러에서 처리한다.

```java
public class MemberSaveControllerV3 implements ControllerV3 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
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
* 회원 등록 폼 파라미터 정보와 모델에 뷰에 필요한 member 객체를 담고 반환한다.

```java
public class MemberSaveControllerV3 implements ControllerV3 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
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
* 회원 저장 ControllerV3

```java
public class MemberListControllerV3 implements ControllerV3 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @Override
  public ModelView process(Map<String, String> paramMap) {
    List<Member> members = memberRepository.findAll();
    ModelView mv = new ModelView("members");
    mv.getModel().put("members", members);
    return mv;
  }
}
```
* 회원 목록 ControllerV3

```java
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {
  private Map<String, ControllerV3> controllerMap = new HashMap<>();
  public FrontControllerServletV3() {
    controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
    controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
    controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
  }
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
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
* FrontControllerV3
* `MyView view = viewResolver(viewName)`
* 컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경한다. 그리고 실제 물리 경로가 있는 MyView 객체를 반환한다.
  + 논리 뷰 이름: `members`
  + 물리 뷰 경로: `/WEB-INF/views/members.jsp
```java
public class MyView {
  private String viewPath;
  public MyView(String viewPath) {
    this.viewPath = viewPath;
  }
  public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
  }
  public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    modelToRequestAttribute(model, request);
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
  }
  private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
    model.forEach((key, value) -> request.setAttribute(key, value));
  }
}
```
* MyView
* `view.render(mv.getModel(), request, response)`
  + 뷰 객체를 통해서 HTML 화면을 렌더링 한다.
  + 뷰 객체의 `render()` 는 모델 정보도 함께 받는다.
  + JSP는 `request.getAttribute()` 로 데이터를 조회하기 때문에, 모델의 데이터를 꺼내서 `request.setAttribute()` 로 담아둔다.
  + JSP로 포워드 해서 JSP를 렌더링 한다.

#### 3.5. 단순하고 실용적인 컨트롤러 - V4
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/5b5469e8-ae5c-4146-b977-f529b293d509)
* V3와 같지만 ModelView를 반환하지 않고, ViewName만 반환한다.

```java
public class MemberFormControllerV4 implements ControllerV4 {
  @Override
  public String process(Map<String, String> paramMap, Map<String, Object>model) {
    return "new-form";
  }
}
```
* 멤버 폼 Controller V4
* V3에서 ModelView를 반환하지 않고 String을 반환한다.

```java
public class MemberSaveControllerV4 implements ControllerV4 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
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
* 멤버 저장 Controller V4
* 위와 마찬가지로 String이 반환된다
* 모델이 파라미터로 전달되기 때문에 모델을 직접 생성하지 않아도된다.

```java
public class MemberListControllerV4 implements ControllerV4 {
private MemberRepository memberRepository = MemberRepository.getInstance();
@Override
  public String process(Map<String, String> paramMap, Map<String, Object> model) {
    List<Member> members = memberRepository.findAll();
    model.put("members", members);
    return "members";
  }
}
```
* 멤버 목록 Controller V4

```java
@WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/v4/*")
public class FrontControllerServletV4 extends HttpServlet {
  private Map<String, ControllerV4> controllerMap = new HashMap<>();
  public FrontControllerServletV4() {
    controllerMap.put("/front-controller/v4/members/new-form", new MemberFormControllerV4());
    controllerMap.put("/front-controller/v4/members/save", new MemberSaveControllerV4());
    controllerMap.put("/front-controller/v4/members", new MemberListControllerV4());
  }
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String requestURI = request.getRequestURI();
    ControllerV4 controller = controllerMap.get(requestURI);
    if (controller == null) {
      response.setStatus(HttpServletResponse.SC_NOT_FOUND);
      return;
    }
    Map<String, String> paramMap = createParamMap(request);
    Map<String, Object> model = new HashMap<>(); //추가
    String viewName = controller.process(paramMap, model);
    MyView view = viewResolver(viewName);
    view.render(model, request, response);
  }
  private Map<String, String> createParamMap(HttpServletRequest request) {
    Map<String, String> paramMap = new HashMap<>();
    request.getParameterNames().asIterator()
    .forEachRemaining(paramName -> paramMap.put(paramName,
    request.getParameter(paramName)));
    return paramMap;
  }
  private MyView viewResolver(String viewName) {
    return new MyView("/WEB-INF/views/" + viewName + ".jsp");
  }
}
```
* 모델 객체 전달, 모델객체를 컨트롤러에서 생성해서 넘겨준다.
* 뷰의 논리 이름을 직접 반환

#### 3.6. 유연한 컨트롤러1 - V5
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/24da69d9-42a9-4754-8272-0c8c82d50a43)
* 누구는 V3의 방식으로 누구는 V4의 방식으로 개발하고 싶다면 어떻게해야할까
* 핸들러 어뎁터, 핸들러를 추가하여서 HTTP 요청을 처리한다.
* 핸들러 어댑터 : 어댑터 역할로 인해 다양한 종류의 컨트롤러를 호출한다.

```java
public interface MyHandlerAdapter {
  boolean supports(Object handler);
  ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```
* `boolean supports(Object handler)`
  + handler는 컨트롤러를 말한다.
  + 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드다.
* `ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)`
  + 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다.
  + 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 ModelView를 직접 생성해서라도 반환해야 한다.
  + 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만 이제는 이 어댑터를 통해서 실제 컨트롤러가 호출된다.
 
```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
  @Override
  public boolean supports(Object handler) {
    return (handler instanceof ControllerV3);
  }
  @Override
  public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) {
    ControllerV3 controller = (ControllerV3) handler;
    Map<String, String> paramMap = createParamMap(request);
    ModelView mv = controller.process(paramMap);
    return mv;
  }
  private Map<String, String> createParamMap(HttpServletRequest request) {
    Map<String, String> paramMap = new HashMap<>();
    request.getParameterNames().asIterator()
    .forEachRemaining(paramName -> paramMap.put(paramName,
    request.getParameter(paramName)));
    return paramMap;
  }
}
```
* supports() : ControllerV3을 처리할 수 있는 어댑터를 뜻한다.
* V3 핸들러 어댑터

```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {
  private final Map<String, Object> handlerMappingMap = new HashMap<>();
  private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();
  public FrontControllerServletV5() {
    initHandlerMappingMap();
    initHandlerAdapters();
  }
  private void initHandlerMappingMap() {
    handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
    handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
    handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
  }
  private void initHandlerAdapters() {
    handlerAdapters.add(new ControllerV3HandlerAdapter());
  }
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
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
    throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다.handler=" + handler);
  }
  private MyView viewResolver(String viewName) {
    return new MyView("/WEB-INF/views/" + viewName + ".jsp");
  }
}
```
* FrontControllerServletV5 : 컨트롤러 핸들러 매핑

#### 3.7. 유연한 컨트롤러2 - V5
```java
private void initHandlerMappingMap() {
  handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
  handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
  handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
  //V4 추가
  handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
  handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
  handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
}
private void initHandlerAdapters() {
  handlerAdapters.add(new ControllerV3HandlerAdapter());
  handlerAdapters.add(new ControllerV4HandlerAdapter()); //V4 추가
}
```
* ControllerV4 추가

```java
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
    .forEachRemaining(paramName -> paramMap.put(paramName,
    request.getParameter(paramName)));
    return paramMap;
  }
}
```
* ControllerV4 는 뷰이름을 반환했지만, 아래와 같이 어댑터는 ModelView로 변경하여 반환한다.
``` java
public interface ControllerV4 {
  String process(Map<String, String> paramMap, Map<String, Object> model);
}
public interface MyHandlerAdapter {
  ModelView handle(HttpServletRequest request, HttpServletResponse response,
  Object handler) throws ServletException, IOException;
}
```

### 4. 스프링 MVC
#### 4.1. 스프링 MVC 구조 이해
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/8f6d23f3-1b27-4189-82f9-91cec280713f)
**직접 만든 프레임워크 스프링 MVC 비교**
* FrontController DispatcherServlet
* handlerMappingMap HandlerMapping
* MyHandlerAdapter HandlerAdapter
* ModelView ModelAndView
* viewResolver ViewResolver
* MyView View

##### 4.1.1. DispatcherServlet 구조 살펴보기
`org.springframework.web.servlet.DispatcherServlet`
* 스프링MVC도 Front Controller 패턴으로 구현되어 있음.
* Front Controller가 DispatcherServlet이다.
* DispatcherServlet도 부모 클래스에서 `HttpServlet`을 상속 받아서 사용하고, 서블릿으로 동작한다.
    + DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
* 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 urlPatterns에 대해서 매핑한다.

##### 4.1.2. 요청 흐름
* 서블릿이 호출되면 `HttpServlet`이 제공하는 Service() 호출
* 스프링 MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 Service()를 오버라이드 해둠
* FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출됨

##### 4.1.3. SpringMVC 동작 순서
1. 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행 : 핸들러 어댑터를 실행한다.
4. 핸들러 실행 : 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환 : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. ViewReSolver 호출 : 뷰 리졸버를 찾고 실행한다.
7. View 반환 : 뷰 리졸버는 뷰의 논리 이름을 물리이름으로 바꾸고, 랜더링 역할을 담당하는 뷰 객체를 반환한다.
8. 뷰 렌더링 : 뷰를 통해서 뷰를 랜더링한다.

> 스프링 MVC의 강점은 DispatcherServlet 코드의 변경 없이, 원하는 기능을 변경하거나 확장할 수 있다. <br/>
> 이러한 인터페이스를 구현하여 `DispatcherServlet`에 등록하면 나만의 컨트롤러를 만들 수 있다. <br/><br/><br/>
> **주요 인터페이스 목록** <br/>
> * 핸들러 매핑: `org.springframework.web.servlet.HandlerMapping`
> * 핸들러 어댑터: `org.springframework.web.servlet.HandlerAdapter`
> * 뷰 리졸버: `org.springframework.web.servlet.ViewResolver`
> * 뷰: `org.springframework.web.servlet.View`

#### 4.2. 핸들러 매핑과 핸들러 어댑터
```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {
  @Override
  public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
    System.out.println("OldController.handleRequest");
    return null;
  }
}
```
* `@Component` : 이 컨트롤러는 `/springmvc/old-controller` 라는 이름의 스프링 빈으로 등록되었다.
* **빈의 이름으로 URL을 매핑**할 것이다.

##### 4.2.1. 컨트롤러의 호출 방법
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/0faec9a8-8661-4bc7-ad27-5c7b2542b0c0)
* HandlerMapping(핸들러매핑)
    + 핸들러 매핑에서 컨트롤러를 찾아야한다.
    + 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
    + 0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
    + 1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
* HandlerAdapter(핸들러 어댑터)
    + 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
    + 0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
    + 1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
    + 2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리
 
##### 4.2.2. Controller 인터페이스 호출 순서

**1. 핸들러 매핑으로 핸들러 조회**
  + 1. `HandlerMapping` 을 순서대로 실행해서, 핸들러를 찾는다.
  + 2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 핸들러를 찾아주는 `BeanNameUrlHandlerMapping` 가 실행에 성공하고 핸들러인 `OldController` 를 반환한다.
**2. 핸들러 어댑터 조회**
  + 1. `HandlerAdapter` 의 `supports()` 를 순서대로 호출한다.
  + 2. `SimpleControllerHandlerAdapter` 가 `Controller` 인터페이스를 지원하므로 대상이 된다.
**3. 핸들러 어댑터 실행**
  + 1. 디스패처 서블릿이 조회한 `SimpleControllerHandlerAdapter` 를 실행하면서 핸들러 정보도 함께 넘겨준다.
  + 2. `SimpleControllerHandlerAdapter` 는 핸들러인 `OldController` 를 내부에서 실행하고, 그 결과를 반환한다.

#### 4.3. View Resolver(뷰리졸버)
```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {
  @Override
  public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
    System.out.println("OldController.handleRequest");
    return new ModelAndView("new-form");
  }
}
```
`application.properties` 
```
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

##### 4.3.1. View Resolver 동작 방식
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/36922bc4-0bb7-44ea-b21d-67235e88f8e2)
* 스프링 부트가 자동 등록하는 뷰 리졸버
  + 1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
  + 2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.

1. 핸들러 호출
    + 핸들러 어댑터를 통해 논리뷰 이름을 획득
2. ViewResolver 호출
    + 논리 뷰 이름으로 viewResolver를 순서대로 호출
    + BeanNameViewResolver는 이름의 스프링빈으로 등록된 뷰를 찾아야한다
    + 없으면 InternalResourceViewResolver가 호출된다.
3. InternalResourceViewResolver
    + 이 뷰 리졸버는 InternalResourceView를 반환한다.
4. 뷰 - InternalResourceView
    + `InternalResourceView` 는 JSP처럼 포워드 `forward()` 를 호출해서 처리할 수 있는 경우에 사용한다.
5. view.render()
    + `view.render()` 가 호출되고 `InternalResourceView` 는 `forward()` 를 사용해서 JSP를 실행한다.
  
### 4.4. 스프링 MVC - 시작하기


