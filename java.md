# java

java知识点

## java基础知识
J2SE --java standard edition 标准版本
J2ME --java Micro edition  一般位于嵌入式应用，例如手机游戏
J2EE --java Enterprise Editon 一般为服务器端程序的应用

## ArrayList和LinkedList区别

ArrayList和LinkedList是两个集合类，用于存储一系列的对象引用(references)

* ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。 （LinkedList是双向链表，有next也有previous）
* 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
* 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。

* ArrayList的插入或删除一个元素意味着剩余的元素都会被移动；LinkedList插入或删除一个元素的开销是固定的。
* LinkedList不支持高效的随机元素访问。
* ArrayList空间浪费主要体现在列表的结尾预留一定的容量空间，LinkedList的空间花费则体现在它的每一个元素都需要消耗相当的空间。

> 当操作是在一列数据的后面添加数据而不是在前面或中间，并且需要随机地访问其中的元素时，使用ArrayList会提供比较好的性能；当你的操作是在一列数据的前面或中间添加或删除数据，并且按照顺序访问其中的元素时，就应该使用LinkedList了。

### Java反射
Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

### JavaConfig
选择JavaConfig的最重要原因是Bean的配置是通过Java来完成的

```java
@Configuration
public class JavaConfiguration {
    // 创建对象，使用@Bean注解
    @Bean
    public UserDao userDao() {
        return new UserDao();
    }
}
```

### 重写与重载区别
* 重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。
* 重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。

### JDK与JRE区别
JRE： Java Runtime Environment java运行时环境
JDK：Java Development Kit java开发工具包

> 如果你需要运行java程序，只需安装JRE就可以了。如果你需要编写java程序，需要安装JDK。

### @Autowired和@Resource两个注解的区别
* @Autowired默认按照byType方式进行bean匹配，多个实现时用@Qualifier标识，@Resource默认按照byName方式进行bean匹配。
* @Autowired是Spring的注解，@Resource是J2EE的注解，Spring属于第三方的，J2EE是Java自己的东西。

> 常用的注入方式主要有三种：构造方法注入，setter注入，基于注解的注入

* @Component：可以用于注册所有bean
* @Repository：主要用于注册dao层的bean
* @Controller：主要用于注册控制层的bean
* @Service：主要用于注册服务层的bean

### HashMap和Hashtable的区别
* HashMap允许空值作为键和值，而Hashtable不允许空值
* HashMap是非同步的，而Hashtable是同步的，这意味着Hashtable是线程安全的

### Hashtable和ConcurrentHashMap的区别
Hashtable是阻塞的，任何操作都会把整个表锁住。坏处是所有调用都要排队，效率较低。
ConcurrentHashMap是非阻塞的，更新时会局部锁住某部分数据，但不会把整个表都锁住。好处是在保证合理的同步前提下，效率很高。

### Java多线程
在 Java 中实现多线程有两种手段，一种是继承 Thread 类，另一种就是实现 Runnable 接口。如果一个类继承 Thread 类，则不适合于多个线程共享资源，而实现了 Runnable 接口，就可以方便的实现资源的共享。

> Java运行时至少会启动两个线程，一个是main线程，另外一个是垃圾收集线程。

## Java注解
Java 注解（Annotation）又称 Java 标注，是 JDK5.0 引入的一种注释机制。Java 语言中的类、方法、变量、参数和包等都可以被标注。和 Javadoc 不同，Java 标注可以通过反射获取标注内容。Java定义了一套注解，共有7个，前3个在java.lang中，剩下4个在java.lang.annotation中。

### 常用注解

* @Override 检查该方法是否是重写方法
* @Deprecated 标记过时方法
* @SuppressWarnings 指示编译器去忽略注解中声明的警告
* @Retention 标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问
* @Documented 标记注解是否包含在用户文档中
* @Target 标记注解应该是哪种Java成员
* @Inherited 标记该注解继承于哪个注解类

### @Target
表明该注解可以应用的java元素类型

|  Target类型  | 描述  |
|  ----  | ----  |
| ElementType.TYPE  | 应用于类、接口（包括注解类型）、枚举 |
| ElementType.FIELD  | 应用于属性（包括枚举中的常量） |
| ElementType.METHOD  | 应用于方法 |
| ElementType.PARAMETER  | 应用于方法的形参 |
| ElementType.CONSTRUCTOR  | 应用于构造函数 |
| ElementType.LOCAL_VARIABLE  | 应用于局部变量 |
| ElementType.ANNOTATION_TYPE  | 应用于注解类型 |
| ElementType.PACKAGE  | 应用于包 |
| ElementType.FIELD  | 应用于属性（包括枚举中的常量） |
| ElementType.TYPE_PARAMETER  | 1.8版本新增，应用于类型变量 |
| ElementType.TYPE_USE  | 1.8版本新增，应用于任何使用类型的语句中（例如声明语句、泛型和强制转换语句中的类型） |

### @Retention
表明该注解的生命周期

|  生命周期类型  | 描述  |
|  ----  | ----  |
| RetentionPolicy.SOURCE  | 编译时被丢弃，不包含在类文件中 |
| RetentionPolicy.CLASS  | JVM加载时被丢弃，包含在类文件中，默认值 |
| RetentionPolicy.RUNTIME  | 由JVM 加载，包含在类文件中，在运行时可以被获取到 |

### 示例如下
```java
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation1 {
}
```

## Spring
### spring接收参数
#### 请求路径参数

/demo/123?name=suki_rong

```java
@GetMapping("/demo/{id}")
public void demo(@PathVariable(name = "id") String id, @RequestParam(name = "name") String name) {
    System.out.println("id="+id);
    System.out.println("name="+name);
}
```

#### Body参数
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

#### 请求头参数以及Cookie
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
