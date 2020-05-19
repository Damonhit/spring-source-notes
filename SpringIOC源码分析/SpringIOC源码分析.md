# SpringIOC源码分析

## 准备工作

本文会分析Spring的IOC模块的整体流程，分析过程需要使用一个简单的demo工程来启动Spring

Demo工程示例代码

本文源码分析基于Spring5.2.5,所以pom文件中引入5.2.5.RELEASE的依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.5.RELEASE</version>
    </dependency>
</dependencies>
```

然后写一个简单的接口和实现类

```java
public interface IOCService {
    void hello();
}
public class IOCServiceImpl implements IOCService {
    public void hello() {
        System.out.println("Hello,IOC!");
    }
}
```

新建一个application-ioc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="iocService" class="com.zyp.service.impl.IOCServiceImpl"/>
</beans>
```

启动Spring

```java
public class DemoApplication {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext(
                "classpath:/application-ioc.xml");
        IOCService iocService = applicationContext.getBean(IOCService.class);
        iocService.hello();
    }
}
```

**ClassPathXmlApplicationContext：**

背景调查

在文章开始的demo工程中，我选择使用了一个xml文件来配置了接口和实现类之间的关系，然后使用了`ClassPathXmlApplicationContext`这个类来加载这个配置文件。现在我们就先来看一下这个类到底是个什么东东

首先看一下继承关系图（只保留了跟本文相关的，省略了很多其他的继承关系）

![ClassPathXmlApplicationContext](https://github.com/Damonhit/spring-source-notes/raw/master/SpringIOC%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/image/ClassPathXmlApplicationContext.png)

可以看到左下角的就是我们今天的主角`ClassPathXmlApplicationContext`、然后它的旁边是一个同门师兄弟`FileSystemXmlApplicationContext`。看名字就可以知道它们哥俩都是通过加载配置文件来启动Spring的，只不过一个是从程序内加载一个是从系统内加载。

除了这两个还有一个类`AnnotationConfigApplicationContext`比较值得我们关注，这个类是用来处理注解式编程的。

而最上边的`ApplicationContext`则是大名鼎鼎的Spring核心上下文了。
