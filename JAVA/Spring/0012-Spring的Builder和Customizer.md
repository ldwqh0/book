# Spring的Builder和Customizer

## Spring 设计模式总结：Builder vs Customizer

### 1. 核心定义

Builder (生成器)：“从无到有”。用于构建复杂的对象，通过链式调用（Fluent API）替代臃肿的构造函数。它控制对象的创建过程。  
Customizer (定制器)：“锦上添花”。通常是一个回调接口（Lambda），用于对已经存在的 Builder 或对象进行二次加工。它实现了配置与创建的分离。

### 2. 协作关系（Spring Boot 典型模式）

Spring 实例化一个 Builder。  
Spring 自动从容器中寻找所有的 Customizer 实现。  
遍历并执行 customizer.customize(builder)。  
最后调用 builder.build() 生成最终的 Bean。

## Spring 生态中的常用实例列举

### A. Builder 模式实例 (核心骨架)

这些类通常用于在代码中“手动”或“定义式”地创建资源。

| 块                | Builder 类名               | 用途                    |
|------------------|--------------------------|-----------------------|
| Spring Framework | UriComponentsBuilder     | 灵活构建 URL 字符串          |
| Spring Framework | BeanDefinitionBuilder    | 编程式注册 Bean 定义         |
| Spring Web       | RestTemplateBuilder      | 创建并配置 RestTemplate 实例 |
| Spring WebFlux   | WebClient.Builder        | 构建响应式 HTTP 客户端        |
| Spring Security  | HttpSecurity             | 构建安全过滤链（核心配置器）        |
| Spring Security  | User.UserBuilder         | 快速创建用户信息对象            |
| Spring Boot      | SpringApplicationBuilder | 链式启动 Spring 应用        |

### B. Customizer 模式实例 (扩展点)

这些通常是接口，你只需要向容器注入它们的实现，Spring Boot 就会自动调用。

| 模块          | Customizer 接口名                        | 作用对象                                | 定制内容                 |
|-------------|---------------------------------------|-------------------------------------|----------------------|
| Servlet 容器  | WebServerFactoryCustomizer            | ConfigurableServletWebServerFactory | 修改端口、ContextPath、SSL | 
| HTTP 客户端    | RestTemplateCustomizer                | RestTemplate                        | 添加拦截器、消息转换器          | 
| Jackson 序列化 | Jackson2ObjectMapperBuilderCustomizer | Jackson2ObjectMapperBuilder         | 注册自定义序列化器/日期格式       | 
| 缓存管理        | CacheManagerCustomizer                | CacheManager                        | 针对特定缓存库进行微调          | 
| 数据源         | DataSourcePoolMetadataCustomizer      | DataSource                          | 监控或修改连接池元数据          | 
| RabbitMQ    | ContainerCustomizer                   | SimpleMessageListenerContainer      | 调整并发消费者数量等参数         | 

## 小贴士

如果你在写业务代码：你会频繁使用 Builder（例如用 WebClient.builder() 发请求）。
如果你在写通用组件/中间件：
你应该提供一个 Builder 方便别人创建对象。
同时你应该暴露一个 Customizer 接口并自动注入它，允许别人在不修改你源码的情况下，通过 Bean 的方式修改你的 Builder 行为。
