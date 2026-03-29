# Spring Bean的作用域

在 Spring 框架中，Bean 的作用域（Scope）定义了 Bean 实例的生命周期和可见范围。简单来说，它决定了 Spring 容器在何时创建
Bean、创建多少个实例，以及这个实例能存活多久。
Spring 提供了多种作用域，其中前两种是基础，后几种则特定于 Web 应用环境。

## Spring 的六种标准作用域

1. singleton（单例）
   这是 Spring 容器的默认作用域。
   核心特点：在整个 Spring IoC 容器中，该 Bean 只会有一个共享实例。无论通过依赖注入还是 getBean() 方法获取，返回的都是同一个对象引用。
   生命周期：与容器绑定，容器启动时创建（可通过 @Lazy 改为懒加载），容器关闭时销毁。
   适用场景：适用于无状态的组件，如 Service 层、DAO 层、工具类等。由于是全局共享，需要注意线程安全问题。
2. prototype（原型）
   核心特点：每次向容器请求该 Bean 时（无论是注入还是 getBean()），都会创建一个新的实例。
   生命周期：容器只负责创建和初始化，不管理其销毁。一旦 Bean 被交付给客户端，其生命周期就由客户端自己管理了。
   适用场景：适用于有状态的组件，每个使用者都需要一个独立的实例来维护自己的状态，例如购物车、用户会话数据等。
3. request（请求）
   核心特点：在 Web 应用中，每个 HTTP 请求都会创建一个独立的 Bean 实例。
   生命周期：与 HTTP 请求的生命周期绑定，请求结束时，Bean 实例会被销毁。
   适用场景：适用于封装单次请求的数据，如请求参数、表单数据等。
4. session（会话）
   核心特点：在 Web 应用中，每个 HTTP 会话（Session）都会创建一个独立的 Bean 实例。
   生命周期：与 HTTP Session 的生命周期绑定，会话超时或失效时，Bean 实例会被销毁。
   适用场景：适用于存储用户级别的状态信息，如登录用户信息、购物车等。
5. application（应用）
   核心特点：在 Web 应用中，整个 ServletContext 生命周期内只创建一个 Bean 实例。
   生命周期：与 Web 应用的上下文（ServletContext）绑定，应用启动时创建，关闭时销毁。
   适用场景：适用于在整个 Web 应用中共享的全局配置或数据。
6. websocket（WebSocket）
   核心特点：在 Web 应用中，每个 WebSocket 会话都创建一个独立的实例。
   适用场景：适用于 WebSocket 通信场景下的状态保持。

## 常见陷阱与解决方案

一个经典的陷阱是：在 singleton 作用域的 Bean 中注入 prototype 作用域的 Bean。

* 现象：你会发现注入的 prototype Bean 并没有每次都创建新实例，它表现得像 singleton 一样。
* 原因：singleton Bean 在容器中只会被创建一次，其依赖注入也只在创建时发生一次。因此，它持有的 prototype Bean 的引用也是固定的。
* 解决方案：  
  方法注入：使用 @Lookup 注解，让 Spring 动态重写该方法，每次调用都返回一个新的 prototype Bean。
  Provider 注入：注入一个 ObjectProvider<YourPrototypeBean>，在需要时调用 getObject() 来获取新实例。  
  手动获取：实现 ApplicationContextAware 接口，在需要时通过 applicationContext.getBean(YourPrototypeBean.class) 手动从容器中获取。

## 自定义作用域的实现原理

如果你想在自己的项目中实现类似的功能（比如“租户作用域”或“事务作用域”），Spring 提供了扩展接口。

* 核心接口：org.springframework.beans.factory.config.Scope
* 关键方法：  
  get(String name, ObjectFactory<?> objectFactory)：获取对象（如果不存在则通过工厂创建）。  
  remove(String name)：移除对象。  
  registerDestructionCallback(String name, Runnable callback)：注册销毁回调。  
  注册方式：通过 CustomScopeConfigurer 将自定义的 Scope 注册到 Spring 容器中。

## 各个框架新增的作用域

| 框架              | 新增作用域        | 生命周期范围     | 典型用途                       |
|-----------------|--------------|------------|----------------------------|
| Spring Batch    | Step         | 单次 Step 执行 | 隔离 Reader/Writer 状态，读取文件参数 |
| Spring Batch    | Job          | 单次 Job 执行  | 共享 Job 级别的配置或状态            |
| Spring Web Flow | View         | 单个页面视图     | 表单数据绑定                     |
| Spring Web Flow | Flow         | 整个业务流程     | 向导步骤间的数据传递                 |
| Spring Core     | Thread (自定义) | 单个线程       | 线程隔离的数据（如事务上下文）            |
 

