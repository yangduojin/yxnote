# JavaWeb

- [JavaWeb](#javaweb)
  - [servlet](#servlet)
    - [重定向](#重定向)
    - [请求转发](#请求转发)
    - [HttpServletRequest & HttpServletResponse](#httpservletrequest--httpservletresponse)
  - [Filter](#filter)
  - [Listener](#listener)
  - [加密](#加密)
  - [Restful API接口规范包括以下部分](#restful-api接口规范包括以下部分)

## servlet

### 重定向

- Redirect 地址栏发生变化，两次请求，不共享request域中数据，不能访问web_inf下面的资源，可以访问工程外的资源.(resp.sendRedirect(XXX))
- resp.setStatus(302);// 设置响应状态码 302 ，表示重定向，（已搬迁）
- resp.setHeader("Location", "http://localhost:8080");// 设置响应头，说明 新的地址在哪里，或 resp.sendRedirect("http://localhost:8080");
- //重定向 model不能携带数据,需要RedirectAttributes他来带数据到新页面,在参数中引入他
- redirectAttributes.addFlashAttribute() 将数据放在session中,只能取一次
- redirectAttributes.addAttribute() 将数据放在url的后面

### 请求转发

- Forward 服务器内部重定向(请求转发) 地址栏不变，一次请求，共享Request/Response域中的数据，不可以访问工程以外的资源
- 可访问web_inf里的文件，建议用request域对象来传输数据，只有访问web_inf文件或请求域对象这两种情况用请求转发，否则用Redirect
- request.getRequestDispatcher("路径").forward(request,response);

### HttpServletRequest & HttpServletResponse

1. HttpServletRequest 类
   - 请求进入tomcat，会被解析http信息，封装到Request对象中，
   - 传入service方法(doget,dopost)中，通过HttpServletRequest对象获取所有信息
     - getRequestURI()  获取请求的资源路径
     - getRequestURL()  获取请求的统一资源定位符（绝对路径）
     - getRemoteHost()  获取客户端的 ip 地址
     - getHeader() 获取请求头
     - getParameter()  获取请求的参数
     - getParameterValues()  获取请求的参数（多个值的时候使用）
     - getMethod()  获取请求的方式 GET 或 POST
     - setAttribute(key, value);  设置域数据
     - getAttribute(key);  获取域数据
     - getRequestDispatcher()  获取请求转发
   - req.setCharacterEncoding("UTF-8"); 设置中文防乱码 放在第一句
   - resp.setContentType("text/html; charset=UTF-8");// 中文乱码 放在第一句
   - Tomcat8及以上版本已经在配置中解决了GET请求乱码的问题(Post没有解决)

2. HttpServletResponse 类
   - 请求进入tomcat，会创建一个Response对象传递个servlet程序使用，
   - 我们通过HttpServletResponse对象来进行设置
   - 字节流 getOutputStream();  常用于下载（传递二进制数据）
   - 字符流 getWriter();  常用于回传字符串（常用）// 一个response只能2选1 不然会报错
   - resp.setContentType("text/html; charset=UTF-8");// 中文乱码

## Filter

| 生命周期阶段 | 执行时机         | 生命周期方法                             |
| ------------ | ---------------- | ---------------------------------------- |
| 创建对象     | Web应用启动时    | init方法，通常在该方法中做初始化工作     |
| 拦截请求     | 接收到匹配的请求 | doFilter方法，通常在该方法中执行拦截过滤 |
| 销毁         | Web应用卸载前    | destroy方法，通常在该方法中执行资源释放  |

```xml
// Filter 运行在服务器，对请求资源过滤(编码，过滤敏感字，事务统一调度，统一跳转错误页面)，优先于请求资源之前执行
    <servlet> // 精准匹配
        <servlet-name>servletDemo01</servlet-name>
        <servlet-class>com.atguigu.ServletDemo01</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>servletDemo01</servlet-name>
        <url-pattern>/ServletDemo01</url-pattern>
    </servlet-mapping>

    <filter-mapping>// 模糊匹配
        <filter-name>Target02Filter</filter-name>
        // 模糊匹配：前杠后星 
        //    /user/demo01
        //   /user/demo02
        //    /user/demo03
        //  /demo04
        <url-pattern>/user/\* </url-pattern>
    </filter-mapping>

    <filter> // 扩展名匹配
        <filter-name>Target04Filter</filter-name>
        <filter-class>com.atguigu.filter.filter.Target04Filter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>Target04Filter</filter-name>
        <url-pattern>*.png</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>Target05Filter</filter-name>
        // 根据Servlet名称匹配(很少用)
        <servlet-name>Target01Servlet</servlet-name>
    </filter-mapping>

// 过滤器链: 过滤器链中每一个Filter执行的顺序是由web.xml中filter-mapping配置的顺序决定的。如果某个Filter是使用ServletName进行匹配规则的配置，那么这个Filter执行的优先级要更低
// web.xml代码
    <servlet>
        <servlet-name>servletDemo01</servlet-name>
        <servlet-class>com.atguigu.ServletDemo01</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>servletDemo01</servlet-name>
        <url-pattern>/ServletDemo01</url-pattern>
    </servlet-mapping>

    <filter-mapping>
        <filter-name>CloseConnectionFilter</filter-name>
        <url-pattern>/protected/orderClient</url-pattern>
        <url-pattern>/manager</url-pattern>
        <url-pattern>/index.html</url-pattern>
        <url-pattern>/user</url-pattern>
        <url-pattern>/protected/cart</url-pattern>
    </filter-mapping>

// ServletDemo01代码
    public class ServletDemo01 extends HttpServlet {
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            doGet(request, response);
        }
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            System.out.println("ServletDemo01接收到了请求...");}}
// 创建多个Filter拦截Servlet
    <filter-mapping>
        <filter-name>TargetChain03Filter</filter-name>
        <url-pattern>/Target05Servlet</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>TargetChain02Filter</filter-name>
        <url-pattern>/Target05Servlet</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>TargetChain01Filter</filter-name>
        <url-pattern>/Target05Servlet</url-pattern>
    </filter-mapping>
```

```java
通过在filter中写入统一的操作 来避免业务代码冗余，耦合度高

    try {// 统一开启事务
        JDBCUtil.getConnection();
        JDBCUtil.startTransaction();
        chain.doFilter(req, resp);
        JDBCUtil.commit();
    } catch (Exception e) {
        e.printStackTrace();
        try {
            JDBCUtil.rollback();
        } catch (Exception ex) {
            ex.printStackTrace();
            throw new RuntimeException(e.getMessage());}
        throw new RuntimeException(e.getMessage());}


    try { // 统一关闭连接
            chain.doFilter(req, resp);
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e.getMessage());
        } finally { JDBCUtil.releaseConnection();}


    HttpServletRequest request = (HttpServletRequest) req;
        try {// 错误统一跳转到 错误界面
            chain.doFilter(req, resp);
        } catch (Exception e) {
            e.printStackTrace();
            request.getRequestDispatcher("/WEB-INF/pages/error.html").forward(request,resp);}


    try { // 路径请求检查是否登录 同步，异步
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;
        User user = (User) request.getSession().getAttribute(BookStoreConstants.USERSESSIONKEY);
        if (user == null) {
            String method = request.getParameter("method");
            if("toCartPage".equals(method) || "cleanCart".equals(method) || "checkout".equals(method)){
                response.sendRedirect(request.getContextPath()+"/user?method=toLoginPage");
            }else {
                JsonUtils.writeResult(response, CommonResult.error().setMessage("unLogin"));}
            return;}
        chain.doFilter(req, resp);
    } catch (Exception e) {
        e.printStackTrace();
        throw new RuntimeException(e.getMessage());}
```

## Listener

## 加密

常见加密( Cipher, SecretKeySpec )

- 对称加密DES / AES base64
- 非对称加密
  - 常见算法 RSA / ECC
  - KeyPairGenerator
- 加密模式: ECB / CBC
- 填充模式: NoPadding / PKCS5Padding
- 消息摘要: MD5 / SHA1 / SHA256 / SHA512

数字签名: Signature

## Restful API接口规范包括以下部分

一、协议

API与用户的通信协议，总是使用HTTPs协议。

二、域名

应该尽量将API部署在专用域名之下，如<https://api.专属域名.com；如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下，如https://专属域名.com/api/>。

三、版本

可以将版本号放在HTTP头信息中，也可以放入URL中，如<https://api.专属域名.com/v1/>

四、路径

路径是一种地址，在互联网上表现为网址，在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数，如<https://api.专属域名.com/v1/students>。

五、HTTP动词

对于资源的具体操作类型，由HTTP动词表示，HTTP动词主要有以下几种，括号中对应的是SQL命令。

1. GET（SELECT）：从服务器取出资源（一项或多项）；

2. POST（CREATE）：在服务器新建一个资源；

3. PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）；

4. PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）；

5. DELETE（DELETE）：从服务器删除资源；

6. HEAD：获取资源的元数据；

7. OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

六、过滤信息

如果记录数量很多，服务器不可能都将它们返回给用户，API会提供参数，过滤返回结果，常见的参数有：

1. ?limit=20：指定返回记录的数量为20；

2. ?offset=8：指定返回记录的开始位置为8；

3. ?page=1&per_page=50：指定第1页，以及每页的记录数为50；

4. ?sortby=name&order=asc：指定返回结果按照name属性进行升序排序；

5. ?animal_type_id=2：指定筛选条件。

七、状态码

服务器会向用户返回状态码和提示信息，以下是常用的一些状态码：

1. 200 OK - [GET]：服务器成功返回用户请求的数据；

2. 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功；

3. 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）；

4. 204 NO CONTENT - [DELETE]：用户删除数据成功；

5. 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作；

6. 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）；

7. 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的；

8. 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作；

9. 406 Not Acceptable - [GET]：用户请求的格式不可得；

10. 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的；

11. 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误；

12. 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。

八、错误处理

如果状态码是4xx，就会向用户返回出错信息，一般来说，返回的信息中将error作为键名，出错信息作为键值。

九、返回结果

针对不同操作，服务器向用户返回的结果应该符合以下规范：

1. GET /collection：返回资源对象的列表（数组）；

2. GET /collection/resource：返回单个资源对象；

3. POST /collection：返回新生成的资源对象；

4. PUT /collection/resource：返回完整的资源对象；

5. PATCH /collection/resource：返回完整的资源对象；

6. DELETE /collection/resource：返回一个空文档。

十、Hypermedia API

RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。

以上是Restful API设计应遵循的十大规范，除此之外，Restful API还需注意身份认证应该使用OAuth 2.0框架，服务器返回的数据格式，应该尽量使用JSON，避免使用XML。
