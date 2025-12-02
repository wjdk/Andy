# Java学习笔记

## web开发

### Servlet

每个`Servlet`类通过注解说明自己能处理的路径;
`HttpServlet`是`Servlet`的子类，通过实现`HttpServlet`子类的`doGet()`,`doPost()`方法处理get/post请求。
`Servlet`是单例类。
在`Servlet`中定义的实例变量会被多个线程同时访问，要注意线程安全；
`HttpServletRequest`和`HttpServletResponse`实例是由`Servlet`容器传入的局部变量，它们只能被当前线程访问；
在`doGet()`或`doPost()`方法中，如果使用了ThreadLocal，但没有清理，那么它的状态很可能会影响到下次的某个请求，因为`Servlet`容器很可能用线程池实现线程复用。

<details>
<summary>登录页面代码示例</summary>

```Java
@WebServlet(urlPatterns = "/signin")
public class SignInServlet extends HttpServlet {
    // 模拟一个数据库:
    private Map<String, String> users = Map.of("bob", "bob123", "alice", "alice123", "tom", "tomcat");

    // GET请求时显示登录页:
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter pw = resp.getWriter();
        pw.write("<h1>Sign In</h1>");
        pw.write("<form action=\"/signin\" method=\"post\">");
        pw.write("<p>Username: <input name=\"username\"></p>");
        pw.write("<p>Password: <input name=\"password\" type=\"password\"></p>");
        pw.write("<p><button type=\"submit\">Sign In</button> <a href=\"/\">Cancel</a></p>");
        pw.write("</form>");
        pw.flush();
    }

    // POST请求时处理用户登录:
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String name = req.getParameter("username");
        String password = req.getParameter("password");
        String expectedPassword = users.get(name.toLowerCase());
        if (expectedPassword != null && expectedPassword.equals(password)) {
            // 登录成功:
            req.getSession().setAttribute("user", name);
            resp.sendRedirect("/");
        } else {
            resp.sendError(HttpServletResponse.SC_FORBIDDEN);
        }
    }
}

```
</details>

### Servlet容器

> 无法在代码中直接通过new创建Servlet实例，必须由Servlet容器（如Tomcat）自动创建Servlet实例；
Servlet容器只会给每个Servlet类创建唯一实例；
Servlet容器会使用多线程执行doGet()或doPost()方法。
```Java
resources.addPreResources(
        new DirResourceSet(resources, "/WEB-INF/classes", new File("target/classes").getAbsolutePath(), "/"));
```
这段代码表示Tomcat会去target/classes目录下去寻找需要加载的`Servlet`类文件

### Cookie和Session

#### Cookie

通过`HttpServletResponse`设置`cookie`以后，浏览器会在有效期内访问生效路径时带上`cookie`。
浏览器请求的RequestHeaders的Cookie部分如下：
<img width="620" height="124" alt="Image" src="https://github.com/user-attachments/assets/90beceb8-aec4-4a40-ac19-2ec77b4c28e1" />


<details>
<summary>cookie设置</summary>

```Java
@WebServlet(urlPatterns = "/pref")
public class LanguageServlet extends HttpServlet {

    private static final Set<String> LANGUAGES = Set.of("en", "zh");

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String lang = req.getParameter("lang");
        if (LANGUAGES.contains(lang)) {
            // 创建一个新的Cookie:
            Cookie cookie = new Cookie("lang", lang);
            // 该Cookie生效的路径范围:
            cookie.setPath("/");
            // 该Cookie有效期:
            cookie.setMaxAge(8640000); // 8640000秒=100天
            // 将该Cookie添加到响应:
            resp.addCookie(cookie);
        }
        resp.sendRedirect("/");
    }
}
```
</details>

可以通过`req.getCookies()`获取到请求的Cookie信息。

