## 步骤
1. 添加依赖

```xml
<!-- 使用Shiro认证 -->

<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.2.5</version>
</dependency>
```
2. 在Controller接口类/属性/方法处增加@RequiresPermissions 注解，注解内的参数可以自定义

```java
@RequestMapping("add")
@RequiresPermissions("user:add")
public String add() {
     return "add";
}
```

3. 在 ShiroConfig 类中配置

```java
/**
     * 
     * @Title:        advisorAutoProxyCreator 
     * @Description:   开启Shiro的注解(如@RequiresRoles,@RequiresPermissions),需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
     *				      配置以下两个bean(DefaultAdvisorAutoProxyCreator(可选)和AuthorizationAttributeSourceAdvisor)即可实现此功能
     * @param:        @return    
     * @return:       DefaultAdvisorAutoProxyCreator    
     * @author        dave
     * @date          2020年9月1日 下午8:06:04
     */
    @Bean
    public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator(){
        DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        advisorAutoProxyCreator.setProxyTargetClass(true);
        return advisorAutoProxyCreator;
    }
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(){
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager());
        return authorizationAttributeSourceAdvisor;
    }
```

4. 异常捕捉


```java

package com.samton.admin.exception;

import org.apache.shiro.authz.AuthorizationException;
import org.apache.shiro.authz.UnauthorizedException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import com.louis.kitty.core.http.HttpResult;

/**
 * 统一异常处理类<br>
 * 捕获程序所有异常，针对不同异常，采取不同的处理方式
 * shiro异常捕获
 *
 */
@ControllerAdvice
public class ExceptionHandleController {
    private static final Logger logger = LoggerFactory.getLogger(ExceptionHandleController.class);

    @ResponseBody
    @ExceptionHandler(UnauthorizedException.class)
    public HttpResult handleShiroException(Exception ex) {
    	logger.error("没有访问权限");
        return HttpResult.error("没有访问权限");
    }

    @ResponseBody
    @ExceptionHandler(AuthorizationException.class)
    public HttpResult AuthorizationException(Exception ex) {
    	logger.error("登录失败");
    	return HttpResult.error("登录失败");
    }

}
```