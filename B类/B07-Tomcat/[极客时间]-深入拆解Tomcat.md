# 目录

![](深入拆解Tomcat-01整体架构.png)

# 开篇词 | Java程序员如何快速成长？

**怎样才能成长为一名高级程序员或者架构师？**

独当一面的能力，这正是高级程序员或者架构师的特质。独当一面的能力，离不开**技术的广度和深度**。

技术的广度体现在你的知识是成体系的，从前端到后端、从应用层面到操作系统、从软件到硬件、从开发、测试、部署到运维…有些领域虽然你不需要挖得很深，但是你必须知道这其中的“门道”。

而技术的深度体现在对于某种技术，你不仅知道怎么用，还知道这项技术如何产生的、它背后的原理是什么，以及它为什么被设计成这样，甚至你还得知道如何去改进它。

但是人的精力是有限的，广度和深度该如何权衡呢？我建议找准一个点先突破深度，而 Tomcat 和 Jetty 就是非常好的选择。

------

模块一：基础（01-04）

# 01 | Web容器学习路径

在开篇词里我提到要成长为一名高级程序员或者架构师，我们需要提高自己知识的广度和深度。你可以先突破深度，再以点带面拓展广度，因此我建议通过深入学习一些优秀的开源系统来达到突破深度的目的。

**Web 容器是什么？**

Servlet 可以简单理解为运行在服务端的 Java 小程序。一个“HTTP 服务器 + Servlet 容器”，我们也叫它们 Web 容器。

其他应用服务器比如 JBoss 和 WebLogic，它们不仅仅有 Servlet 容器的功能，也包含 EJB 容器，是完整的 Java EE 应用服务器。从这个角度看，Tomcat 和 Jetty 算是一个轻量级的应用服务器。

**Web 容器该怎么学？**

- 操作系统基础

  对于 Web 容器来说，操作系统方面你应该掌握它的工作原理，比如什么是进程、什么是内核、什么是内核空间和用户空间、进程间通信的方式、进程和线程的区别、线程同步的方式、什么是虚拟内存、内存分配的过程、什么是 I/O、什么是 I/O 模型、阻塞与非阻塞的区别、同步与异步的区别、网络通信的原理、OSI 七层网络模型以及 TCP/IP、UDP 和 HTTP 协议。

  推荐你读一读《UNIX 环境高级编程》这本经典书籍。

- Java 语言基础

  Java 的基础知识包括 Java 基本语法、面向对象设计的概念（封装、继承、多态、接口、抽象类等）、Java 集合的使用、Java I/O 体系、异常处理、基本的多线程并发编程（包括线程同步、原子类、线程池、并发容器的使用和原理）、Java 网络编程（I/O 模型 BIO、NIO、AIO 的原理和相应的 Java API）、Java 注解以及 Java 反射的原理等。

  此外你还需要了解一些 JVM 的基本知识，比如 JVM 的类加载机制、JVM 内存模型、JVM 内存空间分布、JVM 内存和本地内存的区别以及 JVM GC 的原理等。

  这方面我推荐的经典书籍有《Java 核心技术》、《Java 编程思想》、《Java 并发编程实战》和《深入理解 Java 虚拟机：JVM 高级特性与最佳实践》等。

- Java Web 开发基础

  我的建议是可以从学习 Servlet 和 Servlet 容器开始。
  
  Spring 是用容器来负责创建、组装和销毁这些类的实例，而应用只需要通过配置文件或者注解来告诉 Spring 类与类之间的关系。但是容器的概念不是 Spring 发明的，最开始来源于 Servlet 容器，并且 Servlet 容器也是通过配置文件来加载 Servlet 的。你会发现它们的“元神”是相似的，在 Web 应用的开发中，有一些本质的东西是不变的，而很多“元神”就藏在“老祖宗”那里，藏在 Servlet 容器的设计里。

# 02 | HTTP协议

HTTP 和 HTML 有什么区别？

HTTP 是通信的方式，HTML 才是通信的目的，就好比 HTTP 是信封，信封里面的信（HTML）才是内容；但是没有信封，信也没办法寄出去。

**HTTP 的本质**

假如浏览器需要从远程 HTTP 服务器获取一个 HTML 文本，在这个过程中，浏览器实际上要做两件事情：与服务器建立 Socket 连接、生成**请求数据**并通过 Socket 发送出去。

请求数据里面应该包含这些信息：你是想获取内容还是提交内容？你想要哪个内容？

这些信息以一种什么样的格式放到请求里去呢？这就是 HTTP 协议要解决的问题。

HTTP 协议的本质就是一种浏览器与服务器之间约定好的通信格式。

**HTTP 工作原理**

以一次实际调用为例。

<img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091112.jpg" style="zoom: 67%;" />

Tomcat 服务器接受连接、解析请求数据、处理请求和发送响应。实际情况可能会有成千上万的浏览器同时请求同一个 HTTP 服务器，为了提高服务能力和并发度，Tomcat 使用了多线程技术。

**HTTP 请求响应实例**

那 HTTP 协议的数据包具体长什么样呢？

![](https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091117.png)

HTTP 请求数据由三部分组成，分别是**请求行、请求报头、请求正文**。

当这个 HTTP 请求数据到达 Tomcat 后，Tomcat 会把 HTTP 请求数据字节流解析成一个 Request 对象，这个 Request 对象封装了 HTTP 所有的请求信息。接着 Tomcat 把这个 Request 对象交给 Web 应用去处理，处理完后得到一个 Response 对象，Tomcat 会把这个 Response 对象转成 HTTP 格式的响应数据并发送给浏览器。

HTTP 的响应也是由三部分组成，分别是**状态行、响应报头、报文主体**。

![](https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091124.png)

**Cookie 和 Session**

HTTP 协议有个特点是无状态，请求与请求之间是没有关系的。那 Tomcat 是怎么鉴别多个请求是来自同一个用户的呢？

- Cookie 技术

  Cookie 是 HTTP 报文的一个请求头，Web 应用可以将用户的标识信息或者其他一些信息（用户名等）存储在 Cookie 中。用户经过验证之后，每次 HTTP 请求报文中都包含 Cookie，这样服务器读取这个 Cookie 请求头就知道用户是谁了。

  Cookie 本质上就是一份存储在用户本地的文件，里面包含了每次请求中都需要传递的信息。

- Session 技术

  由于 Cookie 以明文的方式存储在本地，而 Cookie 中往往带有用户信息，这样就造成了非常大的安全隐患。

  Session 的出现解决了这个问题，Session 可以理解为服务器端开辟的存储空间，里面保存了用户的状态，用户信息以 Session 的形式存储在服务端。当用户请求到来时，服务端可以把用户的请求和用户的 Session 对应起来。

  那么 Session 是怎么和请求对应起来的呢？

  具体工作过程是这样的：服务器在创建 Session 的同时，会为该 Session 生成唯一的 Session ID，当浏览器再次发送请求的时候，会将这个 Session ID 带上，服务器接受到请求之后就会依据 Session ID 找到相应的 Session，找到 Session 后，就可以在 Session 中获取或者添加内容了。而这些内容只会保存在服务器中，发到客户端的只有 Session ID，这样相对安全，也节省了网络流量，因为不需要在 Cookie 中存储大量用户信息。

