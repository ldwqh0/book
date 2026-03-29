# Spring MVC 执行流程

在了解流程前，先明确几个关键组件的职责：

* DispatcherServlet: 前端控制器，是整个流程的入口和核心调度器。
* HandlerMapping: 处理器映射器，负责根据请求 URL 找到对应的处理器（Controller 方法）。
* HandlerAdapter: 处理器适配器，负责执行找到的处理器。
* HandlerInterceptor: 处理器拦截器，用于在请求处理的各个阶段进行拦截和增强。
* ViewResolver: 视图解析器，负责将逻辑视图名解析为具体的视图对象。
* View: 视图，负责渲染模型数据，生成最终响应（如 HTML）。

1. 请求到达 DispatcherServlet  
   用户的 HTTP 请求首先被 Web 容器（如 Tomcat）接收，然后根据配置转发给 Spring MVC 的核心——DispatcherServlet。
2. 查找处理器 (HandlerMapping)  
   DispatcherServlet 调用 HandlerMapping，根据请求的 URL (/user/list) 和方法 (GET) 来查找能够处理该请求的处理器（即
   @Controller 中的某个方法）。查找成功后，会返回一个 HandlerExecutionChain 对象，其中包含了目标处理器以及相关的拦截器链。
3. 执行拦截器前置处理 (preHandle)  
   DispatcherServlet 会遍历拦截器链，依次执行每个拦截器的 preHandle() 方法。这个方法常用于权限校验、日志记录等。如果任一
   preHandle() 返回 false，则中断后续流程，直接返回响应。
4. 执行处理器 (HandlerAdapter)  
   如果所有拦截器都通过，DispatcherServlet 会通过 HandlerAdapter 来执行目标处理器（Controller 方法）。HandlerAdapter
   负责处理参数绑定（如@RequestParam, @PathVariable）、数据验证等，然后真正调用 Controller 中的业务逻辑。
5. 执行拦截器后置处理 (postHandle)  
   Controller 方法执行完毕后，DispatcherServlet 会再次遍历拦截器链（以相反的顺序），调用每个拦截器的 postHandle()
   方法。这个方法在视图渲染之前执行，可以用来修改模型数据（ModelAndView）。
6. 解析视图 (ViewResolver)  
   Controller 方法通常会返回一个逻辑视图名（如 "user/list"）。DispatcherServlet 会将这个逻辑视图名交给ViewResolver，由它解析成具体的View
   对象（如 /WEB-INF/views/user/list.jsp）。
7. 渲染视图 (View)
   DispatcherServlet 使用解析得到的 View 对象进行视图渲染。它会将 Controller 返回的模型数据（Model）填充到视图中，生成最终的
   HTML 或其他格式的响应内容。
8. 执行拦截器完成处理 (afterCompletion)
   在整个请求处理完成后（无论成功与否），DispatcherServlet 会执行拦截器链的 afterCompletion() 方法，常用于资源清理和记录最终日志。
9. 返回 返回响应
   最终，渲染好的响应内容通过 DispatcherServlet 返回给 Web 容器，再由容器返回给客户端浏览器进行展示。
10. 特殊情况：RESTful 接口  
    对于 RESTful 风格的接口，Controller 方法通常会使用 @ResponseBody 或 @RestController 注解。在这种情况下，Controller
    方法返回的不是一个视图名，而是一个直接的对象（如 User、List 等）。 此时，HandlerAdapter 会利用 HttpMessageConverter
    直接将返回的对象序列化为 JSON 或 XML 格式，并写入到 HTTP 响应体中。因此，步骤6（解析视图）和步骤7（渲染视图）会被跳过。
