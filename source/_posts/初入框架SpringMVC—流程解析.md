title: 初入框架SpringMVC—流程解析
date: 2017/03/22  20:46:25
tags: 路漫漫兮
categories: 小试牛刀
---
#### 什么是MVC
MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。—— 来自于[百度百科](https://baike.baidu.com/item/MVC框架/9241230?fr=aladdin&fromid=85990&fromtitle=MVC)

设计模式：设计模式是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来<!--more-->


#### MVC请求流程图

##### 最典型的MVC模式：JSP+Servlet+JavaBean
Model：JavaBean作为模型，既可以作为数据模型来封装业务数据，又可以作为业务逻辑模型来包含应用的业务操作

View：JSP作为表现层，负责提供页面为用户展示数据，提供相应的表单（Form）来用于用户的请求，并在适当的时候（点击按钮）向控制器发出请求来请求模型进行更新

Controller：Serlvet作为控制器，用来接收用户提交的请求，然后获取请求中的数据，将之转换为业务模型需要的数据模型，然后调用业务模型相应的业务方法进行更新，同时根据业务执行结果来选择要返回的视图

#### 什么是SpringMVC
Spring Web MVC是一种基于Java的实现了Web MVC设计模式的请求驱动类型的轻量级Web框架，即使用了MVC架构模式的思想，将web层进行职责解耦，基于请求驱动指的就是使用请求-响应模型，框架的目的就是帮助我们简化开发，Spring Web MVC也是要简化我们日常Web开发的。

##### SpringMVC优势

###### 清晰的角色划分

- 前端控制器（DispatcherServlet）
- 请求到处理器映射（HandlerMapping）
- 处理器适配器（HandlerAdapter）
- 视图解析器（ViewResolver）
- 处理器或页面控制器（Controller）
- 验证器（ Validator）
- 命令对象（Command 请求参数绑定到的对象就叫命令对象）
- 表单对象（Form Object 提供给表单展示和提交到的对象就叫表单对象）

###### 分工明确
- 而且扩展点相当灵活，可以很容易扩展，虽然几乎不需要；
- 无需继承API直接命令操作
- 由于命令对象就是一个POJO，无需继承框架特定API，可以使用命令对象直接作为业务对象；

###### 与spring无缝衔接 
- 与Spring 其他框架无缝集成，是其它Web框架所不具备的； 

###### 适配任意类作为处理器
- 可适配，通过HandlerAdapter可以支持任意的类作为处理器； 

###### 支持简单定制
- 可定制性，HandlerMapping、ViewResolver等能够非常简单的定制；

###### 功能强大
- 功能强大的数据验证、格式化、绑定机制； 

###### 灵活的单元测试
- 利用Spring提供的Mock对象能够非常简单的进行Web层单元测试； 

###### 更容易的主题与国际化
- 本地化、主题的解析的支持，使我们更容易进行国际化和主题的切换。

###### 强大的jsp标签库
- 强大的JSP标签库，使JSP编写更容易。 
- 还有比如RESTful风格的支持、简单的文件上传、约定大于配置的契约式编程支持、基于注解的零配置支持等等。

#### SpringMVC流程图

##### springmvc运行原理
核心架构的具体流程步骤如下： 

1. 首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制； 

2. DispatcherServlet——>HandlerMapping,HandlerMapping将会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象，通过这种策略模式，很容易添加新的映射策略； 

3. DispatcherServlet——>HandlerAdapter，HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器； 

4. HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView对象（包含模型数据、逻辑视图名）； 

5. ModelAndView的逻辑视图名——> ViewResolver， ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术； 

6. View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术； 

7. 返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。


##### 步骤详解：
- 第一步：用户发送请求（URL链接）到前端控制器（DispatchServlet）

- 第二步：处理器映射器（HandlerMapping）请求查找处理器（Handler）

- 第三步：处理器映射器（HandlerMapping）向前端控制器（DispatchServlet）返回处理器（Handler）

- 第四步：前端控制器（DispatchServlet）调用处理器适配器（HandlerAdatper）执行处理器（Handler）

- 第五步：处理器适配器（HandlerAdatper）执行处理器（Handler）

- 第六步：处理器（Handler）执行完成后给处理器适配器（HandlerAdatper）返回（ModelAndView）

- 第七步：处理器适配器（HandlerAdatper）向前端控制器（DispatchServlet）返回（ModelAndView）

- 第八步：前端控制器（DispatchServlet）请求视图解析器（ViewResolver）进行视图解析

- 第九步：视图解析器（ViewResolver）向前端控制器（DispatchServlet）返回View

- 第十步：前端控制器（DispatchServlet）进行视图渲染，视图渲染指将模型数据（在ModelAndView对象）填充到Request域

- 第十一步：前端控制器（DispatchServlet）向用户返回响应结果

#### 组件
DispatcherServlet：前端控制器

用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。

HandlerMapping：处理器映射器 

HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

Handler：处理器 

Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。 
由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。

HandlAdapter：处理器适配器 

通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

View Resolver：视图解析器 

View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。

View：视图 

springmvc框架提供了很多的View视图类型的支持，包括：jstlView、freemarkerView、pdfView等。我们最常用的视图就是jsp。 
一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

说明：在springmvc的各个组件中，处理器映射器、处理器适配器、视图解析器称为springmvc的三大组件。 
需要用户开发的组件有handler、view

文章中的图片均来源于百度