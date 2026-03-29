# 自定义 Spring Boot Starter

自定义 Spring Boot Starter 是将通用功能（如 Redis、MQ、内部工具类）封装成“开箱即用”组件的最佳方式。  
它的核心原理其实就是 Spring SPI（即 spring.factories 或 Spring Boot 2.7+ 引入的 imports 文件）+ 自动配置类。  
为了让你不仅能“用”还能“写”，我将基于最新的 Spring Boot 规范（2.7+/3.x），带你一步步实现一个自定义 Starter。

1. 标准项目结构
   官方推荐将 Starter 拆分为两个模块，以实现配置与依赖的分离：  
   xxx-spring-boot-autoconfigure：核心模块。包含自动配置类、属性绑定类、业务逻辑类。  
   xxx-spring-boot-starter：聚合模块。只包含一个 pom.xml，负责依赖上述 autoconfigure 模块。  
