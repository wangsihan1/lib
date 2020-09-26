# springmvc笔记

### springmvc的优势
有清晰的角色划分：
+ 前端控制器（DispatcherServlet）
+ 请求到处理映射器（HandlerMapping）
+ 处理适配器( HandlerAdapter)
+ 视图解析器（ViewResolver）
+ 处理器或页面控制器（Controller）
+ 验证器（Validator）

### 构建maven项目运行第一个springmvc程序

+ 在WEB-INF中配置前端控制器

```xml
<!--  配置前端控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    
     <!--让springmvc的配置文件生效  让服务器开启后配置文件就生效-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

+ 在resources配置文件夹中配置springmvc的配置文件 开启注解扫描

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans  http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/mvc        http://www.springframework.org/schema/mvc/spring-mvc.xsd
          http://www.springframework.org/schema/context    http://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启注解扫描       扫描com.biubixin.controller此包下的注解-->
    <context:component-scan base-package="com.biubixin.controller"></context:component-scan>
    
        <!--开启springmvc注解支持-->
    <mvc:annotation-driven/>
    
</beans>
```

+ 创建控制器类

```java
//控制器类
@Controller
@RequestMapping("/user")
public class Hello {

    @RequestMapping(path = "/hello")
    public String add(){

        System.out.println("lalalalalala");

        return "success";
    }
}
```
springmvc默认返回的字符串是jsp文件的名字，所以在webapp里创建一个名字叫success.jsp的文件，但是要实现这样的跳转，我们还需要在springmvc.xml中配置视图解析器

```xml
    <!--创建视图解析器对象-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--跳转到什么文件夹下的  什么类型的文件-->
        <property name="prefix" value="/pages/"></property>
        <property name="suffix" value=".jsp"></property>
        
    </bean>
```

### @RequestMapping的作用

+ 用于建立请求url和处理请求方法之间的对应关系。
+ 可以放方法上，也可以放在类上

### @RequestMapping的属性
+ @RequestMapping（method={RequestMethod.POST}）   决定此方法的具体请求方式
+ @RequestMapping（param={username}）    指定限制请求参数的条件  如果请求参数中没有则请求失败

### 请参数的绑定

```html
<a href="login?username=wangsihan">zhuce</a> 
```
如果想把username传到后台  只需要在方法参数里传入对应的参数名  框架自动把值封装进去

```java
    @RequestMapping("/login")
    public String a(String username){
        System.out.println(username);
        return "success";
    }
```

+ 如果传入JavaBean中有引用类型对象参数  

```java
public class Account  {
    private User user;
```

则在前端页面参数名为
```html
        <input type="text" name="user.uname"><br/>
        <input type="text" name="user.age"><br/>
```

+ 解决中文参数乱码问题
   在web.xml中配置过滤器
   
```xml
  <!--  解决post请求中文乱码的过滤器   过滤器要放在servlet上面 不然可能会出错  get请求好像tomcat自己会解决-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```
+ 请求参数绑定集合类型

```java
    private List<User>list;
    private Map<String,User>map;
```
在jsp中这样命名
```jsp
        <input type="text" name="list[0].uname"><br/>
        <input type="text" name="list[0].age"><br/>

        <input type="text" name="map['one'].uname"><br/>
        <input type="text" name="map['one'].age"><br/>
```

### 自定义类型转换器

+ 定义一个类，实现Converter接口
```java
import org.springframework.core.convert.converter.Converter;

import java.util.Date;

public class StringToDateConverter implements Converter<String,Date> {

    public Date convert(String s) {
        
        //这里写逻辑代码
        return null;
    }
}
```

+ 在springmvc中注册

```xml
    <!--配置自定义类型转换器-->
    <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.biubixin.util.StringToDateConverter"></bean>
            </set>
        </property>
    </bean>

    <!--开启springmvc注解支持-->
    <mvc:annotation-driven conversion-service="conversionService"/>
```

### 获取Servlet原生API
```
    @RequestMapping("/yuan")
    public String yuan(HttpServletRequest request){
        return "success";
    }
```
+ 直接在方法参数中写想要的API