- Session 创建与存储

  那么 Session 在何时何地创建呢？
  
  不同语言实现的应用程序有不同的创建 Session 的方法。在 Java 中，是 Web 应用程序在调用 HttpServletRequest 的 getSession 方法时，由 Web 容器创建的。
  
  Tomcat 的 Session 管理器提供了多种持久化方案来存储 Session，通常会采用高性能的存储方式，比如 Redis，并且通过集群部署的方式，防止单点故障，从而提升高可用。同时，Session 有过期时间，因此 Tomcat 会开启后台线程定期的轮询，如果 Session 过期了就将 Session 失效。

# 03 | Servlet规范和Servlet容器

通过上一讲我们知道，浏览器发给服务端的是一个 HTTP 格式的请求，HTTP 服务器收到这个请求后，需要调用服务端程序来处理。那么问题来了，HTTP 服务器怎么知道要调用哪个业务类呢？

最直接的做法是在 HTTP 服务器代码里写一大堆 if else 逻辑判断。但这样做明显有问题，因为 HTTP 服务器的代码跟业务逻辑耦合在一起了，如果新加一个业务方法还要改 HTTP 服务器的代码。

**Servlet 规范**

面向接口编程是解决耦合问题的法宝，各种业务类都必须实现这个接口，这个接口就叫 Servlet 接口。Servlet 容器用来加载和管理业务类，当接收到一个 HTTP 请求时，Servlet 容器会将请求转发到具体的 Servlet，如果这个 Servlet 还没创建，就加载并实例化这个 Servlet，然后调用这个 Servlet 的接口方法。

下面我们通过一张图来加深理解。

![](https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091131.jpg)

Servlet 接口和 Servlet 容器这一整套规范叫作 Servlet 规范。

Tomcat 和 Jetty 都按照 Servlet 规范的要求实现了 Servlet 容器，同时它们也具有 HTTP 服务器的功能。作为 Java 程序员，如果我们要实现新的业务功能，只需要实现一个 Servlet，并把它注册到 Tomcat（Servlet 容器）中，剩下的事情就由 Tomcat 帮我们处理了。

**Servlet 接口**

Servlet 接口定义了下面五个方法：

```java
public interface Servlet {
    void init(ServletConfig config) throws ServletException;
    
    ServletConfig getServletConfig();
    
    void service(ServletRequest req, ServletResponse res）throws ServletException, IOException;
    
    String getServletInfo();
    
    void destroy();
}
```

- service()

   service 方法实现具体业务处理逻辑。ServletRequest 用来封装请求信息，ServletResponse 用来封装响应信息。

  HTTP 协议中的请求和响应就是对应了 HttpServletRequest 和 HttpServletResponse 这两个类。可以通过 HttpServletRequest 来获取所有请求相关的信息，包括请求路径、Cookie、HTTP 头、请求参数等，还可以通过 HttpServletRequest 来创建和获取 Session。

  HttpServletResponse 是用来封装 HTTP 响应的。

- init()

  Servlet 容器在加载 Servlet 类的时候会调用 init 方法。比如 Spring MVC 中的 DispatcherServlet，就是在 init 方法里创建了自己的 Spring 容器。

- destroy()

  Servlet 容器在卸载 Servlet 类的时候会调用 destroy 方法，释放一些资源。

- getServletConfig()

  ServletConfig 的作用就是封装 Servlet 的初始化参数。你可以在 web.xml 给 Servlet 配置参数，并在程序里通过 getServletConfig 方法拿到这些参数。

Servlet 规范提供了 GenericServlet 抽象类，我们可以通过扩展它来实现 Servlet。虽然 Servlet 规范并不在乎通信协议是什么，但是大多数的 Servlet 都是在 HTTP 环境中处理的，因此 Servet 规范还提供了 HttpServlet 来继承 GenericServlet，并且加入了 HTTP 特性。这样我们通过继承 HttpServlet 类来实现自己的 Servlet，只需要重写两个方法：doGet 和 doPost。

**Servlet 容器**

为了解耦，HTTP 服务器不直接调用 Servlet，而是把请求交给 Servlet 容器来处理，那 Servlet 容器又是怎么工作的呢？

- 工作流程

  当客户请求某个资源时，HTTP 服务器会用一个 ServletRequest 对象把客户的请求信息封装起来，然后调用 Servlet 容器的 service 方法，Servlet 容器拿到请求后，根据请求的 URL 和 Servlet 的映射关系，找到相应的 Servlet，如果 Servlet 还没有被加载，就用反射机制创建这个 Servlet，并调用 Servlet 的 init 方法来完成初始化，接着调用 Servlet 的 service 方法来处理请求，把 ServletResponse 对象返回给 HTTP 服务器，HTTP 服务器会把响应发送给客户端。

  <img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091138.jpg" style="zoom: 67%;" />

- Web应用

  Servlet 容器会实例化和调用 Servlet，那 Servlet 是怎么注册到 Servlet 容器中的呢？

  根据 Servlet 规范，Web 应用程序有一定的目录结构，在这个目录下分别放置了 Servlet 的类文件、配置文件以及静态资源，Servlet 容器通过读取配置文件，就能找到并加载 Servlet。

  Web 应用的目录结构大概是下面这样的：

  ```text
  | -  MyWebApp
        | -  WEB-INF/web.xml        -- 配置文件，用来配置 Servlet 等
        | -  WEB-INF/lib/           -- 存放 Web 应用所需各种 JAR 包
        | -  WEB-INF/classes/       -- 存放你的应用类，比如 Servlet 类
        | -  META-INF/              -- 目录存放工程的一些信息
  ```

  Servlet 规范里定义了 **ServletContext** 这个接口来对应一个 Web 应用。

  Servlet 容器在启动时会加载 Web 应用，并为每个 Web 应用创建唯一的 ServletContext 对象。一个 Web 应用可能有多个 Servlet，这些 Servlet 可以通过全局的 ServletContext 来共享数据，这些数据包括 Web 应用的初始化参数、Web 应用目录下的文件资源等。由于 ServletContext 持有所有 Servlet 实例，你还可以通过它来实现 Servlet 请求的转发。

