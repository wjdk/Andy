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
resources.addPreResources(new DirResourceSet(resources, "/WEB-INF/classes", new File("target/classes").getAbsolutePath(), "/"));
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

<img width="471" height="243" alt="Image" src="https://github.com/user-attachments/assets/ac3fc8f8-e5ad-477d-91e7-0bbbcc77d605" />

Controller(UserServlet)根据请求内容从数据库中获取Model(user：一个Javabean或map)并把Model转发给View(user.jsp)渲染，View返回响应内容给Browser。

### MVC框架

我们希望业务逻辑(Controller)和Servlet接口分离，由Java类实现；
JSP更换为其他更好用的模版引擎。

新框架如下：

<img width="417" height="305" alt="Image" src="https://github.com/user-attachments/assets/dabf4e7f-d56f-4be1-a616-8b5f5d8d403f" />

#### 项目的具体实现：

在这个项目中Servelet只有`DispatcherServlet`一个(`@WebServlet(urlPatterns = "/")`)，也就是说所有请求都会匹配到这里。
> 映射到/的IndexServlet比较特殊，它实际上会接收所有未匹配的路径，相当于/*

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

### Filter
在请求到达Servlet之前，先通过Filter预处理或过滤。
```Java
@WebFilter("/*")//需要该Filter过滤的url，此处表示所有url都需要过滤
public class LogFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("LogFilter: process " + ((HttpServletRequest) request).getRequestURI());
        chain.doFilter(request, response);//请求继续处理（通过过滤）
    }
}
```
### Listener
监听webApp的事件作出反应，如：
```Java
@WebListener//标注了@WebListener，且实现了特定接口的类会被Web服务器自动初始化
public class AppListener implements ServletContextListener {
    //监听`ServletContext`实例的创建和销毁
    public void contextInitialized(ServletContextEvent sce) {//在整个Web应用程序初始化完成后被调用
        System.out.println("WebApp initialized.");
    }
    public void contextDestroyed(ServletContextEvent sce) {//在Web应用程序关闭后被调用
        System.out.println("WebApp destroyed.");
    }
}
```
一个Web服务器可以运行一个或多个WebApp，对于每个WebApp，Web服务器都会为其创建一个全局唯一的`ServletContext`实例。

## Spring开发

[Spring注解1_Spring&SpringBoot常用注解总结](https://javaguide.cn/system-design/framework/spring/spring-common-annotations.html#spring-boot-%E5%9F%BA%E7%A1%80%E6%B3%A8%E8%A7%A3)
[Spring注解2_Spring 的 @Bean 和 @Component 有什么区别？](https://cloud.tencent.com/developer/article/1984063)

### IoC

传统的程序开发模式实例化一个组件需要先实例化该组件所有的子组件，而子组件的实例化也需要先实例化其子组件，再加上部分组件实例化后应当被很多父组件共享，这会导致及其复杂的依赖关系，且组件之间紧密耦合不易维护。

IoC(Inversion of Control)，所有组件不再由应用程序自己创建和配置，而是由IoC容器负责。

#### DI：依赖注入

```Java
public class BookService {
    private DataSource dataSource;
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }//属性注入可以改为构造方法注入
}
```
> BookService自己并不会创建DataSource，而是等待外部通过setDataSource()方法来注入一个DataSource。

实例化子组件->外部注入组件：BookService不必关心如何具体实例化dataSource（配置dataSource）属性，而是将其交给IoC容器。

#### IoC容器

IoC容器负责实例化所有的组件并管理组件的生命周期。

可以通过xml文件告诉容器各组件之间的依赖关系。
可以使用注解来告诉容器如何组装组件（类似xml）。
容器使用：
```Java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");//IoC容器(从xml文件读取配置)
// ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class); //从annotation中读取配置
UserService userService = context.getBean(UserService.class);//从IoC容器中获取相应类型的bean,IoC容器会自动装配组件
```
注解使用：
```Java
@Component //标注为需要IoC容器装配的Javabean
public class UserService {
    @Autowired//表示该字段需要IoC容器注入
    MailService mailService;
    ...
}
...
@Configuration//配置类，表示在该类中可以生成多个@Bean方法交由IoC容器处理(同时也相当于声明了@Component)
@ComponentScan//自动扫描带@Component的类并组装为Bean
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        UserService userService = context.getBean(UserService.class);
        User user = userService.login("bob@example.com", "password");
        System.out.println(user.getName());
    }
}

```
> tip:@@Component和@Bean的区别
@Component注解作用于类，而@Bean注解作用于方法；两者都将之后的代码标记为需要IoC容器创建的bean对象。

可以通过@PropertySource注解加载自定义的配置文件，再通过@Value注入，如
```Java
@Configuration
@ComponentScan
@PropertySource("app.properties") // 表示读取classpath的app.properties
public class AppConfig {
    ...
}
@Component
public class SmtpConfig { 
    @Value("${smtp.host:localhost}")//以${key:defaultValue}的形式注入
    private String host;
    ...
}
...
@Component
public class MailService {
    //#{}表示从JavaBean读取属性
    //Class名为SmtpConfig的Bean，它在Spring容器中的默认名称就是smtpConfig
    @Value("#{smtpConfig.host}")//从名称为smtpConfig的Bean读取host属性
    private String smtpHost;
}

```

### AOP

Aspect Oriented Programming
...

### 数据库操作

JDBC见附录

#### JdbcTemplate

创建一个`JdbcTemplate`实例需要注入`DataSource`

`T execute(ConnectionCallback<T> action)`方法提供了Jdbc的Connection

`T queryForObject(String sql, RowMapper<T> rowMapper, Object... args)`
rowMapper是一个lambda函数，负责将ResultSet的当前行`(ResultSet rs, int rowNum)`映射为一个JavaBean；
args参数是sql语句的参数（填充问号）。
queryForObject返回一行记录对应的对象，query返回多行。
如果在设计表结构的时候，能够和JavaBean的属性一一对应，RowMapper可以直接使用BeanPropertyRowMapper，如`new BeanPropertyRowMapper<>(User.class)`。

```Java
public User getUserByEmail(String email) {
    // 传入SQL，参数和RowMapper实例:
    return jdbcTemplate.queryForObject("SELECT * FROM users WHERE email = ?",
            (ResultSet rs, int rowNum) -> {
                // 将ResultSet的当前行映射为一个JavaBean:
                return new User( // new User object:
                        rs.getLong("id"), // id
                        rs.getString("email"), // email
                        rs.getString("password"), // password
                        rs.getString("name")); // name
            },
            email);
}

```

#### 声明式事务
事务具有ACID特性。
`@Transactional`标注一个事务。
默认的事务传播级别是`REQUIRED`：如果当前没有事务，就创建一个新事务，如果当前有事务，就加入到当前事务中执行。（事务只能在当前线程传播，无法跨线程传播）

#### Mybatis


### Spring MVC

Spring MVC的实现类似于web开发中的MVC框架。

启动流程：

Tomcat（servlet容器）启动并添加webApp路径读取web.xml配置；
根据配置实例化DispatcherServlet并绑定Spring容器（需读取Spring容器类型和对应@Configuration类名）；
Spring容器会根据注解扫描组装所有@Component标记的bean(使用@ComponentScan)和@Bean标记的bean（@Configuration下）；
DispatcherServlet接受所有web.xml中配置的路径（如/*）中的请求，并在Spring容器的帮助下根据注解分配到@Controller标记的类中。

修改密码功能实现：

在业务层userController添加修改密码页面的Get/Post方法（对应路径/updpassword）；
在webapp添加/updpassword页面的view模版html文件；
在数据访问层userService添加修改密码对应的数据库操作（合法性判定&和update语句）。
请求处理流程: DistpatcherServlet->userController->userService从数据库获取Model或者修改数据库->UserController将ModelAndView返回->....(移交DispatcherServlet自动处理)

### Rest

Rest服务：输入输出都是json便于第三方调用。
实现：使用`@RestController`替代`@Controller`

[Jackson的序列化和反序列化](https://juejin.cn/post/7196852501190557752)

### 集成Filter

和Servlet开发的内容一样，集成Filter只需要在xml文件中配置，Servlet容器就会自动创建对应的Filter实例；
但这样创建出来的Filter在Servlet容器中，无法使用Spring容器中的功能（如userService的signin功能）;
可以使用代理模式创建名为`DelegatingFilterProxy`的Filter，这个Filter会自动去Spring容器中查找对应名字的bean；
这样我们就可以把Filter想实现的功能写在Spring容器的bean里面，在Servlet容器创建代理Filter调用Spring容器里bean的方法。
xml配置代理Filter的内容如下：
```xml
<web-app>
    <filter>
        <filter-name>authFilter</filter-name> <!--会在Spring容器中查找同名的bean-->
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>authFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>

```

## 附录：JDBC

### 简介
![alt text](image.png)
JDBC是Java程序访问数据库的标准接口，接口的实现由具体的数据库驱动实现。
### 使用

CRUD使用PreparedStatement而不是直接通过参数拼字符串来避免SQL注入。
- tip: [try_with_resoures使用](https://juejin.cn/post/6844903446185951240)
可以使用try(connection)建立连接，这样try语句块结束时connection会被自动正确关闭。


```Java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("INSERT INTO students (id, grade, name, gender) VALUES (?,?,?,?)")) {
        ps.setObject(1, 999); // 注意：索引从1开始
        ps.setObject(2, 1); // grade
        ps.setObject(3, "Bob"); // name
        ps.setObject(4, "M"); // gender
        int n = ps.executeUpdate(); // 1
    }
}
```


`javax.sql.DataSource`: 通过数据库连接池复用连接,类似线程池。

```Java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/test");
config.setUsername("root");
config.setPassword("password");
config.addDataSourceProperty("connectionTimeout", "1000"); // 连接超时：1秒
config.addDataSourceProperty("idleTimeout", "60000"); // 空闲超时：60秒
config.addDataSourceProperty("maximumPoolSize", "10"); // 最大连接数：10
DataSource ds = new HikariDataSource(config);
...
try (Connection conn = ds.getConnection()) { // 从连接池中获取连接
}
```
## 参考资料

[廖雪峰的Java教程](https://liaoxuefeng.com/books/java/introduction/index.html)