### Restful编程风格
+ 请求地址一样  但请求的方式不同 例如：post  get put..

如果 两个方法的请求方式一样则在路径中加入相应的数字或字符  详见常用注解的@PathVariable。

### 常用注解
#### @RequestParam

```java
@RequestMapping("/textanno")
    public String textanno(@RequestParam(name = "username") String us){
        return "success";
    }
```
假如前端name和参数name不一样  则可在@RequestParam里写前端name值  到时候这个值传到后台就会和方法参数里的参数进行绑定

#### @RequestBody

获取请求体内容，直接使用得到的是key=value&key=value..的键值对  get请求方式不适用
```java
    @RequestMapping("/textanno")
    public String textanno(@RequestBody String body){
        return "success";
    }
```
这样String body 就被赋值了body中的值

####  @PathVariable

用于绑定url中的占位符  例如请求url中/delete/{id},这个{id}就是占位符。  是springmvc支持rest风格的重要标志。

前端页面请求：

```html
<a href="anno/textanno/10">点我</a>
```
后端绑定数据
```java
    @RequestMapping("/textanno/{id}")
    public String find(@PathVariable(name = "id")String s){
        System.out.println(s);
        return "success";
    }
```

#### @RequestHeader

用于获取请求消息头

```java
    @RequestMapping("/textanno")
    public String textanno(@RequestHeader(name = "Accept") String body){
        System.out.println(body);
        return "success";
    }
```

#### @CookieValue

拿到指定cookie的值
```java
    @RequestMapping("/cookie")
    public String textanno1(@CookieValue(value = "JSESSIONID") String body){
        System.out.println(body);
        return "success";
    }
```

#### @ModelAttribute
   可以用于修饰方法或参数上

	出现在方法上：表示当前方法回在控制器的方法执行之前，先执行。
	出现在参数上，获取指定的数据给参数赋值。

####  @SessionAttributes

   只能作用在类上

```java
    @RequestMapping("/sessioncun")
    public String fdfdf(Model model){
        model.addAttribute("msg","meimeidfafa");
        System.out.println(model);
        return "success";
    }
```
```jsp
	在session域中也存入msg
	@SessionAttributes(value = {"msg"})
	public class Anno {、
	
jsp中这样取值：
    ${ sessionScope }
```


## 响应

### 返回值是String类型

+ 创建一个JavaBean 有以下属性和getset方法 有参构造和空参构造。
```java
    private String username;
    private String password;
    private Integer age;
```
+ 创建一个controller类

```java
    @RequestMapping("/s")
    public String textstring(Model model){//model能把值存入request对象域中
        System.out.println("text方法执行了");
        User user = new User("美美","123",18);
        model.addAttribute("user",user);
        return "success";
    }
```
+ 然后可以在jsp中取到值

```jsp
${ user.username }
${ user.password }
${ user.age }
```
### 返回值是void类型

用到了request的请求转发
```java
    @RequestMapping("/v")
    public void textvoid(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("void方法执行了");
        //请求转发方式
        //手动转发的时候不会走视图解析器，需要自己提供完整的目录
        request.getRequestDispatcher("/pages/success.jsp").forward(request,response);
        
           //重定向方式  无法访问WEB-INF文件中的东西
        response.sendRedirect(request.getContextPath()+"/pages/success.jsp");
    }
```

### 返回值是ModelAndView类型

其实第一种返回字符串类型底层调用的也是这个方式

```java
    @RequestMapping("/m")
    public ModelAndView textmodelandview(){  
        ModelAndView mv = new ModelAndView();
        User user = new User("美美","123",18);

        //把user存入mv中，mv也会把user存入request中  底层还是和model的方法一样
        mv.addObject("user",user);

        //跳转到哪个页面 (使用的是视图解析器 ，和return “success”效果一样)
        mv.setViewName("success");
        return mv;
    }
```

#### 使用关键字进行转发和跳转（了解）