- 扩展机制

  引入了 Servlet 规范后，你不需要关心 Socket 网络通信、不需要关心 HTTP 协议，也不需要关心你的业务类是如何被实例化和调用的，因为这些都被 Servlet 规范标准化了，你只要关心怎么实现的你的业务逻辑。

  考虑到可扩展性。Servlet 规范提供了两种扩展机制：**Filter** 和 **Listener**。

  **Filter**是过滤器，这个接口允许你对请求和响应做一些统一的定制化处理，比如你可以根据请求的频率来限制访问，或者根据国家地区的不同来修改响应内容。过滤器的工作原理是这样的：Web 应用部署完成后，Servlet 容器需要实例化 Filter 并把 Filter 链接成一个 FilterChain。当请求进来时，获取第一个 Filter 并调用 doFilter 方法，doFilter 方法负责调用这个 FilterChain 中的下一个 Filter。

  **Listener**是监听器，这是另一种扩展机制。当 Web 应用在 Servlet 容器中运行时，Servlet 容器内部会不断的发生各种事件，如 Web 应用的启动和停止、用户请求到达等。 Servlet 容器提供了一些默认的监听器来监听这些事件，当事件发生时，Servlet 容器会负责调用监听器的方法。当然，你可以定义自己的监听器去监听你感兴趣的事件，将监听器配置在 web.xml 中。比如 Spring 就实现了自己的监听器，来监听 ServletContext 的启动事件，目的是当 Servlet 容器启动时，创建并初始化全局的 Spring 容器。

到这里相信你对 Servlet 容器的工作原理有了深入的了解。

**课后思考**

Servlet 容器与 Spring 容器有什么关系？

# 04 | 实战：纯手工打造Servlet和Tomcat

**用配置文件部署 Servlet**

**用注解的方式部署 Servlet**

参考：https://github.com/feifa168/mytomcat

------

模块二：整体架构（05-13）

# 05 | Tomcat系统架构（上）： 连接器是如何设计的？

在面试时我们可能经常被问到：你做的 XX 项目的架构是如何设计的，请讲一下实现的思路。对于面试官来说，可以通过你对复杂系统设计的理解，了解你的技术水平以及处理复杂问题的思路。

**Tomcat 总体架构**

我们知道如果要设计一个系统，首先是要了解需求。通过专栏前面的文章，我们已经了解了 Tomcat 要实现 2 个核心功能：

- 处理 Socket 连接，负责网络字节流与 Request 和 Response 对象的转化。
- 加载和管理 Servlet，以及具体处理 Request 请求。

因此 Tomcat 设计了两个核心组件连接器（Connector）和容器（Container）来分别做这两件事情。连接器负责对外交流，容器负责内部处理。

Tomcat 支持的 I/O 模型有：

- 多路复用 IO：采用 Java NIO 类库实现。
- NIO2：异步 I/O，采用 JDK 7 最新的 NIO2 类库实现。
- APR：采用 Apache 可移植运行库实现，是 C/C++ 编写的本地库。

Tomcat 支持的应用层协议有：

- HTTP/1.1：这是大部分 Web 应用采用的访问协议。
- AJP：用于和 Web 服务器集成（如 Apache）。
- HTTP/2：HTTP 2.0 大幅度的提升了 Web 性能。

Tomcat 为了实现支持多种 I/O 模型和应用层协议，一个容器可能对接多个连接器。但是单独的连接器或者容器都不能对外提供服务，需要把它们组装起来才能工作，组装后这个整体叫作 Service 组件。

这里请你注意，Service 本身没有做什么重要的事情，只是在连接器和容器外面多包了一层，把它们组装在一起。Tomcat 内可能有多个 Service，通过在 Tomcat 中配置多个 Service，可以实现通过不同的端口号来访问同一台机器上部署的不同应用。

<img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091145.jpg" style="zoom: 50%;" />

从图上你可以看到，最顶层是 Server，这里的 Server 指的就是一个 Tomcat 实例。一个 Server 中有一个或者多个 Service，一个 Service 中有多个连接器和一个容器。连接器与容器之间通过标准的 ServletRequest 和 ServletResponse 通信。

**连接器**

连接器对 Servlet 容器屏蔽了协议及 I/O 模型等的区别，无论是 HTTP 还是 AJP，在容器中获取到的都是一个标准的 ServletRequest 对象。

我们可以把连接器的功能需求进一步细化：

- 监听网络端口。
- 接受网络连接请求。
- 读取请求网络字节流。
- 根据具体应用层协议（HTTP/AJP）解析字节流，生成统一的 Tomcat Request 对象。
- 将 Tomcat Request 对象转成标准的 ServletRequest。
- 调用 Servlet 容器，得到 ServletResponse。
- 将 ServletResponse 转成 Tomcat Response 对象。
- 将 Tomcat Response 转成网络字节流。
- 将响应字节流写回给浏览器。

我们要考虑的下一个问题是，连接器应该有哪些子模块？优秀的模块化设计应该考虑**高内聚、低耦合**。

- **高内聚**是指相关度比较高的功能要尽可能集中，不要分散。
- **低耦合**是指两个相关的模块要尽可能减少依赖的部分和降低依赖的程度，不要让两个模块产生强依赖。

因此 Tomcat 的设计者设计了 3 个组件来实现这 3 个功能，分别是 EndPoint、Processor 和 Adapter。

EndPoint 负责提供字节流给 Processor，Processor 负责提供 Tomcat Request 对象给 Adapter，Adapter 负责提供 ServletRequest 对象给容器。

<img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091152.jpg" style="zoom:50%;" />

Tomcat 的设计者将网络通信和应用层协议解析放在一起考虑，设计了一个叫 ProtocolHandler 的接口来封装这两种变化点。各种协议和通信模型的组合有相应的具体实现类。比如：Http11NioProtocol 和 AjpNioProtocol。

继承关系如下图所示：

<img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091158.jpg" style="zoom:50%;" />

通过上图，你可以看到每一种 I/O 模型和协议的组合都有相应的具体实现类，我们在使用时可以自由选择。

**ProtocolHandler 组件**

连接器用 ProtocolHandler 来处理网络连接和应用层协议，包含了 2 个重要部件：EndPoint 和 Processor。

- EndPoint

  EndPoint 是一个接口，是对传输层的抽象，对应的抽象实现类是 AbstractEndpoint，而 AbstractEndpoint 的具体子类，比如在 NioEndpoint 和 Nio2Endpoint 中，有两个重要的子组件：Acceptor 和 SocketProcessor。

  其中 Acceptor 用于监听 Socket 连接请求。SocketProcessor 用于处理接收到的 Socket 请求，它实现 Runnable 接口，在 Run 方法里调用协议处理组件 Processor 进行处理。为了提高处理能力，SocketProcessor 被提交到线程池来执行。而这个线程池叫作执行器（Executor)。

- Processor

  Processor 是对应用层协议的抽象。如果说 EndPoint 是用来实现 TCP/IP 协议的，那么 Processor 用来实现 HTTP 协议，Processor 接收来自 EndPoint 的 Socket，读取字节流解析成 Tomcat Request 和 Response 对象，并通过 Adapter 将其提交到容器处理。

  Processor 是一个接口，定义了请求的处理等方法。它的抽象实现类 AbstractProcessor 对一些协议共有的属性进行封装，没有对方法进行实现。具体的实现有 AJPProcessor、HTTP11Processor 等，这些具体实现类实现了特定协议的解析方法和请求处理方式。

我们再来看看连接器的组件图：

<img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091202.jpg" style="zoom:50%;" />

从图中我们看到，EndPoint 接收到 Socket 连接后，生成一个 SocketProcessor 任务提交到线程池去处理，SocketProcessor 的 Run 方法会调用 Processor 组件去解析应用层协议，Processor 通过解析生成 Request 对象后，会调用 Adapter 的 Service 方法。

