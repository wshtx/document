### 默认配置

- 配置web.xml

```xml
<!-- springmvc 前端控制器
	SpringMVC的配置文件默认的位置和名称：WEB-INF/<servlet-name>-servlet 
-->
    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```



![image-20221201173650423](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/image-20221201173650423.png)

- 创建控制器.SpringMVC的控制器是一个普通的POJO类，使用`@Controller`标识。

- 配置SpringMVC配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8" standalone="no"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xmlns:p="http://www.springframework.org/schema/p"
         xmlns:websocket="http://www.springframework.org/schema/websocket"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
         http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
         http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
         http://www.springframework.org/schema/websocket http://www.springframework.org/schema/websocket/spring-websocket-4.0.xsd">
     
      <!-- <mvc:annotation-driven /> 会自动注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter 两个bean,是spring MVC为@Controllers分发请求所必须的。它提供了数据绑定支持，读取json的支持 -->
      <mvc:annotation-driven />
  
      <!-- 设置自动注入bean的扫描范围，use-default-filters默认为true，会扫描所有的java类进行注入 ，
  springmvc和application文件都需要配置，但mvc文件只扫描controller类，application扫描不是controller类-->
      <context:component-scan base-package="mytest.*" use-default-filters="false">
          <context:include-filter expression="org.springframework.stereotype.Controller" type="annotation"/>
      </context:component-scan>
      
       <!-- 配置jsp视图解析器 -->
      <bean id="" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
          <property name="order" value="1"/>
          <property name="characterEncoding" value="UTF-8"/>        
          <property name="templateEngine">
              <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                  <property name="prefix" value="/WEB-INF/templates/"/>
                  <property name="suffix" value=".html"/>                  
                  <property name="templateMode" value="HTML5"/>                 		
                  <property name="characterEncoding" value="UTF-8"/>  
              </bean>
          </property>
      </bean>
  
      <!-- 文件上传功能需该配置 -->
      <bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver" id="multipartResolver">
      <property name="defaultEncoding" value="UTF-8"/>
      </bean>
  
      <!-- ResourceBundleThemeSource是ThemeSource接口默认实现类-->
      <bean class="org.springframework.ui.context.support.ResourceBundleThemeSource" id="themeSource"/>
  
      <!-- 用于实现用户所选的主题，以Cookie的方式存放在客户端的机器上-->
      <bean class="org.springframework.web.servlet.theme.CookieThemeResolver" id="themeResolver" p:cookieName="theme" p:defaultThemeName="standard"/>
  
      <!-- 由于web.xml文件中进行了请求拦截
          <servlet-mapping>
              <servlet-name>dispatcher</servlet-name>
              <url-pattern>/</url-pattern>
          </servlet-mapping>
      这样会影响到静态资源文件的获取，mvc:resources的作用是帮你分类完成获取静态资源的责任
      -->
      <mvc:resources mapping="/resources/**" location="/WEB-INF/resources/" />
  
      <!-- 配置使用 SimpleMappingExceptionResolver 来映射异常 -->
      <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver" >
  
      <!-- 定义默认的异常处理页面 -->
      <property name="defaultErrorView" value="error"/>
           <!-- 配置异常的属性值为ex，那么在错误页面中可以通过 ${exception} 来获取异常的信息如果不配置这个属性，它的默认值为exception-->
          <property name="exceptionAttribute" value="exception"></property>
          <property name="exceptionMappings">
              <props>
              <!-- 映射特殊异常对应error.jsp这个页面 -->
                  <prop key=".DataAccessException">error</prop>
                  <prop key=".NoSuchRequestHandlingMethodException">error</prop>
                  <prop key=".TypeMismatchException">error</prop>
                  <prop key=".MissingServletRequestParameterException">error</prop>
              </props>
          </property>
      </bean>
  </beans>
  ```

  

  

  