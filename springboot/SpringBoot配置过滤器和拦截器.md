## 前言
本篇博客为记录在SpringBoot项目中配置过滤器和拦截器的方法。
## 过滤器
过滤器（Filter）是Servlet中常用的技术，可以实现用户在访问某个目标资源之前，对访问的请求和响应进行拦截，常用的场景有登录校验、权限控制、敏感词过滤等，下面介绍下Spring Boot配置过滤器的两种方式。
### 1. @WebFilter注解方式
使用 @WebFilter 注解为声明当前类为filter，第一个参数为该filter起一个名字，第二个参数为说明要拦截的请求地址，当前类需要实现Filter接口，里面有三个方法，分别为过滤器初始化、过滤方法和过滤器销毁。
```java
@WebFilter(filterName = "myFilter1", urlPatterns = "/*")
public class MyFilter1 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info(filterConfig.getFilterName() + " init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        log.info("myFilter1 begin");
        try {
            log.info("业务方法执行");
            chain.doFilter(request, response);
        } catch (Exception e) {
            log.error("error!", e);
        }
        log.info("myFilter1 end");
    }

    @Override
    public void destroy() {
    }
}
```
启动类添加@ServletComponentScan注解，@ServletComponentScan注解所扫描的包路径必须包含该Filter，代码如下:
```java
@SpringBootApplication
@ServletComponentScan(basePackages = "com.dave.demo.filter")
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```
### 2. @Bean注解方式
新建一个过滤类继承Filter，不加注解@WebFilter：
```java
public class MyFilter2 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info(filterConfig.getFilterName() + " init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        log.info("myFilter2 begin");
        try {
            log.info("业务方法执行");
            chain.doFilter(request, response);
        } catch (Exception e) {
            log.error("error!", e);
        }
        log.info("myFilter2 end");
    }

    @Override
    public void destroy() {
    }
}
```
新建配置类WebConfig.java，配置bean，代码如下：
```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean testFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean(new MyFilter2());
        registration.addUrlPatterns("/test"); //
        registration.setName("myFilter2");
        return registration;
    }
}
```
## 拦截器
配置拦截器
```java
@Component
public class InitInterceptor extends HandlerInterceptorAdapter {
	private static final Logger logger = LoggerFactory.getLogger(InitInterceptor.class);
 
 
	/**
	 * 在执行controller方法之前进行请求参数处理
	 */
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
 
		if (条件) {
			dosomething...
			// 拦截
			return false;	
			
		}
		// 放行
        return true;
	}
 
 
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
 
	}
 
}
```
拦截器配置类（需要springboot扫描到注解）
```java

@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
	
	@Autowired
	private InitInterceptor initInterceptor; 
	
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		//设置（模糊）匹配的url
        List<String> urlPatterns = Lists.newArrayList();
        urlPatterns.add("/login");
        urlPatterns.add("/user/*");
        urlPatterns.add("/admin/*");
		registry.addInterceptor(initInterceptor).addPathPatterns(urlPatterns);
	}
```