**Adapter 组件**

由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat 定义了自己的 Request 类来“存放”这些请求信息。ProtocolHandler 接口负责解析请求并生成 Tomcat Request 类。但是这个 Request 对象不是标准的 ServletRequest，也就意味着，不能用 Tomcat Request 作为参数来调用容器。Tomcat 设计者的解决方案是引入 CoyoteAdapter，这是适配器模式的经典运用，连接器调用 CoyoteAdapter 的 Sevice 方法，传入的是 Tomcat Request 对象，CoyoteAdapter 负责将 Tomcat Request 转成 ServletRequest，再调用容器的 Service 方法。

**课后思考**

回忆一下你在工作中曾经独立设计过的系统，或者你碰到过的设计类面试题，结合今天专栏的内容，你有没有一些新的思路？

# 06 | Tomcat系统架构（下）：聊聊多层容器的设计

容器，顾名思义就是用来装载东西的器具，在 Tomcat 里，容器就是用来装载 Servlet 的。那 Tomcat 的 Servlet 容器是如何设计的呢？

**容器的层次结构**

Tomcat 设计了 4 种容器，分别是 Engine、Host、Context 和 Wrapper。这 4 种容器是父子关系，如下图所示。

<img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091207.jpg" style="zoom:50%;" />

为什么要设计这么多层次的容器呢？

Context 表示一个 Web 应用程序；Wrapper 表示一个 Servlet；Host 代表的是一个虚拟主机，或者说一个站点；Engine 表示引擎，用来管理多个虚拟站点；一个 Service 最多只能有一个 Engine。

你可以再通过 Tomcat 的 server.xml 配置文件来加深对 Tomcat 容器的理解。

```xml
<server>            <!-- 顶层组件，可以包括多个service -->
  <service>         <!-- 顶层组件，可包含一个engine，多个连接器 -->
    <connector>     <!-- 连接器组件，代表通信接口 -->
    </connector>    
    <engine>        <!-- 容器组件，一个Engine组件处理Service中的所有请求，包含多个Host -->
      <host>        <!-- 容器组件，处理特定的Host下客户请求，可包含多个Context -->
         <context>  <!-- 容器组件，为特定的Web应用处理所有的客户请求 -->
         </context>
      </host>
    </engine>
  </service>
</server>
```

如果想要配置多个Host，可参考下面代码。

```xml
<server port=“8005” shutdown=“SHUTDOWN”>
  <service name=“Catalina”>
    <engine defaulthost=“localhost” name=“Catalina”>
      <host appbase=“webapps” autodeploy=“true” name=“localhost” unpackwars=“true”></host>
      <host appbase=“webapps1” autodeploy=“true” name=“www.domain1.com” unpackwars=“true”></host>
      <host appbase=“webapps2” autodeploy=“true” name=“www.domain2.com” unpackwars=“true”></host>
      <host appbase=“webapps3” autodeploy=“true” name=“www.domain3.com” unpackwars=“true”></host>
    </engine>
  </service>
</server>
```

那么，Tomcat 是怎么管理这些容器的呢？

Tomcat 是用组合模式来管理这些容器的。具体实现方法是，所有容器组件都实现了 Container 接口。

```java
public interface Container extends Lifecycle {
    public void setName(String name);
    public Container getParent();
    public void setParent(Container container);
    public void addChild(Container child);
    public void removeChild(Container child);
    public Container findChild(String name);
}
```

LifeCycle 接口用来统一管理各组件的生命周期。

**请求定位 Servlet 的过程**

设计了这么多层次的容器，Tomcat 是怎么确定请求是由哪个 Wrapper 容器里的 Servlet 来处理的呢？

Tomcat 是用 Mapper 组件来完成这个任务的。Mapper 组件的功能就是将用户请求的 URL 定位到一个 Servlet，它的工作原理是：Mapper 组件里保存了 Web 应用的配置信息，其实就是**容器组件与访问路径的映射关系**，比如 Host 容器里配置的域名、Context 容器里的 Web 应用路径，以及 Wrapper 容器里 Servlet 映射的路径，你可以想象这些配置信息就是一个多层次的 Map。

当一个请求到来时，Mapper 组件通过解析请求 URL 里的域名和路径，再到自己保存的 Map 里去查找，就能定位到一个 Servlet。

接下来我通过一个例子来解释这个定位的过程。

假如有一个网购系统，有面向网站管理人员的后台管理系统，还有面向终端客户的在线购物系统。这两个系统跑在同一个 Tomcat 上，为了隔离它们的访问域名，配置了两个虚拟域名：`manage.shopping.com`和`user.shopping.com`。网站管理人员可以管理用户和商品；终端客户可以搜索商品和下订单，搜索功能和订单管理也是两个独立的 Web 应用。如下图所示。

<img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091213.jpg" style="zoom: 50%;" />

假如有用户访问一个 URL，比如图中的`http://user.shopping.com:8080/order/buy`，Tomcat 如何将这个 URL 定位到一个 Servlet 呢？

- 首先，根据协议和端口号选定 Service 和 Engine。

  我们知道 Tomcat 的每个连接器都监听不同的端口，比如 Tomcat 默认的 HTTP 连接器监听 8080 端口、默认的 AJP 连接器监听 8009 端口。一个连接器是属于一个 Service 组件的，这样 Service 组件就确定了。我们还知道一个 Service 组件里除了有多个连接器，还有一个容器组件，具体来说就是一个 Engine 容器，因此 Service 确定了也就意味着 Engine 也确定了。

- 然后，根据域名选定 Host。

  Service 和 Engine 确定后，Mapper 组件通过 URL 中的域名去查找相应的 Host 容器，比如例子中的 URL 访问的域名是`user.shopping.com`，因此 Mapper 会找到 Host2 这个容器。

- 之后，根据 URL 路径找到 Context 组件。

  Host 确定以后，Mapper 根据 URL 的路径来匹配相应的 Web 应用的路径，比如例子中访问的是 /order，因此找到了 Context4 这个 Context 容器。

- 最后，根据 URL 路径找到 Wrapper（Servlet）。

  Context 确定后，Mapper 再根据 web.xml 中配置的 Servlet 映射路径来找到具体的 Wrapper 和 Servlet。

并不是说只有 Servlet 才会去处理请求，实际上这个查找路径上的父子容器都会对请求做一些处理。我在上一期说过，连接器中的 Adapter 会调用容器的 Service 方法来执行 Servlet，最先拿到请求的是 Engine 容器，Engine 容器对请求做一些处理后，会把请求传给自己子容器 Host 继续处理，依次类推，最后这个请求会传给 Wrapper 容器，Wrapper 会调用最终的 Servlet 来处理。那么这个调用过程具体是怎么实现的呢？答案是使用 Pipeline-Valve 管道。

**Pipeline-Valve 管道**

Pipeline-Valve 是责任链模式，责任链模式是指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完之后将再调用下一个处理者继续处理。

