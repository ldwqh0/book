# BeanFactory和FactoryBean的区别

## 对比分析

| 维度   | BeanFactory                                    | FactoryBean                                       |
|------|------------------------------------------------|---------------------------------------------------|
| 角色定位 | 容器管理者 (IoC 容器的顶层接口)                            | 特殊 Bean (被容器管理的对象)                                |
| 核心职责 | 管理所有 Bean 的生命周期（创建、注入、销毁）                      | 封装复杂 Bean 的创建逻辑，返回目标对象                            |
| 依赖关系 | 它是 FactoryBean 的管理者                            | 它依赖 BeanFactory 运行，是 BeanFactory 的子民              |
| 典型代表 | ApplicationContext, DefaultListableBeanFactory | SqlSessionFactoryBean (MyBatis), ProxyFactoryBean |

## 深度解析

### BeanFactory：Spring 的“骨架”

它是 Spring IoC 容器的基础接口。你可以把它想象成一个“大仓库”或“工厂总管”。  
职责：它定义了获取 Bean、判断 Bean 是否存在、管理 Bean 作用域（单例/原型）等核心功能。  
关系：我们常用的 ApplicationContext 其实就是 BeanFactory 的增强版（增加了国际化、事件发布等功能）。
一句话总结：它是 Spring 的核心容器，负责统筹全局。

### FactoryBean：复杂的“定制工厂”

它是一个接口，实现这个接口的类会被 Spring 当作一个普通的 Bean 来管理，但它很特殊。  
职责：它不直接作为业务对象使用，而是用来生产其他对象。当某个 Bean 的创建过程非常复杂（比如需要读取配置文件、动态生成代理、连接第三方服务）时，直接写配置会很麻烦，这时就可以实现
FactoryBean 接口，把复杂的创建逻辑封装在 getObject() 方法中。
核心方法：  
getObject()：返回由该工厂创建的目标对象。  
getObjectType()：返回目标对象的类型。  
isSingleton()：返回目标对象是否是单例。  
一句话总结：它是用来简化复杂 Bean 创建的“辅助工具”。

## 如何获取它们？（关键考点）

如何获取BeanFactory() 就是applicationContext

这是开发中最容易踩坑的地方。当你通过 BeanFactory (或 ApplicationContext) 去获取一个 FactoryBean 时，Spring
默认返回的是它生产出来的对象，而不是工厂本身。
假设你有一个 FactoryBean 名字叫 myFactory：

获取目标对象（默认行为）：

```java
// 返回的是 myFactory.getObject() 产生的对象
Object obj = context.getBean("myFactory");
```

获取工厂本身（特殊语法）：  
如果你确实想获取 FactoryBean 这个工厂对象，需要在名字前加 & 前缀。

```java
Object factory = context.getBean("&myFactory"); 
```
