# Spring相关

## 参数校验及错误返回

* 使用注解校验前端传入接口的参数
* 使用异常拦截类，返回定制化错误信息。

```java
//参数类
public class Student{
    @NotNull(message="传入的姓名为null，请传值")
    @NotEmpty(message= "传入的姓名为空字符串，请传值")
    private String name;  // 姓名
    @Min(value=0,message="")
    @Max(value=100,message="")
    private Integer score;
    @Length(min=11,max=11,message="")
    private String mobile;
}

//入口
@PostMapping("/add")
public String addStudent(@RequestBody @Valid Student student){
    ....
}

//错误处理
public class GlobalExceptionInterceptor{
    @ExceptionHandler(value=Exception.class)
    public String exceptionHandler(HttpServletRequest request,Exception e){
        String failMsg=null;
        if(e instanceof MethodArgumentNotValidException){
            failMsg=((MethodArgumentNotValidException)e).getBindingResult().getFieldError().getDefaultMessage();
        }
        return failMsg;
    }
}
```

## 手写注解

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Length
{
    int min();// 允许字符串长度的最小值
    int max();// 允许字符串长度的最大值
    String errorMsg();// 自定义的错误提示信息
}
```

1. 注解的成员变量只能使用基本类型、 `String`或者 `enum`枚举，比如 `int`可以，但 `Integer`这种包装类型就不行，需注意 
2.  `@Target(xxx)` 用来说明该自定义注解可以用在什么位置，比如：
   1. ElementType.FIELD`：说明自定义的注解可以用于类的变量`
   2. ElementType.METHOD`：说明自定义的注解可以用于类的方法`
   3. ElementType.TYPE`：说明自定义的注解可以用于类本身、接口或 `enum`类型
3. `@Retention(xxx)` 用来说明你自定义注解的生命周期 
   1. `@Retention(RetentionPolicy.RUNTIME)`：表示注解可以一直保留到运行时，因此可以通过反射获取注解信息
   2. `@Retention(RetentionPolicy.CLASS)`：表示注解被编译器编译进 `class`文件，但运行时会忽略
   3. `@Retention(RetentionPolicy.SOURCE)`：表示注解仅在源文件中有效，编译时就会被忽略

```java
public static String validate(Object object) throws IllegalAccessException{
    //首先通过反射获取object对象的类有哪些字段
    //对本文来说就可以获取到Student类的id、name、mobile三个字段
    Field[] fields=object.getClass().getDeclaredFields();
    //for循环逐个字段校验，看哪个字段上标了注解
    for(Field field:fields){
        if(field.isAnnotationPresent(Length.class)){
            //通过反射获取到该字段上标注的@Length注解的详细信息
            Length length=field.getAnnotation(Length.class);
            field.setAccessible(true);// 让我们在反射时能访问到私有变量
            //用过反射获取字段的实际值
            int value=((String)field.get(object)).length();
            //将字段的实际值和注解上做标示的值进行比对
            if(value<length.min()||value>length.max()){
                return length.errorMsg();
            }
        }
    }
    return null;
}

```