Valve 表示一个处理点，比如权限认证和记录日志。

```java
public interface Valve {
  public Valve getNext();
  public void setNext(Valve valve);
  public void invoke(Request request, Response response)
}
```

invoke 方法就是来处理请求的。

```java
public interface Pipeline extends Contained {
  public void addValve(Valve valve);
  public Valve getBasic();
  public void setBasic(Valve valve);
  public Valve getFirst();
}
```

Pipeline 中维护了 Valve 链表，Valve 可以插入到 Pipeline 中，对请求做某些处理。

每一个容器都有一个 Pipeline 对象，只要触发这个 Pipeline 的第一个 Valve，这个容器里 Pipeline 中的 Valve 就都会被调用到。

但是，不同容器的 Pipeline 是怎么链式触发的呢？

 Pipeline 中还有个 getBasic 方法。这个 BasicValve 处于 Valve 链表的末端，负责调用下层容器的 Pipeline 里的第一个 Valve。

<img src="https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091220.jpg" style="zoom:50%;" />

整个调用过程由连接器中的 Adapter 触发的，它会调用 Engine 的第一个 Valve：

```java
// Calling the container
connector.getService()
    .getContainer()
    .getPipeline()
    .getFirst()
    .invoke(request, response);
```

Wrapper 容器的最后一个 Valve 会创建一个 Filter 链，并调用 doFilter() 方法，最终会调到 Servlet 的 service 方法。

**Valve 与 Filter**

Valve 和 Filter 有什么区别吗？

- Valve 是 Tomcat 的私有机制，与 Tomcat 的基础架构 /API 是紧耦合的。Servlet API 是公有的标准，所有的 Web 容器包括 Jetty 都支持 Filter 机制。
- Valve 工作在 Web 容器级别，拦截所有应用的请求；而 Servlet Filter 工作在应用级别，只能拦截某个 Web 应用的所有请求。如果想做整个 Web 容器的拦截器，必须通过 Valve 来实现。

**课后思考**

Tomcat 内的 Context 组件跟 Servlet 规范中的 ServletContext 接口有什么区别？跟 Spring 中的 ApplicationContext 又有什么关系？

答：1）Servlet规范中ServletContext表示web应用的上下文环境，而web应用对应tomcat的概念是Context，所以从设计上，ServletContext自然会成为tomcat的Context具体实现的一个成员变量。

2）tomcat内部实现也是这样完成的，ServletContext对应tomcat实现是org.apache.catalina.core.ApplicationContext，Context容器对应tomcat实现是org.apache.catalina.core.StandardContext。ApplicationContext是StandardContext的一个成员变量。

3）Spring的ApplicationContext之前已经介绍过，tomcat启动过程中ContextLoaderListener会监听到容器初始化事件，它的contextInitialized方法中，Spring会初始化全局的Spring根容器ApplicationContext，初始化完毕后，Spring将其存储到ServletContext中。

总而言之，Servlet规范中ServletContext是tomcat的Context实现的一个成员变量，而Spring的ApplicationContext是Servlet规范中ServletContext的一个属性。

# 07 | Tomcat如何实现一键式启停？

我们通过一张简化的类图来回顾一下 Tomcat 的组件。

![](https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091228.png)

图中的虚线表示一个请求在 Tomcat 中流转的过程。

在我们实际的工作中，如果你需要设计一个比较大的系统或者框架时，你同样也需要考虑这几个问题：如何统一管理组件的创建、初始化、启动、停止和销毁？如何做到代码逻辑清晰？如何方便地添加或者删除组件？如何做到组件启动和停止不遗漏、不重复？

先来看看组件之间的关系：组件有大有小，大组件管理小组件；组件有外有内，外层组件控制内层组件；

因此在创建组件时应该遵循一定的顺序：先创建子组件，再创建父组件，子组件需要被“注入”到父组件中；先创建内层组件，再创建外层组件，内层组建需要被“注入”到外层组件。

因此，最直观的做法就是将图上所有的组件按照先小后大、先内后外的顺序创建出来，然后组装在一起。不知道你注意到没有，这个思路其实很有问题！因为这样不仅会造成代码逻辑混乱和组件遗漏，而且也不利于后期的功能扩展。

为了解决这个问题，我们希望找到一种通用的、统一的方法来管理组件的生命周期，就像汽车“一键启动”那样的效果。

**一键式启停：Lifecycle 接口**

设计就是要找到系统的变化点和不变点。这里的不变点就是每个组件都要经历创建、初始化、启动这几个过程，这些状态以及状态的转化是不变的。而变化点是每个具体组件的初始化方法，也就是启动方法是不一样的。

因此，我们把不变点抽象出来成为一个接口，这个接口跟生命周期有关，叫作 LifeCycle。

在父组件的 init() 方法里需要创建子组件并调用子组件的 init() 方法。同样，在父组件的 start() 方法里也需要调用子组件的 start() 方法，因此调用者可以无差别的调用各组件的 init() 方法和 start() 方法，这就是**组合模式**的使用，并且只要调用最顶层组件，也就是 Server 组件的 init() 和 start() 方法，整个 Tomcat 就被启动起来了。

下面是 LifeCycle 接口的定义。

```java
public interface Lifecycle {
    void init();
    void start();
    void stop();
    void destroy();
}
```

**可扩展性：Lifesycle 事件**

我们再来考虑另一个问题，那就是系统的可扩展性。因为各个组件 init() 和 start() 方法的具体实现是复杂多变的，比如在 Host 容器的启动方法里需要扫描 webapps 目录下的 Web 应用，创建相应的 Context 容器，如果将来需要增加新的逻辑，直接修改 start() 方法？这样会违反开闭原则，那如何解决这个问题呢？

我们注意到，组件的 init() 和 start() 调用是由它的父组件的状态变化触发的，上层组件的初始化会触发子组件的初始化，上层组件的启动会触发子组件的启动，因此我们把组件的生命周期定义成一个个状态，把状态的转变看作是一个事件。而事件是有监听器的，在监听器里可以实现一些逻辑，并且监听器也可以方便的添加和删除，这就是典型的**观察者模式**。

具体来说就是在 Lifecycle 接口里加入两个方法：添加监听器和删除监听器。除此之外，我们还需要定义一个 Enum 来表示组件有哪些状态，以及处在什么状态会触发什么样的事件。因此 Lifecycle 接口和 LifecycleState 就定义成了下面这样。

![](https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091234.png)

一旦组件到达相应的状态就触发相应的事件，比如 NEW 状态表示组件刚刚被实例化；而当 init() 方法被调用时，状态就变成 INITIALIZING 状态，这个时候，就会触发 BEFORE_INIT_EVENT 事件，如果有监听器在监听这个事件，它的方法就会被调用。

**重用性：LifecycleBase 抽象基类**

有了接口，我们就要用类去实现接口。为了避免重复代码，我们用基类来实现共同的逻辑。基类中会定义一些抽象方法，留给各个子类去实现。

