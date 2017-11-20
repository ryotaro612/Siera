---
layout: post
title:  "Welcome to sona!"
date:   2017-11-14 16:52:07
categories: sona update
tags: sona update
image: /images/pic01.jpg
---
springboot打war包遇到的那些坑

官方链接：http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-create-a-deployable-war-file

公司开发使用了Springboot,但是为什么我们还会打成war包呢，因为我们公司的运维做了钩子，只要有代码往master上合并，它会自动部署到对应tomcat下面，所以我们公司都用war包部署项目，而且tomcat是一个很完整的servlet容器。

关于打war包让我们看看官网怎么说:

The first step in producing a deployable war file is to provide a SpringBootServletInitializer subclass and override its configure method. This makes use of Spring Framework’s Servlet 3.0 support and allows you to configure your application when it’s launched by the servlet container. Typically, you update your application’s main class to extend SpringBootServletInitializer:

@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}
第一步，balaba一大堆，大概就是说让你在你的主程序类上继承SpringBootServletInitializer，并实现configure方法。

The next step is to update your build configuration so that your project produces a war file rather than a jar file. If you’re using Maven and using spring-boot-starter-parent (which configures Maven’s war plugin for you) all you need to do is to modify pom.xml to change the packaging to war:

<packaging>war</packaging>

第二步让你将maven中的pom.xml打包方式改成war包。

The final step in the process is to ensure that the embedded servlet container doesn’t interfere with the servlet container to which the war file will be deployed. To do so, you need to mark the embedded servlet container dependency as provided.

<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
</dependency>

第三部，让你在maven中加入这个jar包，然后你就可以安心的打成war包放到生产环境中了。

其实还有第四步（官方文档没有给),写出war名称，finalName这里。


<build>
     <finalName>websocket</finalName>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
      </plugins>
</build>

生产环境测试(http://host:端口/war包名称），测试后发现没什么问题，美滋滋的下班了，第二天你又对这个项目开发，你发现该项目在你的idea上启动不起来了。启动项目时报了这样的错误。

java.lang.IllegalStateException: ApplicationEventMulticaster not initialized - call 'refresh' before multicasting events via the context: org.springframework.context.annotation.AnnotationConfigApplicationContext@4386f16: startup date [Sat Nov 18 23:58:05 CST 2017]; root of context hierarchy
	at org.springframework.context.support.AbstractApplicationContext.getApplicationEventMulticaster(AbstractApplicationContext.java:414)
	at org.springframework.context.support.ApplicationListenerDetector.postProcessBeforeDestruction(ApplicationListenerDetector.java:97)
	at org.springframework.beans.factory.support.DisposableBeanAdapter.destroy(DisposableBeanAdapter.java:253)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroyBean(DefaultSingletonBeanRegistry.java:578)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingleton(DefaultSingletonBeanRegistry.java:554)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.destroySingleton(DefaultListableBeanFactory.java:961)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingletons(DefaultSingletonBeanRegistry.java:523)
	at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.destroySingletons(FactoryBeanRegistrySupport.java:230)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.destroySingletons(DefaultListableBeanFactory.java:968)
	at org.springframework.context.support.AbstractApplicationContext.destroyBeans(AbstractApplicationContext.java:1030)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:556)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:693)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:360)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:303)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1118)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1107)
	at com.easyto.websocket.WebsocketApplication.main(WebsocketApplication.java:11)
2017-11-18 23:58:05.752 [ERROR] org.springframework.boot.SpringApplication - Application startup failed
org.springframework.beans.factory.BeanDefinitionStoreException: Failed to parse configuration class [com.easyto.websocket.WebsocketApplication]; nested exception is java.lang.IllegalStateException: Failed to introspect annotated methods on class org.springframework.boot.web.support.SpringBootServletInitializer
	at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:181)
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.processConfigBeanDefinitions(ConfigurationClassPostProcessor.java:308)
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(ConfigurationClassPostProcessor.java:228)
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanDefinitionRegistryPostProcessors(PostProcessorRegistrationDelegate.java:270)
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(PostProcessorRegistrationDelegate.java:93)
	at org.springframework.context.support.AbstractApplicationContext.invokeBeanFactoryPostProcessors(AbstractApplicationContext.java:687)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:525)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:693)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:360)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:303)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1118)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1107)
	at com.easyto.websocket.WebsocketApplication.main(WebsocketApplication.java:11)
Caused by: java.lang.IllegalStateException: Failed to introspect annotated methods on class org.springframework.boot.web.support.SpringBootServletInitializer
	at org.springframework.core.type.StandardAnnotationMetadata.getAnnotatedMethods(StandardAnnotationMetadata.java:163)
	at org.springframework.context.annotation.ConfigurationClassParser.retrieveBeanMethodMetadata(ConfigurationClassParser.java:380)
	at org.springframework.context.annotation.ConfigurationClassParser.doProcessConfigurationClass(ConfigurationClassParser.java:314)
	at org.springframework.context.annotation.ConfigurationClassParser.processConfigurationClass(ConfigurationClassParser.java:245)
	at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:198)
	at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:167)
	... 12 common frames omitted
Caused by: java.lang.NoClassDefFoundError: javax/servlet/ServletContext
	at java.lang.Class.getDeclaredMethods0(Native Method)
	at java.lang.Class.privateGetDeclaredMethods(Class.java:2701)
	at java.lang.Class.getDeclaredMethods(Class.java:1975)
	at org.springframework.core.type.StandardAnnotationMetadata.getAnnotatedMethods(StandardAnnotationMetadata.java:152)
	... 17 common frames omitted
Caused by: java.lang.ClassNotFoundException: javax.servlet.ServletContext
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:335)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 21 common frames omitted

你会想这是什么鬼？其实这就是官网文档的坑，解决此问题，需要你把引入的spring-boot-starter-tomcat包的范围注释掉，或者整个jar包注释掉

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <!--<scope>provided</scope>-->
</dependency>

经过实验，你可以把整个包都注释掉。

<!--<dependency>-->
      <!--<groupId>org.springframework.boot</groupId>-->
      <!--<artifactId>spring-boot-starter-tomcat</artifactId>-->
      <!--<scope>provided</scope>-->
<!--</dependency>-->

然后你也可以把你启动类中的这段代码注释掉。

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

然后你会发现世界很美好，打成的包在tomcat下完美运行，项目在idea下使用springboot方式也完美运行。
