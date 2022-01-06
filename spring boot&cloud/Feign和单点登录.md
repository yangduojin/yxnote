

###### 主类上面添加``@EnableFeignClients`` 开启``feign``
###### 注解``@FeignClient(value,fallback)``写接口上,被类实现不过一般都是写好接口将接口引入feign中
value是提供者的名称,动态拼接路径，fallback是实现类class,失败调用对应的实现方法

ribbon / Hystrix / 请求压缩 / 日志级别

###### 远程调用是将对象转化为``json``放在请求体中,接收方不一定要同类型的对象接收,只要对象字段一样,都可以``@RequestBody``接收,最好所有远程调用feign都用``@Post``请求
``feign``远程调用设置的响应时间是1s,可能会导致没有快速响应直接报错不存在,需要设置响应时间

```xml
feign:
  client:
	config:
		default:
		connectTimeout: 50000
		readTimeout: 50000
```

##### 客户端和服务端的负载均衡
![](/spring%20boot&cloud/img/serverLoadblance.jpg)

服务端的负载均衡是一个url先经过一个代理服务器（这里是nginx），然后通过这个代理服务器通过算法（轮询，随机，权重等等）反向代理你的服务，来完成负载均衡。

客户端的负载均衡则是一个请求在客户端的时候已经通过eureka获取了要调用服务的集群信息，然后通过具体的负载均衡算法来完成调用具体某个服务。

简而言之，服务端负载均衡需要先经过nginx代理服务器才能知道调用服务的集群信息。而客户端负载均衡请求在客户端的时候就已经知道了调用服务的集群信息。

##### feign和nacos
![](/spring%20boot&cloud/img/FeignRequest.png)
feign自身可以维护服务列表,当他配合nacos时让nacos维护服务列表,定时拉去,如果nacos挂掉,可以维护之前拉取的服务列表缓存

## Feign请求头无信息
feign动态代理,远程调用是新的请求,没有原始的请求头和信息,需要一个拦截器,在远程调用之前往请求里面加入原始请求头信息,更改源码的配置类,添加拦截器(这是老的每个微服务都这么弄一个,可以在工具类里面弄一个公用的)

``` java
@Configuration
public class GlFeignConfig {
	@Bean("requestInterceptor")
	public RequestInterceptor requestInterceptor(){
		// Feign在远程调用之前都会先经过这个方法
		return new RequestInterceptor() {
			@Override
			public void apply(RequestTemplate template) {
				//RequestContextHolder spring提供的上下文保持器,就是threadlocal, 拿到刚进来的这个原始请求
				//远程调用的线程就是请求的线程,只是请求不一样了,所以没有原始的请求头
				ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
				// 获取当前的请求的信息,加入到新的feign请求里面
				if(attributes != null){
					System.out.println("feign远程调用之前,先进行RequestInterceptor.apply");
					HttpServletRequest request = attributes.getRequest();
					if(request != null){
						// 同步请求头数据
						String cookie = request.getHeader("Cookie");
						// 给新请求同步Cookie
						template.header("Cookie", cookie);}}}};}}
```

也可以这样 这样的需要在gateway里面放入id信息  
```java
request.mutate().header("userId",userId).build();
@Component
public class FeignInterceptor implements RequestInterceptor {
	public void apply(RequestTemplate requestTemplate){
			ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
			if(requestAttributes!=null){
				HttpServletRequest request = requestAttributes.getRequest();
				System.out.println("拦截器的 userTempId是" + request.getHeader("userTempId"));
				requestTemplate.header("userTempId", request.getHeader("userTempId"));
				System.out.println("拦截器的 userId是" + request.getHeader("userId"));
				requestTemplate.header("userId", request.getHeader("userId"));}}}
```


## 请求到新的域名验证是否登陆
``` java
@Configuration
	public class GlMallWebConfig implements WebMvcConfigurer {
	  @Override
	  public void addInterceptors(InterceptorRegistry registry) {
	    registry.addInterceptor(new CartInterceptor()).addPathPatterns("/**");
	  }
	}
```

拦截器如下

```java
public class CartInterceptor implements HandlerInterceptor {
public static ThreadLocal<UserInfoTo> threadLocal = new ThreadLocal<>();
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
UserInfoTo userInfoTo = new UserInfoTo();
HttpSession session = request.getSession();
MemberRsepVo user = (MemberRsepVo) session.getAttribute(AuthServerConstant.LOGIN_USER);
if (user != null){
	// 用户登陆了
	userInfoTo.setUsername(user.getUsername());
	userInfoTo.setUserId(user.getId());
}
Cookie[] cookies = request.getCookies();
if(cookies != null && cookies.length > 0){
	for (Cookie cookie : cookies) {
	String name = cookie.getName();
	if(name.equals(CartConstant.COOKIE_TEMP_USER_KEY)){
		userInfoTo.setUserKey(cookie.getValue());
		userInfoTo.setTempUser(true);
	}
	}
}
// 如果没有临时用户 则分配一个临时用户
if (StringUtils.isEmpty(userInfoTo.getUserKey())){
	String uuid = UUID.randomUUID().toString().replace("-","");
	userInfoTo.setUserKey("yxgulimall-" + uuid);
}
threadLocal.set(userInfoTo);
return true;
}
/**
* 执行完毕之后分配临时用户让浏览器保存
*/
@Override
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
UserInfoTo userInfoTo = threadLocal.get();
if(!userInfoTo.isTempUser()){
	Cookie cookie = new Cookie(CartConstant.COOKIE_TEMP_USER_KEY, userInfoTo.getUserKey());
	// 设置这个cookie作用域 过期时间
	cookie.setDomain("gulimall.com");
	cookie.setMaxAge(CartConstant.COOKIE_TEMP_USER_KEY_TIMEOUT);
response.addCookie(cookie);}}}
```