Tomcat 定义一个基类 LifecycleBase 来实现 Lifecycle 接口，把一些公共的逻辑放到基类中去，比如生命状态的转变与维护、生命事件的触发以及监听器的添加和删除等，而子类就负责实现自己的初始化、启动和停止等方法。

```java
public abstract class LifecycleBase implements Lifecycle {
    abstract void initInternal();
    abstract void startInternal();
    abstract void stopInternal();
    abstract void destroyInternal();
}
```

上面就是典型的**模板设计模式**。

我们还是看一看代码，可以帮你加深理解，下面是 LifecycleBase 的 init() 方法实现。

```java
@Override
public final synchronized void init() throws LifecycleException {
    //1. 状态检查，比如当前状态必须是 NEW 然后才能进行初始化
    if (!state.equals(LifecycleState.NEW)) {
        invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
    }
 
    try {
        //2. 触发 INITIALIZING 事件的监听器，调用监听器的业务方法
        setStateInternal(LifecycleState.INITIALIZING, null, false);
        
        //3. 调用具体子类的初始化方法
        initInternal();
        
        //4. 触发 INITIALIZED 事件的监听器，调用监听器的业务方法
        setStateInternal(LifecycleState.INITIALIZED, null, false);
    } catch (Throwable t) {
      ...
    }
}
```

LifeCycleBase 负责触发事件，并调用监听器的方法，那是什么时候、谁把监听器注册进来的呢？

Tomcat 自定义了一些监听器，这些监听器是父组件在创建子组件的过程中注册到子组件的。比如 MemoryLeakTrackingListener 监听器，用来检测 Context 容器中的内存泄漏，这个监听器是 Host 容器在创建 Context 容器时注册到 Context 中的。

我们还可以在 server.xml 中定义自己的监听器，Tomcat 在启动时会解析 server.xml，创建监听器并注册到容器组件。

**生命周期管理总体类图**

![](https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091239.png)

这里请你注意，图中的 StandardServer、StandardService 等是 Server 和 Service 组件的具体实现类，它们都继承了 LifeCycleBase。

StandardEngine、StandardHost、StandardContext 和 StandardWrapper 是相应容器组件的具体实现类，因为它们都是容器，所以继承了 ContainerBase 抽象基类，而 ContainerBase 实现了 Container 接口，也继承了 LifeCycleBase 类，它们的生命周期管理接口和功能接口是分开的，这也符合设计中**接口分离的原则**。

**课后思考**

从文中最后的类图上你会看到所有的容器组件都扩展了 ContainerBase，跟 LifeCycleBase 一样，ContainerBase 也是一个骨架抽象类，请你思考一下，各容器组件有哪些“共同的逻辑”需要 ContainerBase 由来实现呢？

答：ContainerBase提供了针对Container接口的通用实现，所以最重要的职责包含两个:
1) 维护容器通用的状态数据；
2) 提供管理状态数据的通用方法；

# 08 | Tomcat的“高层们”都负责做什么？

我们通过 Tomcat 的 /bin 目录下的脚本 startup.sh 来启动 Tomcat，那你是否知道我们执行了这个脚本后发生了什么呢？你可以通过下面这张流程图来了解一下。