```java
    @RequestMapping("/guanjianzi")
    public String guanjianzi(){ 
        //请求的转发
       return "forward:/pages/success.jsp";
        //重定向
        return "redirect:/pages/success.jsp";
    }
```
### @ResponseBody响应json数据 


防止前端控制器拦截静态文件  使引用的静态文件无法访问
```xml
<!--    告诉前端控制器以后哪些静态资源不被拦截-->
    <mvc:resources mapping="/js/" location="/js/**"></mvc:resources>
```

+ 前端通过ajax把json数据发送给后端

```javascript
    $("#btn").click(function () {
        $.ajax({
            url:"http://localhost:8080/springmvctext02/user/textjson",
            contentType:"application/json;charset=UTF-8",
            data:'{"username":"跨域","password":"123","age":30}',
            dataType:"json",
            type:"post",
            success:function (data) {
                /data表示服务器响应的json数据
              }
          });
      })
```
+ 后端通过@RequestBody注解 把前端数据注入到String body中

```java
    @RequestMapping(value = "/textjson",method = RequestMethod.POST)
    public void textjson(@RequestBody String body){
        System.out.println("fangfazhixinl");
        System.out.println(body);
    }
```

+ 返回给浏览器json数据格式

```java
    @RequestMapping(value = "/textjson03",method = RequestMethod.POST)
    public @ResponseBody Boolean textjson03(){
        Boolean  a = true;
        return a;
    }
```
在返回类型前加入注解@ResponseBody


## 异常处理

+ 编写自定义异常类（做提示信息的）
+ 编写异常处理器
+ 配置异常处理器（跳转到提示页面）

创建自定义异常类
```java
  //自定义异常类
public class YiChang  extends Exception{
    //存储提示信息
    private String msg;
    public YiChang(String msg) {
        this.msg = msg;
    }
    public String getMsg() {
        return msg;
    }
    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

 在controller中模拟异常
```java
     @RequestMapping("text")
    public String text() throws YiChang {
        //模拟异常
        try {
            int i =10/0;
        } catch (Exception e) {
            //这是打印异常
            e.printStackTrace();
            //抛出自定义异常信息
            throw new YiChang("查询出错了");
        }
        return "success";
    }
```
编写异常处理器
```java
//异常处理器
public class YiChangResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        YiChang p = null;
        if (e instanceof YiChang){
            p = (YiChang) e;
        }else {
            new YiChang("系统在维护");
        }
        ModelAndView mv = new ModelAndView();
        mv.addObject("erroMsg",e.getMessage());
        mv.setViewName("error");//跳转页面
        return mv;
    }
}
```
配置异常处理器  在springmvc.xml中
```xml
    <!--配置异常处理器  路径为异常处理器的路径-->
    <bean id="yiChangResolver" class="com.biubixin.exception.YiChangResolver"/>
```
最后建一个叫error的错误页面即可

## springmvc中的拦截器

>  + 编写拦截器，实现HandlerInterceptor接口
>  +  配置拦截器

+ 新建一个类MyInter1，为自定义拦截器

```java
//自定义拦截器
public class MyInter1 implements HandlerInterceptor {
    //预处理 ，在controller方法执行前
    //return true 放行，执行下一个拦截器，如果没有则执行controller方法
    //return false  不放行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInter1执行了");
        return true;
    }
}
```

+ 在springmvc.xml中配置拦截器

```xml
    <!--配置拦截器-->
    <mvc:interceptors>
        <mvc:interceptor>
            <!--你要拦截的方法-->
            <mvc:mapping path="/user/**"/>
            <!--你不拦截的方法
            <mvc:exclude-mapping path=""/>
            -->
            <!--配置拦截器对象-->
            <bean class="com.biubixin.inter.MyInter1"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

+ 	后处理和页面执行后处理

```java
    //后处理 controller执行完后执行 （controller执行后，success.jsp执行前）
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }
    //success.jsp执行之后才执行
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
```

+ 多个拦截器的配置

```
在 <mvc:interceptors> </mvc:interceptors>标签下顺序配置， 拦截器的执行顺序是在标签中的顺序。
```
