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

### 重定向和转发

重定向要求浏览器发送新的请求到新的路径，转发是通过转发请求到服务器内部`servlet`来处理请求。

重定向
```Java
@WebServlet(urlPatterns = "/hi")
public class RedirectServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 构造重定向的路径:
        String name = req.getParameter("name");
        String redirectToUrl = "/hello" + (name == null ? "" : "?name=" + name);
        // 发送重定向响应:
        resp.sendRedirect(redirectToUrl);
    }
}

```
转发
```Java
@WebServlet(urlPatterns = "/morning")
public class ForwardServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.getRequestDispatcher("/hello").forward(req, resp);
    }
}

```
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

### JSP

JSP是Java Server Pages的缩写，它的文件必须放到/src/main/webapp下，文件名必须以.jsp结尾，整个文件与HTML并无太大区别，但需要插入变量，或者动态输出的地方，使用特殊指令<% ... %>。
JSP在执行前首先被Servlet容器编译成一个Servlet。

JSP页面内置了几个变量：
out：表示HttpServletResponse的PrintWriter；
session：表示当前HttpSession对象；
request：表示HttpServletRequest对象。

### 简易MVC例子

![](image.png)

Controller(UserServlet)根据请求内容从数据库中获取Model(user：一个Javabean或map)并把Model转发给View(user.jsp)渲染，View返回响应内容给Browser。

### MVC框架

我们希望业务逻辑(Controller)和Servlet接口分离，由Java类实现；
JSP更换为其他更好用的模版引擎。

新框架如下：

![alt text](image-1.png)

#### 项目的具体实现：

Controller是Java业务类对象（单例类），一个Controller可能封装多个方法处理不同路径的请求。
DispatcherServlet的map对象`getMappings`和`postMappings`保存了路径到GetDispatcher/PostDispatcher对象的映射。
GetDispatcher/PostDispatcher对象保存了对应Controller的单个方法（包括方法引用、类、参数名列表和参数类型列表），用于处理单个路径的请求。
`GetDispatcher/PostDispatcher`继承自同一抽象类`AbstractDispatcher`

请求响应过程：
1. Browser发送请求到`DispatcherServlet`;
2. `DispatcherServlet`根据请求类型(get/post)和路径从map里找到对应的`AbstractDispatcher`实例；
3. `AbstractDispatcher`实例根据请求和参数要求解析出具体参数，利用反射语法调用`Controller`的对应方法获得`ModelAndView`；
4. `DispatcherServlet`再将得到的`ModelAndView`交给`ViewEngine`渲染返回响应。

资源加载过程：
> Servlet容器创建当前Servlet实例后，会自动调用init(ServletConfig)方法
1. Servlet容器调用`DispatcherServlet`的`init()`方法;
2. `init()`加载所有预加载的业务类(controller)方法，通过注解得到这些方法处理的请求路径，创建对应的`(path,AbstractDispatcher)`键值对加到`DispatcherServlet`的map对象里。
3. 创建`viewEngine`对象（模版引擎）

渲染过程：
ModelAndView分别表示需要渲染的数据对象（如Javabean）和页面(如html)；
模版引擎（如pebbleEngine）会根据Model和View自动渲染页面。
tip:可通过`{% extends "_base.html" %}`实现模版的继承，每一个页面都首先渲染`_base.html`再渲染自己独有的内容。

总结：

实现了业务逻辑和框架分离。
## 参考资料

[廖雪峰的Java教程](https://liaoxuefeng.com/books/java/introduction/index.html)