# java基础知识

J2SE --java standard edition 标准版本
J2ME --java Micro edition  一般位于嵌入式应用，例如手机游戏
J2EE --java Enterprise Editon 一般为服务器端程序的应用

## JDK与JRE区别

JRE： Java Runtime Environment java运行时环境
JDK：Java Development Kit java开发工具包

> 如果你需要运行java程序，只需安装JRE就可以了。如果你需要编写java程序，需要安装JDK。

## 依赖注入的方式

```java
public class ClassB {
    ClassA classA;

    // 参数注入
    public void setClassA(ClassA classA) {
        this.classA = classA;
    }

    // 构造函数注入
    ClassB(ClassA classA) {
        this.classA = classA;
    }
}
```

## spring接收参数

### 请求路径参数

/demo/123?name=suki_rong

```java
@GetMapping("/demo/{id}")
public void demo(@PathVariable(name = "id") String id, @RequestParam(name = "name") String name) {
    System.out.println("id="+id);
    System.out.println("name="+name);
}
```

### Body参数

Body内容：

```json
{
    "name":"suki_rong",
    "age":18,
    "hobby":"programing"
}
```

方式一：

```java
@PostMapping(path = "/demo")
public void demo1(@RequestBody Person person) {
    System.out.println(person.toString());
}
```

> 输出结果：name:suki_rong;age=18;hobby:programing

方式二：

```java
@PostMapping(path = "/demo")
public void demo1(@RequestBody Map<String, String> person) {
    System.out.println(person.get("name"));
}
```

> 输出结果：suki_rong

### 请求头参数以及Cookie

方式一：

```java
@GetMapping("/demo")
public void demo3(@RequestHeader(name = "myHeader") String myHeader, @CookieValue(name = "myCookie") String myCookie) {
    System.out.println("myHeader=" + myHeader);
    System.out.println("myCookie=" + myCookie);
}
```

方式二：

```java
@GetMapping("/demo3")
public void demo3(HttpServletRequest request) {
    System.out.println(request.getHeader("myHeader"));
    for (Cookie cookie : request.getCookies()) {
        if ("myCookie".equals(cookie.getName())) {
            System.out.println(cookie.getValue());
        }
    }
}
```

## @Autowired和@Resource两个注解的区别

* @Autowired默认按照byType方式进行bean匹配，多个实现时用@Qualifier标识，@Resource默认按照byName方式进行bean匹配。
* @Autowired是Spring的注解，@Resource是J2EE的注解，Spring属于第三方的，J2EE是Java自己的东西。
