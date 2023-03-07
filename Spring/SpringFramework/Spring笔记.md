## 一、注解自动装配

byName:会自动在容器上下文中自动查找，和自己对象中set方法参数名相同的bean的id名
byType:会自动在容器上下文中自动查找，和自己对象中set方法参数类型相同的bean

- @Autowired（byType）:可以直接在属性上使用（此时可以省略相应的set方法），也可以在相应的set方法上使用。如果Autowired不能唯一自动装配，需要通过@Qualifier(value="XXX")

- @Resource（先byName，后byType）

- @Nullable：如果属性标记了这个注解，说明该属性值可以为null



- @Component：组件，放在类上，被context:component-scan扫描，说明该类被Spring管理了，即创建了bean。相当于<bean id="user" class="com.yan.pojo.User"/>

    ​		衍生注解，功能相似，在web开发中，会按照MVC三层架构分层！

    - dao 【@Repository】
    - service 【@Service】
    - controller 【@Controller】

    **这四个注解的功能是一样的，都是将某个类注册到Spring中装配！**

    ```java
    @Component("user") 作用在类上
    ```

    

    

- @Value("XXX")：给某个属性注册值。相当于<property name="name" value="XXX"/>

```java
@Value("yanchen") 作用在字段或set方法上
```



- @Scope("") 

    - prototype：多个
    - singleton: 单例
    - session:会话
    - request:请求

    ```java
    @Scope("prototype") 作用在类上
    ```



- @ComponentScan("com.yan.pojo.User")：扫描某个配置类（appConfig.java）

- @Import(appConfig.class)：将配置类appConfig.java导入到另一个配置类中，作用和XML配置文件中import标签一致（相当于<import resource="beans.xml"/>）

- @Configuration：代表该类是一个配置类，相当于之前的beans.xml
- @Bean：该方法的作用相当于之前的一个bean标签
    - 方法名=bean标签中的id属性
    - 返回值=bean标签中的class属性





## 二、SpringAOP

>目前最流行的AOP框架有SpringAOP和AspectJ。
>
>SpringAOP：使用纯Java实现，不需要专门的编译过程和类加载器，在运行期间通过代理方式向目标类织入增强代码。
>
>AspectJ：从Spring2.0开始支持，是一个基于Java语言的AOP框架，提供了一个专门的编译器，在编译时提供横向代码的织入。



SpringAOP：

- JDK动态代理
- CGLIB代理

AspectJ：

- 基于XML的声明式AspectJ
- 基于Annotation的声明式AspectJ



## 三、SpringMVC

### 3.1、常用注解

```text
@ResponseBody ：添加后该方法的返回值不会走视图解析器，会直接返回一个字符串

@RestController ： 该类下的所有方法只会返回一个字符串，不会走视图解析器
```



### 3.2、RestFul风格

>```java
>Restful就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。
>
>原来的风格 ：http://localhost:8080/add?a=1&b=2
>RestFul风格 : http://localhost:8080/add/{a}/{b}
>```



@PathVariable : 让方法参数的值对应绑定到一个URI模板变量上。

@RequestMapping ：能够处理 HTTP 请求的方法, 比如 GET, PUT, POST, DELETE 以及 PATCH。

**限制请求的方法，可以实现请求路径一样，但功能却可以不一样**

*@GetMapping ("请求路径")*：相当于@RequestMapping (value="请求路径",method="RequestMethod.GET")
*@PostMapping*
*@PutMapping*
*@DeleteMapping*
*@PatchMapping*



```java
//@RequestMapping("/add/{a}/{b}") 一般的请求，不限制请求方法
//@RequestMapping(path/value = "/add/{a}/{b}",method = RequestMethod.POST) 限制请求方法为POST
//@GetMapping("/add/{a}/{b}") 限制请求方法为GET
public String test1(@PathVariable int a, @PathVariable int b, Model model){
    int res=a+b;
    model.addAttribute("msg","结果为："+res);
    return "test";
}
```



### 3.3、重定向和转发

>重定向和转发均不同通过**视图解析器**
>
>重定向不能定向到WEB-INF下的页面，因为WEB-INF目录下是服务器下的，对用户来说是不可见的。
>
>

```java
	@RequestMapping("/t1")
    public String test1(Model model){
        model.addAttribute("msg","RedirectTest1");
        return "redirect:index.jsp";//重定向
    }

    @RequestMapping("/t2")
    public String test2(Model model){
        model.addAttribute("msg","RedirectTest2");
        return "forward:/WEB-INF/jsp/admin/admin.jsp";//转发
    }
```



**重定向的问题，转发则不会出现这样的问题**

```java
@RequestMapping("/r/t1") //这样的请求路径会出错
public String test1(Model model){
    model.addAttribute("msg","RedirectTest1");
    return "redirect:index.jsp";//重定向
}

类型 状态报告
消息 文.件[/r/index.jsp] 未找到
描述 源服务器未能找到目标资源的表示或者是不愿公开一个已经存在的资源表示。
```



### 3.4、数据格式转换

#### 时间日期转换

1.解决方式一：注解

前端传来的是字符串格式的日期，后端以Date的格式接收

```java
public String findOrder3(@DateTimeFormat(pattern = "yyyy-MM-ddTHH:mm") Date orderDateTime, Model model){
        model.addAttribute("orderTime3",orderDateTime);
        System.out.println(orderDateTime);

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm");
        System.out.println(sdf.format(orderDateTime));

        return "success";
}
```



前端传来的是字符串格式的日期，后端以实体类接收

```java
//实体类
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    private int id;
    private User user;

    // @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date orderDateTime;
}

//控制器
@Controller
public class OrderController {

    @RequestMapping("/o1")
    public String findOrder(Order order, Model model){
        System.out.println(order);
        model.addAttribute("orderId",order.getId());
        model.addAttribute("orderUser",order.getUser());
        model.addAttribute("orderTime1",order.getOrderDateTime());
        return "success";
    }
}

```



2.解决方式二：转换器实现

```java
import org.springframework.core.convert.converter.Converter;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateConverter implements Converter<String, Date> {

    @Override
    public Date convert(String source) {
        String  datePatten="yyyy-MM-dd HH:mm:ss";
        SimpleDateFormat  sdf = new SimpleDateFormat(datePatten);
        Date date=null;
        try {
            date = sdf.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}

```



```xml
<!--    配置转换器-->
    <bean id="converter" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.yan.controller.shiyan8.converter.DateConverter"/>
            </set>
        </property>
    </bean>
    <mvc:annotation-driven conversion-service="converter"/>
```