![](https://gitee.com/yanglu_u/ImgRepository/raw/master/images/20201120091247.png)

1. startup.sh 脚本会启动一个 JVM 来运行 Tomcat 的启动类 Bootstrap；
2. Bootstrap 初始化 Tomcat 的类加载器，并创建 Catalina；
3. Catalina 是一个启动类，它解析 server.xml 、创建相应的组件，并调用 Server 的 start 方法；
4. Server 组件的职责就是管理 Service 组件，它负责调用 Service 的 start 方法；
5. Service 组件的职责就是管理连接器和顶层容器 Engine，因此它会调用连接器和 Engine 的 start 方法；

你可以把 Bootstrap 看作是上帝，它初始化了类加载器，也就是创造万物的工具。如果我们把 Tomcat 比作是一家公司，那么 Catalina 应该是公司创始人，因为 Catalina 负责组建团队，也就是创建 Server 以及它的子组件。Server 是公司的 CEO，负责管理多个事业群，每个事业群就是一个 Service。Service 是事业群总经理，它管理两个职能部门：一个是对外的市场部，也就是连接器组件；另一个是对内的研发部，也就是容器组件。Engine 则是研发部经理，因为 Engine 是最顶层的容器组件。

你可以看到这些启动类或者组件不处理具体请求，它们的任务主要是“管理”，管理下层组件的生命周期，并且给下层组件分配任务，也就是把请求路由到负责“干活儿”的组件。因此我把它们比作 Tomcat 的“高层”。

今天我们就来看看这些“高层”的实现细节，目的是让我们逐步理解 Tomcat 的工作原理。

**Catalina**

Catalina 的主要任务就是创建 Server，它不是直接 new 一个 Server 实例就完事了，而是需要解析 server.xml，把在 server.xml 里配置的各种组件一一创建出来，接着调用 Server 组件的 init 方法和 start 方法，这样整个 Tomcat 就启动起来了。

作为“管理者”，Catalina 还需要处理各种“异常”情况，比如当我们通过“Ctrl + C”关闭 Tomcat 时，Tomcat 将如何优雅的停止并且清理资源呢？因此 Catalina 在 JVM 中注册一个“关闭钩子”。

```java
public void start() {
    //1. 如果持有的 Server 实例为空，就解析 server.xml 创建出来
    if (getServer() == null) {
        load();
    }
    //2. 如果创建失败，报错退出
    if (getServer() == null) {
        log.fatal(sm.getString("catalina.noServer"));
        return;
    }
    //3. 启动 Server
    try {
        getServer().start();
    } catch (LifecycleException e) {
        return;
    }
    // 创建并注册关闭钩子
    if (useShutdownHook) {
        if (shutdownHook == null) {
            shutdownHook = new CatalinaShutdownHook();
        }
        Runtime.getRuntime().addShutdownHook(shutdownHook);
    }
    // 用 await 方法监听停止请求
    if (await) {
        await();
        stop();
    }
}
```

JVM 在停止之前会尝试执行这个线程的 run 方法。下面我们来看看 CatalinaShutdownHook 做了些什么。

```java
protected class CatalinaShutdownHook extends Thread {
 
    @Override
    public void run() {
        try {
            if (getServer() != null) {
                Catalina.this.stop();
            }
        } catch (Throwable ex) {
           ...
        }
    }
}
```

实际上就执行了 Server 的 stop 方法，Server 的 stop 方法会释放和清理所有的资源。

**Server 组件**

Server 组件的具体实现类是 StandardServer。Server 继承了 LifecycleBase，它的生命周期被统一管理，并且它的子组件是 Service，因此它还需要管理 Service 的生命周期，也就是说在启动时调用 Service 组件的启动方法，在停止时调用它们的停止方法。Server 在内部维护了若干 Service 组件，它是以数组来保存的，那 Server 是如何添加一个 Service 到数组中的呢？

```java
@Override
public void addService(Service service) {
 
    service.setServer(this);
 
    synchronized (servicesLock) {
        // 创建一个长度 +1 的新数组
        Service results[] = new Service[services.length + 1];
        
        // 将老的数据复制过去
        System.arraycopy(services, 0, results, 0, services.length);
        results[services.length] = service;
        services = results;
 
        // 启动 Service 组件
        if (getState().isAvailable()) {
            try {
                service.start();
            } catch (LifecycleException e) {
                // Ignore
            }
        }
 
        // 触发监听事件
        support.firePropertyChange("service", null, service);
    }
 
}
```

除此之外，Server 组件还有一个重要的任务是启动一个 Socket 来监听停止端口，这就是为什么你能通过 shutdown 命令来关闭 Tomcat。不知道你留意到没有，上面 Caralina 的启动方法的最后一行代码就是调用了 Server 的 await 方法。

在 await 方法里会创建一个 Socket 监听 8005 端口，并在一个死循环里接收 Socket 上的连接请求，如果有新的连接到来就建立连接，然后从 Socket 中读取数据；如果读到的数据是停止命令“SHUTDOWN”，就退出循环，进入 stop 流程。

**Service 组件**

Service 组件的具体实现类是 StandardService，我们先来看看它的定义以及关键的成员变量。

```java
public class StandardService extends LifecycleBase implements Service {
    // 名字
    private String name = null;
    
    //Server 实例
    private Server server = null;
 
    // 连接器数组
    protected Connector connectors[] = new Connector[0];
    private final Object connectorsLock = new Object();
 
    // 对应的 Engine 容器
    private Engine engine = null;
    
    // 映射器及其监听器
    protected final Mapper mapper = new Mapper();
    protected final MapperListener mapperListener = new MapperListener(this);
}
```

StandardService 继承了 LifecycleBase 抽象类，此外 StandardService 中还有一些我们熟悉的组件，比如 Server、Connector、Engine 和 Mapper。

那为什么还有一个 MapperListener？这是因为 Tomcat 支持热部署，当 Web 应用的部署发生变化时，Mapper 中的映射信息也要跟着变化，MapperListener 就是一个监听器，它监听容器的变化，并把信息更新到 Mapper 中，这是典型的观察者模式。

作为“管理”角色的组件，最重要的是维护其他组件的生命周期。此外在启动各种组件时，要注意它们的依赖关系，也就是说，要注意启动的顺序。我们来看看 Service 启动方法：

```java
protected void startInternal() throws LifecycleException {
 
    //1. 触发启动监听器
    setState(LifecycleState.STARTING);
 
    //2. 先启动 Engine，Engine 会启动它子容器
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }
    
    //3. 再启动 Mapper 监听器
    mapperListener.start();
 
    //4. 最后启动连接器，连接器会启动它子组件，比如 Endpoint
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            if (connector.getState() != LifecycleState.FAILED) {
                connector.start();
            }
        }
    }
}
```

从启动方法可以看到，Service 先启动了 Engine 组件，再启动 Mapper 监听器，最后才是启动连接器。这很好理解，因为内层组件启动好了才能对外提供服务，才能启动外层的连接器组件。而 Mapper 也依赖容器组件，容器组件启动好了才能监听它们的变化，因此 Mapper 和 MapperListener 在容器组件之后启动。组件停止的顺序跟启动顺序正好相反的，也是基于它们的依赖关系。

**Engine 组件**

最后我们再来看看顶层的容器组件 Engine 具体是如何实现的。Engine 本质是一个容器，因此它继承了 ContainerBase 基类，并且实现了 Engine 接口。

```java
public class StandardEngine extends ContainerBase implements Engine {
    
}
```

Engine 的子容器是 Host，所以它持有了一个 Host 容器的数组，这些功能都被抽象到了 ContainerBase 中，ContainerBase 中有这样一个数据结构：

```java
protected final HashMap<String, Container> children = new HashMap<>();
```

ContainerBase 用 HashMap 保存了它的子容器，并且 ContainerBase 还实现了子容器的“增删改查”，甚至连子组件的启动和停止都提供了默认实现，比如 ContainerBase 会用专门的线程池来启动子容器。

```java
for (int i = 0; i < children.length; i++) {
   results.add(startStopExecutor.submit(new StartChild(children[i])));
}
```

所以 Engine 在启动 Host 子容器时就直接重用了这个方法。

那 Engine 自己做了什么呢？我们知道容器组件最重要的功能是处理请求，而 Engine 容器对请求的“处理”，其实就是把请求转发给某一个 Host 子容器来处理，具体是通过 Valve 来实现的。

通过专栏前面的学习，我们知道每一个容器组件都有一个 Pipeline，而 Pipeline 中有一个基础阀（Basic Valve），而 Engine 容器的基础阀定义如下：

```java
final class StandardEngineValve extends ValveBase {
 
    public final void invoke(Request request, Response response)
      throws IOException, ServletException {
  
      // 拿到请求中的 Host 容器
      Host host = request.getHost();
      if (host == null) {
          return;
      }
  
      // 调用 Host 容器中的 Pipeline 中的第一个 Valve
      host.getPipeline().getFirst().invoke(request, response);
  }
  
}
```

这个基础阀实现非常简单，就是把请求转发到 Host 容器。你可能好奇，从代码中可以看到，处理请求的 Host 容器对象是从请求中拿到的，请求对象中怎么会有 Host 容器呢？这是因为请求到达 Engine 容器中之前，Mapper 组件已经对请求进行了路由处理，Mapper 组件通过请求的 URL 定位了相应的容器，并且把容器对象保存到了请求对象中。

**课后思考**

Server 组件的在启动连接器和容器时，都分别加了锁，这是为什么呢？

答：

可能存在并发操作的场景：Tomcat 提供 MBean 的机制对管理的对象进行并发操作，如添加/删除某个service。

Server本身包含多个Service，内部实现上用数组来存储services，数组的并发操作(包含缩容，扩容)是不安全的。所以，在并发操作(添加/修改/删除/遍历等)services数组时，需要进行加锁处理。

# 12 | 实战：优化并提高Tomcat启动速度

我们在使用 Tomcat 时可能会碰到启动比较慢的问题，比如我们的系统发布新版本上线时，可能需要重启服务，这个时候我们希望 Tomcat 能快速启动起来提供服务。

**清理你的 Tomcat**

1. 清理不必要的 Web 应用

   首先我们要做的是删除掉 webapps 文件夹下不需要的工程，一般是 host-manager、example、doc 等这些默认的工程，可能还有以前添加的但现在用不着的工程，最好把这些全都删除掉。如果你看过 Tomcat 的启动日志，可以发现每次启动 Tomcat，都会重新布署这些工程。

2. 清理 XML 配置文件

   我们知道 Tomcat 在启动的时候会解析所有的 XML 配置文件，但 XML 解析的代价可不小，因此我们要尽量保持配置文件的简洁，需要解析的东西越少，速度自然就会越快。

3. 清理 JAR 文件

   我们还可以删除所有不需要的 JAR 文件。JVM 的类加载器在加载类时，需要查找每一个 JAR 文件，去找到所需要的类。如果删除了不需要的 JAR 文件，查找的速度就会快一些。这里请注意：**Web 应用中的 lib 目录下不应该出现 Servlet API 或者 Tomcat 自身的 JAR**，这些 JAR 由 Tomcat 负责提供。如果你是使用 Maven 来构建你的应用，对 Servlet API 的依赖应该指定为`<scope>provided</scope>`。

4. 清理其他文件

   及时清理日志，删除 logs 文件夹下不需要的日志文件。同样还有 work 文件夹下的 catalina 文件夹，它其实是 Tomcat 把 JSP 转换为 Class 文件的工作目录。有时候我们也许会遇到修改了代码，重启了 Tomcat，但是仍没效果，这时候便可以删除掉这个文件夹，Tomcat 下次启动的时候会重新生成。

**禁止 Tomcat TLD 扫描**

Tomcat 为了支持 JSP，在应用启动的时候会扫描 JAR 包里面的 TLD 文件，加载里面定义的标签库。可以配置一下 Tomcat 不要去扫描这些 JAR 包，这样可以提高 Tomcat 的启动速度，并节省 JSP 编译时间。

如何配置不去扫描这些 JAR 包呢？

如果你的项目没有使用 JSP 作为 Web 页面模板。找到 Tomcat 的`conf/`目录下的`context.xml`文件，在这个文件里 Context 标签下，加上 JarScanner 和 JarScanFilter 子标签，像下面这样。

```xml
<Context>
  <JarScanner>
    <JarScanFilter defaultTldScan="false" />
  </JarScanner>
</Context>
```

如果你的项目使用了 JSP 作为 Web 页面模块，可以通过配置来告诉 Tomcat，只扫描那些包含 TLD 文件的 JAR 包。找到 Tomcat 的`conf/`目录下的`catalina.properties`文件，在这个文件里的 jarsToSkip 配置项中，加上你的 JAR 包。

```text
tomcat.util.scan.StandardJarScanFilter.jarsToSkip=xxx.jar
```

**关闭 WebSocket 支持**

Tomcat 会扫描 WebSocket 注解的 API 实现，比如`@ServerEndpoint`注解的类。我们知道，注解扫描一般是比较慢的，如果不需要使用 WebSockets 就可以关闭它。具体方法是，找到 Tomcat 的`conf/`目录下的`context.xml`文件，给 Context 标签加一个**containerSciFilter**的属性，像下面这样。

```xml
<Context containerSciFilter="org.apache.tomcat.websocket.server.WsSci">
  ...
</Context>
```

**关闭 JSP 支持**

跟关闭 WebSocket 一样，如果你不需要使用 JSP，可以通过类似方法关闭 JSP 功能，像下面这样。

```xml
<Context containerSciFilter="org.apache.jasper.servlet.JasperInitializer">
  ...
</Context>
```

我们发现关闭 JSP 用的也是**containerSciFilter**属性，如果你想把 WebSocket 和 JSP 都关闭，那就这样配置：

```xml
<Context containerSciFilter="org.apache.tomcat.websocket.server.WsSci | org.apache.jasper.servlet.JasperInitializer">
  ...
</Context>
```

**禁止 Servlet 注解扫描**

Servlet 3.0 引入了注解 Servlet，Tomcat 为了支持这个特性，会在 Web 应用启动时扫描你的类文件，因此如果你没有使用 Servlet 注解这个功能，可以告诉 Tomcat 不要去扫描 Servlet 注解。具体配置方法是，在你的 Web 应用的`web.xml`文件中，设置`<web-app>`元素的属性`metadata-complete="true"`，像下面这样。

```xml
<web-app metadata-complete="true">
  ...
</web-app>
```

`metadata-complete`的意思是，`web.xml`里配置的 Servlet 是完整的，不需要再去库类中找 Servlet 的定义。

**配置 Web-Fragment 扫描**

Servlet 3.0 还引入了“Web 模块部署描述符片段”的`web-fragment.xml`，这是一个部署描述文件，可以完成`web.xml`的配置功能。而这个`web-fragment.xml`文件必须存放在 JAR 文件的`META-INF`目录下，而 JAR 包通常放在`WEB-INF/lib`目录下，因此 Tomcat 需要对 JAR 文件进行扫描才能支持这个功能。

你可以通过配置`web.xml`里面的`<absolute-ordering>`元素直接指定了哪些 JAR 包需要扫描`web fragment`，如果`<absolute-ordering/>`元素是空的， 则表示不需要扫描，像下面这样。

```xml
<web-app>
  ...
  <absolute-ordering />  
  ...
</web-app>
```

**随机数熵源优化**

这是一个比较有名的问题。Tomcat 7 以上的版本依赖 Java 的 SecureRandom 类来生成随机数，比如 Session ID。而 JVM 默认使用阻塞式熵源（`/dev/random`）， 在某些情况下就会导致 Tomcat 启动变慢。当阻塞时间较长时， 你会看到这样一条警告日志：

```text
<DATE> org.apache.catalina.util.SessionIdGenerator createSecureRandom
INFO: Creation of SecureRandom instance for session ID generation using [SHA1PRNG] took [8152] milliseconds.
```

这其中的原理我就不展开了，你可以阅读[资料](https://stackoverflow.com/questions/28201794/slow-startup-on-tomcat-7-0-57-because-of-securerandom)获得更多信息。解决方案是通过设置，让 JVM 使用非阻塞式的熵源。

我们可以设置 JVM 的参数：

```text
-Djava.security.egd=file:/dev/./urandom
```

或者是设置`java.security`文件，位于`$JAVA_HOME/jre/lib/security`目录之下： `securerandom.source=file:/dev/./urandom`

这里请你注意，`/dev/./urandom`中间有个`./`的原因是 Oracle JRE 中的 Bug，Java 8 里面的 SecureRandom 类已经修正这个 Bug。 阻塞式的熵源（`/dev/random`）安全性较高， 非阻塞式的熵源（`/dev/./urandom`）安全性会低一些，因为如果你对随机数的要求比较高， 可以考虑使用硬件方式生成熵源。

**并行启动多个 Web 应用**

Tomcat 启动的时候，默认情况下 Web 应用都是一个一个启动的，等所有 Web 应用全部启动完成，Tomcat 才算启动完毕。如果在一个 Tomcat 下你有多个 Web 应用，为了优化启动速度，你可以配置多个应用程序并行启动，可以通过修改`server.xml`中 Host 元素的 startStopThreads 属性来完成。startStopThreads 的值表示你想用多少个线程来启动你的 Web 应用，如果设成 0 表示你要并行启动 Web 应用，像下面这样的配置。

```xml
<Engine startStopThreads="0">
  <Host startStopThreads="0">
    ...  
  </Host>
</Engine>
```

这里需要注意的是，Engine 元素里也配置了这个参数，这意味着如果你的 Tomcat 配置了多个 Host（虚拟主机），Tomcat 会以并行的方式启动多个 Host。

**课后思考**

在 Tomcat 启动速度优化上，你都遇到了哪些问题？

