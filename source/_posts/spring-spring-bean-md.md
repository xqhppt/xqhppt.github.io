---
title: Spring Bean装配方式
date: 2017-10-28 19:49:35
tags: Spring
categories: Spring
---

### Spring装配机制
- 在xml中进行显示配置
- 在Java中进行显示配置
- 隐式bean发现机制和自动装配


#### 自动化装配bean
- 组件扫描（component scanning），Spring会自动发现应用上下文中的bean
- 自动装配（autowiring），Spring自动满足bean之间的依赖

**Speak.java**

```
public interface Speak {
    void say();
}
```

**ChineseSpeak.java**

```
@Component("chineseSpeak")
public class ChineseSpeak implements Speak {
    public void say() {
        System.out.println("用中文说");
    }
}
```

**SpeakConfig.java**

```
@Component
@ComponentScan(basePackages = "com.xqh.spring.autowire")
public class SpeakConfig {
}
```


```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpeakConfig.class)
public class SpeakTest {
    @Autowired
    private Speak speak;

    @Test
    public void sayTest(){
        speak.say();
    }
}
```

> @Component注解告诉Spring这是一个组件类并为之创建bean，bean的id为首字母变小写的类名

> @ComponentScan注解启用组件扫描并默认扫描与配置类相同的包

> @Autowired注解会在Spring上下文中在自动装配符合的bean

#### 通过java代码装配bean

- 创建配置类并标记@Configuration注解
- 创建方法标记@Bean注解

**Speak.java**

```
public interface Speak {
    void say();
}
```

**ChineseSpeak.java**

```
public class ChineseSpeak implements Speak {
    public void say(){
        System.out.println("通过java代码装配-用中文说");
    }
}
```

**SpeakConfig.java**

```
@Configuration
public class SpeakConfig {
    @Bean
    public Speak chineseSpeak() {
        return new ChineseSpeak();
    }
}
```


```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpeakConfig.class)
public class SpeakTest {
    @Autowired
    private Speak speak;

    @Test
    public void sayTest(){
        speak.say();
    }
}
```

> @Configuration标记的类表名是一个配置类并且包含在Spring上下文中如何创建bean的细节

> @Bean注解告诉Spring这个方法将返回一个对象并且注册为Spring上下文中的bean

#### 通过xml装配bean

> 创建Spring Xml配置文件
> 在JavaConfig中导入bean配置文件


**Speak.java**

```
public interface Speak {
    void say();
}
```

**ChineseSpeak.java**

```
public class ChineseSpeak implements Speak {
    public void say() {
        System.out.println("通过xml装配bean-用中文说");
    }
}
```

**SpeakConfig.java**

```
@Configuration
@ImportResource("classpath:spring-beans.xml")
public class SpeakConfig {
}
```

**spring-beans.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="chineseSpeak" class="com.xqh.spring.xmlconfig.ChineseSpeak"></bean>
</beans>
```


```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpeakConfig.class)
public class SpeakTest {
    @Autowired
    private Speak speak;

    @Test
    public void sayTest(){
        speak.say();
    }
}
```



















