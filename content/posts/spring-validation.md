---
title: "Spring 集成Javax Validation 和 Hibernate Validator" # 文章标题
date: 2023-08-24 # 文章编写日期
tags: # 文章所属标签
  ["Spring"]
featuredImage: "https://s2.loli.net/2024/01/23/dNxfvZVi97lMgWz.png"
featuredImagePreview: "https://s2.loli.net/2024/01/23/dNxfvZVi97lMgWz.png"
draft: false
---
<!--more-->

> 当需要在 controller 层对参数进行验证的时候，可以基于 javax-validation 的注解，结合 hibernate-validator 的验证实现器，以实现参数的校验。[Spring 的官方文档参考](https://docs.spring.io/spring-framework/docs/5.3.29/reference/html/core.html#validation )  
> 如果是使用 springboot 的小伙伴，则更加简便，因为 springboot 的 starter-web本身已经集成了 hibernate-validator 的依赖，因为只需要引入 javax-validation 的依赖即可。

## Spring 集成 javax-validation
我们项目用的 Spring 版本是 5.3.x，javax-validation 的版本是 2.0.1.Final。  
在没有 hibernate-validator 的情况下，需要在 controller显式声明 BindingResult 这个参数，用于捕获验证结果，以下是一个例子：

```java
public class User {
    @NotNull
    private String id;
    //gettter and setter...
}

@RestController
public class UserController {
    @GetMapping(path = "/test")
    public void test(@RequestBody @Validated User user, BindingResult result) {
        if (result.hasErrors()) {
            // process error..
        }
    }
}
```


## Spring 集成 javax-validation 和 hibernate-validator
显然，这样的错误处理太 hard code 了，并且会导致大量的重复代码，因此，需要引入 hibernate-validator 的依赖，我的版本是5.0.1.Final，它会自动捕获错误，并抛出异常。
以下是一个例子：

```java
@Validated
@RestController
public class UserController {
    @GetMapping(path = "/test")
    public void test(@RequestBody @Valid User user) {
        // service call
    }
}
```

基于 hibernate 的验证处理，当验证失败的时候，会自动抛出一个异常：`MethodArgumentNotValidException`，可以看下这个异常继承了`BindingException`类，
```java
public class MethodArgumentNotValidException extends BindException
```

`BindingException`类又实现了`BindingResult`接口，重写了 getMessage方法，来拼接 BindingResult 中得到的信息

```java
public class BindException extends Exception implements BindingResult {
    @Override
    public String getMessage() {
        StringBuilder sb = new StringBuilder("Validation failed for argument [")
            .append(this.parameter.getParameterIndex()).append("] in ")
            .append(this.parameter.getExecutable().toGenericString());
        BindingResult bindingResult = getBindingResult();
        if (bindingResult.getErrorCount() > 1) {
            sb.append(" with ").append(bindingResult.getErrorCount()).append(" errors");
        }
        sb.append(": ");
        for (ObjectError error : bindingResult.getAllErrors()) {
            sb.append('[').append(error).append("] ");
        }
        return sb.toString();
    }
}
```
当验证错误都统一捕获并抛出特定异常之后，我们就可以结合Spring 的 `@RestControllerAdvice`，对抛出的异常统一处理，这里不多做介绍了。