#### Session
http是无状态协议，但是可以通过建立`session`来保存连接的状态。
可以通过```session=req.getSession()```获取当前会话，调用`session.setAttribute()`和`session.getAttribute()`修改和获取当前会话的属性。
通过Cookie来实现Session机制：
Servelet容器在第一次调用`getSession()`时会创建`Session`；
Servlet容器会自动创建`JSESSIONID`送给浏览器，浏览器请求时会带上`JSESSIONID`;

<details>
<summary>req.getSession()源码（具体实现在request.class)</summary>

```Java
public class Request implements HttpServletRequest {
    ...
    public HttpSession getSession(boolean create) {
        Session session = this.doGetSession(create);
        return session == null ? null : session.getSession();
    }
    ...
    protected Session doGetSession(boolean create) {
        Context context = this.getContext();
        if (context == null) {
            return null;
        } else {
            if (this.session != null && !this.session.isValid()) {
                this.session = null;
            }

            if (this.session != null) {
                return this.session;
            } else {
                Manager manager = context.getManager();
                if (manager == null) {
                    return null;
                } else {
                    if (this.requestedSessionId != null) {
                        try {
                            this.session = manager.findSession(this.requestedSessionId);
                        } catch (IOException e) {
                            if (log.isDebugEnabled()) {
                                log.debug(sm.getString("request.session.failed", new Object[]{this.requestedSessionId, e.getMessage()}), e);
                            } else {
                                log.info(sm.getString("request.session.failed", new Object[]{this.requestedSessionId, e.getMessage()}));
                            }

                            this.session = null;
                        }

                        if (this.session != null && !this.session.isValid()) {
                            this.session = null;
                        }

                        if (this.session != null) {
                            this.session.access();
                            return this.session;
                        }
                    }

                    if (!create) {
                        return null;
                    } else {
                        boolean trackModesIncludesCookie = context.getServletContext().getEffectiveSessionTrackingModes().contains(SessionTrackingMode.COOKIE);
                        if (trackModesIncludesCookie && this.response.getResponse().isCommitted()) {
                            throw new IllegalStateException(sm.getString("coyoteRequest.sessionCreateCommitted"));
                        } else {
                            String sessionId = this.getRequestedSessionId();
                            if (!this.requestedSessionSSL) {
                                if ("/".equals(context.getSessionCookiePath()) && this.isRequestedSessionIdFromCookie()) {
                                    if (context.getValidateClientProvidedNewSessionId()) {
                                        boolean found = false;

                                        for(Container container : this.getHost().findChildren()) {
                                            Manager m = ((Context)container).getManager();
                                            if (m != null) {
                                                try {
                                                    if (m.findSession(sessionId) != null) {
                                                        found = true;
                                                        break;
                                                    }
                                                } catch (IOException var13) {
                                                }
                                            }
                                        }

                                        if (!found) {
                                            sessionId = null;
                                        }
                                    }
                                } else {
                                    sessionId = null;
                                }
                            }

                            this.session = manager.createSession(sessionId);
                            if (this.session != null && trackModesIncludesCookie) {
                                Cookie cookie = ApplicationSessionCookieConfig.createSessionCookie(context, this.session.getIdInternal(), this.isSecure());
                                this.response.addSessionCookieInternal(cookie);
                            }

                            if (this.session == null) {
                                return null;
                            } else {
                                this.session.access();
                                return this.session;
                            }
                        }
                    }
                }
            }
        }
    }
    ...
}
```
</details>

通过源码可以发现：
`request`类的`doGetSession()`会首先试图访问缓存的`this.session`；
如果找不到则会通过`manager`寻找请求的`requestedSessionId`寻找对应的`Session`；
如果依旧找不到并且`create`参数为真，会根据请求的`sessionId`创建对应`Session`（请求的`sessionId`可能为空）；
创建完成`Session`以后会在返回时设置浏览器cookie的`SessionId`（对应代码如下）。
```Java
Cookie cookie = ApplicationSessionCookieConfig.createSessionCookie(context, this.session.getIdInternal(), this.isSecure());
this.response.addSessionCookieInternal(cookie);
```

## 参考资料

[廖雪峰的Java教程](https://liaoxuefeng.com/books/java/introduction/index.html)