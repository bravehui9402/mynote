Spring MVC主要由DispatcherServlet、处理器映射、处理器(控制器)、视图解析器、视图组成。他的两个核心是两个核心：

**处理器映射**：选择使用哪个控制器来处理请求
**视图解析器**：选择结果应该如何渲染

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210507110607.png)、

Spring MVC框架的众多组件通力配合、各司其职地完成整个流程工作。在整个框架中，Spring MVC通过一个前端控制器接收所有的请求，并将具体工作委托给其他组件进行处理，所以说DispatcherServlet处于核心地位，它负责协调组织不同组件完成请求处理并返回响应结果。

1、客户端发出HTTP请求，Web应用服务器接收此请求。如匹配DispatcherServlet的请求映射路径，则Web容器将该请求转交给DispatcherServlet处理；

2、DispatcherServlet拿到请求之后，根据请求的信息（URL、请求参数、HTTP方法等）及HandlerMapping的配置找到处理请求的处理器（Handler）；

3、当DispatcherServlet找到相应的Handler之后，通过HandlerAdapter对Handler进行封装，再以统一的适配器接口调用Handler。HandlerAdapter可以理解为真正使用Handler来干活的人。

4、在请求信息真正到达调用Handler的处理方法之前的这段时间，Spring MVC还完成了很多工作，它会将请求信息以一定的方式转换并绑定到请求方法的入参，对于入参的对象会进行数据转换、数据格式化以及数据校验等。这些都做完以后，最后才真正调用Handler的处理方法进行相应的业务逻辑处理。

5、处理器完成业务处理之后，将一个ModelAndView对象返回给DispatcherServlet，其中包含了逻辑视图名和模型数据信息。

6、DispatcherServlet通过ViewResolver将逻辑视图名解析为真正的视图对象View，可以是JSP、HTML、XML、PDF、JSON等等，Spring MVC均可灵活配置。

7、得到真正的视图对象之后，DispatcherServlet会根据ModelAndView对象中的模型数据对View进行视图渲染。

8、最终客户端获得响应消息。

```
1、前端控制器DispatcherServlet（不需要程序员开发）。
　　作用：接收请求，响应结果，相当于转发器，中央处理器。有了DispatcherServlet减少了其它组件之间的耦合度。
2、处理器映射器HandlerMapping（不需要程序员开发）。
　　作用：根据请求的url查找Handler。
3、处理器适配器HandlerAdapter（不需要程序员开发）。
　　作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler。
4、处理器Handler（需要程序员开发）。
　　注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler
5、视图解析器ViewResolver（不需要程序员开发）。
　　作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
6、视图View（需要程序员开发jsp）。
　　注意：View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf…）
ps:不需要程序员开发的，需要程序员自己做一下配置即可。
```

