## 前言
本篇博客为记录 Spring 注解中 @validated 注解的使用及常见的注解类型

## 基础使用
因为spring-boot已经引入了基础包，所以直接使用就可以了。
1. 在bean类中声明需要校验的字段
```java
@data
public class XoPO{
    
	// 嵌套校验
    @validated
    private List<OrderPerson> personList;
    
    @NotNull(message = "code不能为空")
    @Size(max=32,message="code is null")
    private String code;

    @NotBlank
    @Size(max=32,message="product is null")
    private String product;
}
```
2. 在controller层的方法上声明需要对数据进行校验，即加上@Validated注解
```java
@PostMapping(value="/xxx")
public HttpResult xxmethod( @RequestBody @Validated  XoPO xoPo) {}
```
3. 参数不是实体类时可直接在参数上加校验
```java
@GetMapping(value="/xxx")
public HttpResult xxmethod( @RequestParam("id") @NotNull(message = "id不能为空") Long id,
@RequestParam("name") @NotBlank(message = "name不能为空") String name) {}
```
当输入不能满足条件时，会抛出异常，而后统一由异常中心处理
也可以用BindingResult，但是用了这个后就必须手动处理异常，侵入了正常的逻辑过程，并不推荐。

## 常用注解类型
```java
@AssertFalse // 校验false
@AssertTrue // 校验true
@DecimalMax(value=,inclusive=) // 小于等于value,inclusive=true,是小于等于
@DecimalMin(value=,inclusive=) // 与上类似
@Max(value=) // 小于等于value
@Min(value=) // 大于等于value
@NotNull  // 检查Null
@Past  // 检查日期
@Pattern(regex=,flag=)  // 正则
@Size(min=, max=)  // 字符串，集合，map限制大小
@Validate // 对po实体类进行校验
```

