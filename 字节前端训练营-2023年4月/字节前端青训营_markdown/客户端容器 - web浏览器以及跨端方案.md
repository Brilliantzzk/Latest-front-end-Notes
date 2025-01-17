# 客户端容器 - web浏览器以及跨端方案

## 浏览器架构

### 浏览器架构演进

1. 单进程架构：所有模块运行在同一个进程里，包含网络、插件、JavaScript运行环境等
   - 在早期的浏览器中，所有的功能都在一个进程中完成，包括用户界面、JavaScript执行、CSS渲染和网络请求等。单进程架构的优势是简单、易于实现，但缺点也很明显，如：不稳定（一个标签页崩溃可能导致整个浏览器崩溃）、不安全（攻击者可以利用漏洞来控制整个浏览器）以及性能瓶颈（多个标签页争抢资源，导致性能下降）。
2. 多进程架构：主进程、网络进程、渲染进程、GPU进程、插件进程
   - 为了解决单进程架构的问题，浏览器厂商开始将浏览器的功能拆分为多个进程。例如，谷歌推出的Chrome浏览器就采用了多进程架构。在这种架构下，浏览器将每个标签页（或每个渲染实例）分配给一个单独的进程，从而实现了进程间的隔离，防止单个网页崩溃影响到其他界面。多进程架构提高了浏览器的稳定性、安全性和性能，但同时也增加了内存消耗
   - 每一个Tab运行在独立的沙盒里面
3. 面向服务架构：将原来的UI、数据库、文件、设备、网络等，作为一个独立的基础服务
   - 面向服务架构 => 多进程架构升级版本
   - 随着Web平台的发展，浏览器需要处理越来越多的任务，这使得多进程架构变得复杂且难以维护。面向服务架构的出现旨在将浏览器的各个功能模块化，将它们作为独立的服务来运行和维护。在这种架构下，每个服务都可以独立地使用一个或多个进程，根据需求自动扩展或缩减资源。这样可以进一步提高浏览器的稳定性、安全性和性能，同时优化资源利用率

![image-20230419104937891](.\byte-images\image-20230419104937891.png)

### 浏览器架构对比

![image-20230419105022871](.\byte-images\image-20230419105022871.png)

### 浏览器架构 - 任务管理器

> 在多进程浏览器架构中，任务管理器是一个非常重要的组件。任务管理器负责监控、管理和分配浏览器的各个进程，确保整个浏览器系统的稳定性和性能。
>
> 任务管理器主要负责以下几方面的功能：
>
> 1. 进程监控：任务管理器可以实时查看浏览器中的各个进程，包括标签页进程、插件进程和扩展进程等。它可以展示各个进程的资源使用情况，如内存占用、CPU使用率和网络活动等。
> 2. 进程管理：任务管理器负责启动和关闭浏览器的进程。当用户打开一个新的标签页或启动一个插件时，任务管理器会创建相应的进程。当标签页被关闭或插件停止运行时，任务管理器会结束相应的进程，以释放系统资源。
> 3. 资源分配：任务管理器根据各个进程的资源需求，动态地分配系统资源。例如，对于需要大量CPU资源的进程，任务管理器会分配更多的CPU时间片。同时，任务管理器还可以确保关键进程（如用户界面进程）始终获得足够的资源，从而保证整个浏览器的流畅运行。
> 4. 进程隔离：为了提高浏览器的安全性，任务管理器会对各个进程进行隔离。这意味着一个进程无法直接访问另一个进程的内存空间。进程隔离可以防止潜在的安全漏洞影响到整个浏览器系统。
> 5. 异常处理：任务管理器负责检测和处理进程中的异常情况。当某个进程发生崩溃或无响应时，任务管理器会尝试自动恢复该进程。如果无法恢复，任务管理器会提示用户关闭相应的标签页或插件，以防止整个浏览器崩溃。
>
> 通过任务管理器，浏览器可以有效地监控和管理其各个进程，确保整个系统的稳定性、性能和安全性。不同浏览器的任务管理器实现可能会有所差异，但它们的核心功能和目标都是类似的。在谷歌Chrome浏览器中，用户可以通过按下`Shift + Esc`快捷键或者在菜单中选择“更多工具” > “任务管理器”来打开任务管理器。

![image-20230419105818510](.\byte-images\image-20230419105818510.png)

### 浏览器架构 - 多进程分工

|       进程名称       |                           进程描述                           |
| :------------------: | :----------------------------------------------------------: |
|    浏览器(主进程)    | 主要负责页面展示逻辑，用户交互，子进程管理；包括地址栏、书签、前进、后退、收藏夹等 |
|       GPU进程        |               负责UI绘制，包含整个浏览器全部UI               |
|       网络进程       |                网络服务进程，负责网络资源加载                |
| **标签页(渲染进程)** | **控制tab内的所有内容，将HTML、CSS和JavaScript转换为用户可交互的网页** |
|       插件进程       |          控制网站运行的插件，比如flash、ModHeader等          |
|       其他进程       |     如上图所示：适用程序Storage/Network/Audio Service等      |

### 问题思考

1. **为什么会有单进程架构？**

   - 单进程架构在早期的浏览器中出现，当时的Web技术相对较为简单，浏览器需要处理的任务和复杂性远不及现在。在这个背景下，单进程架构具有一些优势：

     1. 简单性：单进程架构更易于实现和维护，因为它不需要处理多进程间的通信和资源分配问题。对于早期的浏览器开发者来说，单进程架构提供了一种快速实现浏览器功能的方法。
     2. 资源消耗：相较于多进程架构，单进程架构在内存和CPU资源消耗方面较低。因为多进程架构中，每个进程都需要占用一定的系统资源，而单进程架构只需要维护一个进程。在计算机硬件资源有限的早期，这一点是非常重要的。
     3. 兼容性：早期的Web技术和标准相对较为简单，单进程架构在兼容性方面表现优秀。因为不需要处理多进程间的兼容性问题，单进程浏览器更容易支持不同的操作系统和硬件平台。

     然而，随着Web技术的发展，浏览器需要处理越来越复杂的任务，如多媒体播放、实时通信和大量的JavaScript代码等。这使得单进程架构的局限性变得越来越明显，包括稳定性、安全性和性能等方面的问题。因此，浏览器厂商开始采用多进程架构和面向服务架构，以适应这些新的挑战。

     单进程架构在早期的浏览器中出现，是因为它在简单性、资源消耗和兼容性方面具有优势。然而，随着Web技术的发展，这种架构的局限性逐渐暴露，导致现代浏览器转向多进程架构和面向服务架构

2. **面向服务架构是否会替代多进程架构？**

   - 面向服务架构在某种程度上可以看作是多进程架构的一种演进和优化。这种架构将浏览器的各个功能模块化，作为独立的服务来运行和维护。这样可以进一步提高浏览器的稳定性、安全性和性能，同时优化资源利用率。

     尽管面向服务架构具有一定的优势，但是它不一定会完全替代多进程架构。这是因为两种架构各自有不同的应用场景和优缺点：

     1. 面向服务架构更适用于复杂的浏览器系统，它可以更好地管理和维护浏览器的各个功能模块。然而，在一些轻量级或特定用途的浏览器中，多进程架构可能仍然是一个合适的选择，因为它相对简单且易于实现。
     2. 面向服务架构的优势在于模块化和灵活性，它可以根据需求自动扩展或缩减资源。但是，**这种架构可能需要更多的系统资源和开发维护成本。多进程架构在资源消耗和开发成本方面可能更具优势**。
     3. 不同浏览器厂商可能会根据自己的产品定位和技术栈选择不同的架构。例如，谷歌Chrome浏览器采用了多进程架构，并在此基础上引入了一些面向服务架构的设计理念。而其他浏览器厂商**可能会选择完全基于面向服务架构**的实现。

     总之，面向服务架构在某些方面具有优势，但它不一定会完全替代多进程架构。两种架构各自适用于不同的应用场景，浏览器厂商可能会根据实际需求和技术栈选择合适的架构。随着Web技术的不断发展，未来浏览器架构可能会继续演进，以适应更多的需求和挑战。

## 渲染进程

### 常见浏览器内核

|         内核          |               浏览器                |     JS引擎      |                           补充说明                           |
| :-------------------: | :---------------------------------: | :-------------: | :----------------------------------------------------------: |
|        Trident        |               IE4-11                | JScript，Chakra | 出生于1994年，IE8以前使用JScript引擎，IE9开始使用Chakra引擎  |
|         Gecko         |               Firefox               |  SpiderMonkey   | Gecko内核主要用在Firefox浏览器上，同时是一个跨平台的内核，支持在Windows、BSD、Linux、Mac OS X中使用 |
|        Webkit         | Safari、Chrome、Android(安卓)浏览器 | JavaScriptCore  |           由Apple公司技术团队开发，并在2005年开源            |
|         Blink         |            Chrome，Opera            |       V8        | Google:基于Webkit开发的内核，在Vebkit的基础上加入多进程，沙箱等技术，于2013年开源。(NodeJS使用的就是这个引擎) |
|         Edge          |                Edge                 |     Chakra      | 2015年由微软发布，用于Edge浏览器上，由于性能较差，运行不稳定等原因，2018年微软将Edge浏览器内核迁移到Chromium |
| Trident+Webkit(Blink) |    国产浏览器QQ、360、搜狗、UC等    |      都有       | 早期银行系统都是在IE上进行开发，想要支持银行系统就切换到Trident内核，想要体验就切到Webkit内核 |

### 渲染进程 - 多线程架构

> 在这里我们涉及到了(多)线程，前面涉及到了(多)进程。这里补充额外知识来说明两者：
>
> - 进程和线程都是操作系统进行任务管理的基本单位，它们之间有一定的关联性，但也有明显的区别
>
> 1. 进程：
>
> 进程是一个独立的程序执行实例，它包括程序的代码、数据以及运行时所需的系统资源（如文件描述符、内存空间等）。每个进程都有自己的独立地址空间，操作系统会为每个进程分配和管理资源。
>
> 进程的特点：
>
> - 独立性：进程具有独立的地址空间，一个进程无法直接访问另一个进程的资源，除非通过进程间通信（IPC）机制。
> - 隔离性：操作系统会为每个进程分配和管理资源，确保它们之间的隔离。这有助于提高系统的稳定性和安全性。
> - 资源开销：创建和维护进程需要消耗一定的系统资源，如内存、CPU时间等。
>
> 应用场景：进程通常用于执行独立的、需要隔离的任务。例如，在多进程浏览器中，每个Tab标签页都运行在一个单独的进程中，以实现进程间的隔离和资源管理。
>
> 2. 线程：
>
> 线程是进程内的一个执行单元，它共享进程的地址空间和资源，但拥有自己的执行堆栈和程序计数器。一个进程可以包含多个线程，它们可以并发地执行任务。
>
> 线程的特点：
>
> - 轻量级：线程比进程更轻量级，创建和切换线程的开销较小。
> - 共享资源：线程之间可以共享进程的地址空间和资源，这使得线程间的通信和数据共享变得更加简单高效。
> - 上下文切换：线程之间的上下文切换成本较低，因为它们共享相同的地址空间。
>
> 应用场景：线程通常用于实现并发执行和任务分解。例如，在一个图形编辑器中，可以使用多个线程分别处理用户输入、图形渲染和文件保存等任务，从而提高程序的响应速度和执行效率。
>
> 总结：进程是一个独立的程序执行实例，具有独立的地址空间和资源；线程是进程内的一个执行单元，共享进程的地址空间和资源。进程通常用于执行独立的、需要隔离的任务，而线程则用于实现并发执行和任务分解

- 内部是多线程实现，主要负责页面渲染，脚本执行，事件处理，网络请求等

![image-20230419111933405](.\byte-images\image-20230419111933405.png)

### JS引擎 VS 渲染引擎

> JS引擎和渲染引擎是浏览器中两个非常重要的组件。它们分别负责处理JavaScript代码和渲染网页内容

1. 解析执行JS
   - JS引擎：JS引擎负责解析和执行JavaScript代码。它会将JavaScript源代码解析成抽象语法树（AST），然后通过JIT编译器将AST编译成机器代码，最后执行机器代码。在这个过程中，JS引擎还会进行优化，提高代码的执行效率。常见的JS引擎包括V8（用于Chrome和Node.js）、SpiderMonkey（用于Firefox）和JavaScriptCore（用于Safari）。
   - 渲染引擎：渲染引擎的主要任务是解析HTML和CSS，构建DOM树和渲染树，然后将渲染树转换为屏幕上的像素。在这个过程中，渲染引擎并不直接处理JavaScript代码。然而，当遇到需要执行的JavaScript代码时（例如，通过`<script>`标签或事件处理程序），渲染引擎会将控制权交给JS引擎。一旦JS引擎执行完毕，渲染引擎会继续执行后续的渲染任务。
2. XML解析生成渲染树，显示在屏幕
   - JS引擎：JS引擎主要处理JavaScript代码，不负责解析XML或生成渲染树。然而，JavaScript可以通过DOM API操作DOM树，从而间接地影响渲染树的生成和更新。
   - 渲染引擎：渲染引擎负责解析HTML和XML文档，构建DOM树。然后，渲染引擎会解析CSS样式，并将样式信息附加到DOM树上，生成渲染树。接下来，渲染引擎会计算每个节点的布局信息（如位置和大小），并将渲染树转换为屏幕上的像素。这个过程被称为“绘制”或“渲染”。
3. 桥接方式通信
   - JS引擎和渲染引擎之间需要进行通信，以确保JavaScript代码的执行和网页内容的渲染能够协同工作。这种通信通常通过浏览器内部的API和事件机制实现。
   - JS引擎：JS引擎提供了DOM API，使得JavaScript代码可以访问和操作DOM树。当JavaScript代码需要对DOM树进行操作时（如添加、删除或修改节点），JS引擎会通过浏览器内部的API通知渲染引擎进行相应的

![image-20230419112646106](.\byte-images\image-20230419112646106.png)

### 渲染进程 - 多线程工作流程

1. 网络线程负责加载网页资源
   - 网络线程的主要任务是从服务器加载网页资源，如HTML、CSS、JavaScript、图片等。加载过程中，网络线程会将HTML和CSS交给渲染线程进行解析，并将JavaScript交给JS引擎进行解析和执行
2. JS引擎解析JS脚本并且执行
   - JS引擎将接收到的JavaScript代码解析成抽象语法树（AST），然后通过JIT编译器将AST编译成机器代码，最后执行机器代码。在这个过程中，JS引擎还会进行优化，提高代码的执行效率
3. JS解析引擎空闲时，渲染线程立即工作
   - 当JS引擎空闲时（即没有JavaScript代码需要执行），渲染线程开始工作。渲染线程负责解析HTML和CSS，构建DOM树和渲染树，计算布局信息，并将渲染树转换为屏幕上的像素。如果在此过程中遇到JavaScript代码（如`<script>`标签或事件处理程序），渲染线程会暂停，将控制权交给JS引擎
4. 用户交互、定时器操作等产生回调函数放入任务队列中
   - 当用户与网页进行交互（如点击、滚动等），或者有定时器触发（如`setTimeout`或`setInterval`）时，相应的回调函数会被放入任务队列中。任务队列是一个先进先出（FIFO）的数据结构，用于存储待处理的任务
5. 事件线程进行事件循环，将队列里的任务取出交给JS引擎执行
   - 事件线程负责管理事件循环。事件循环的主要任务是检查任务队列，将队列中的任务按顺序取出，并交给JS引擎执行。JS引擎执行任务时，可能会对DOM树进行操作，从而触发渲染线程的更新。事件循环会持续进行，直到任务队列为空，或者浏览器关闭。

![image-20230419114102609](.\byte-images\image-20230419114102609.png)

### 八股文笔试题

> 这段代码主要用于演示`setTimeout`的执行顺序以及JavaScript的单线程执行特点
>
> 1. 输出1：定时器1的回调函数在10ms后加入事件队列。然而，由于`while`循环会阻塞主线程，直到至少20ms过去，所以定时器1的回调函数将在`while`循环结束后立即执行。因此，输出1的值会大于等于20ms，例如："time10 20"。
> 2. 输出2：定时器2的回调函数在30ms后加入事件队列。因为`while`循环在20ms后就结束了，所以定时器2的回调函数会在预期的30ms后执行。因此，输出2的值会接近30ms，例如："time30 30"。
> 3. 输出3：这个输出表示`while`循环结束时已经过去的时间。由于`while`循环会一直执行，直到至少20ms过去，所以输出3的值会大于等于20ms，例如："20"。
>
> 主要注意的是，由于JavaScript的单线程特性和事件循环机制，`setTimeout`并不能保证回调函数会在精确的延迟时间后执行。实际的执行时间可能会受到其他任务（如`while`循环）的影响。

```javascript
// 获取当前时间戳
const now = Date.now();

// 定时器1：10ms后输出"time10"和定时器1的执行延时
setTimeout(() => {
  console.log('time10', Date.now() - now); // 输出1：？
}, 10);

// 定时器2：30ms后输出"time30"和定时器2的执行延时
setTimeout(() => {
  console.log('time30', Date.now() - now); // 输出2：？
}, 30);

// 一直执行循环，直到距离当前时间戳至少过去20ms
while (true) {
  if (Date.now() - now >= 20) {
    break;
  }
}

// 输出当前已经过去的时间（自now开始计算）
console.log(Date.now() - now); // 输出3：？

```

## Chrome运行原理

### Chrome运行原理 - 如何展示网页

- 浏览器地址输入URL后发生了什么(完整流程)

> 1. 输入解析：浏览器首先解析输入的内容，**判断它是一个有效的URL还是搜索查询**。如果输入的内容被识别为搜索查询，浏览器将使用默认的搜索引擎进行搜索。
> 2. DNS查询：如果输入的内容是一个有效的URL，浏览器将进行域名系统（DNS）查询以将域名解析为对应的IP地址。浏览器会首先检查本地DNS缓存，如果找不到对应的记录，它会向DNS服务器发送查询请求。
> 3. 建立连接：浏览器与目标服务器建立一个TCP连接。这通常包括三次握手过程，以确保双方都准备好进行数据传输。
> 4. 发送HTTP请求：TCP连接建立后，浏览器向服务器发送一个HTTP请求。请求通常包括请求方法（如GET或POST）、请求的资源路径、HTTP协议版本、请求头（包含诸如用户代理、接受的编码和语言等信息）以及可能的请求体（如POST请求所包含的表单数据）。
> 5. 接收响应：服务器处理HTTP请求，并将响应数据发送回浏览器。响应通常包括HTTP状态码（如200表示成功，404表示未找到等）、响应头（包含诸如内容类型、内容长度等信息）以及响应体（通常是HTML文档）。
> 6. 关闭或重用连接：一旦浏览器接收到完整的响应数据，它可以选择关闭TCP连接或将其保持在活动状态以用于后续请求。
> 7. 解析HTML：浏览器解析HTML文档，构建DOM树。在解析过程中，浏览器可能遇到`<script>`标签或其他需要立即执行的脚本，这时浏览器将暂停解析，执行脚本，然后继续解析。
> 8. 请求其他资源：HTML文档通常包含其他资源的引用，如CSS、JavaScript和图片等。浏览器将发送额外的HTTP请求来获取这些资源。这些请求可能与初始请求的服务器相同，也可能涉及其他服务器。
> 9. 构建渲染树：浏览器解析CSS样式，并将其应用于DOM树，生成渲染树。渲染树包含了所有可见元素及其样式信息。
> 10. 布局：浏览器计算渲染树中每个元素的位置和大小，生成布局信息。
> 11. 绘制：浏览器根据渲染树和布局信息将元素绘制到屏幕上。这个过程称为绘制或渲染。浏览器会将各个层的元素绘制到位图中，然后将这些位图合成到屏幕上显示的最终图像。
> 12. 交互：在完成页面绘制后，浏览器开始接收和处理用户的交互，如点击、滚动、输入等。这些交互可能会触发JavaScript事件处理程序，从而修改DOM或应用新的样式。这些修改可能会导致浏览器重新布局和绘制页面的部分或全部内容。
> 13. 关闭或卸载：当用户导航到其他页面或关闭浏览器选项卡时，浏览器将触发相应的页面卸载事件，如`beforeunload`和`unload`。这给开发者一个机会来执行清理操作，如保存用户数据或取消挂起的网络请求。一旦完成这些操作，浏览器将卸载页面并释放相关资源。
>
> 总结：当您在浏览器地址栏输入URL后，浏览器会经历一系列的操作，从DNS查询和TCP连接建立，到HTTP请求、响应处理、HTML解析、资源加载、渲染树构建、布局、绘制以及用户交互等。这些操作共同构成了从输入URL到显示完整网页的整个过程

![image-20230419120646523](.\byte-images\image-20230419120646523.png)

视频中的流程(属于浏览器主进程中的导航流程，目的是为了保证用户能够成功加载并显示所请求的网页)：

1. 输入处理：浏览器主进程会根据用户输入的URL进行解析和处理，包括检查URL的有效性、判断是否需要使用搜索引擎、判断是否需要使用HTTPS等等。在处理完输入之后，浏览器主进程将发起开始导航的请求。
2. 开始导航：浏览器主进程会向网络进程发送开始导航的请求，以便进行后续的页面加载操作。在发送请求之前，浏览器主进程会先判断当前页面是否支持复用，如果支持，则会重用渲染进程，否则会创建一个新的渲染进程来加载页面。
3. 读取响应：一旦网络进程开始加载页面，它会读取服务器响应的数据，并将其返回给浏览器主进程。在读取响应的过程中，网络进程会进行数据解码和安全检查等操作。
4. 寻找渲染进程：一旦浏览器主进程收到网络进程返回的响应数据，它会根据响应数据中的渲染进程ID来寻找对应的渲染进程。如果已经存在对应的渲染进程，则将响应数据发送给该进程进行处理；否则，浏览器主进程会创建一个新的渲染进程来处理该页面。

####  输入处理

1. 用户Url框输入内容的后，UI线程会判断输入的是一个URL地址呢，还是一个query查询条件
2. 如果是URL,直接请求站点资源
3. 如果是query,将输入发送给搜索引擎

<img src=".\byte-images\image-20230419124209421.png" style="zoom:50%;" />

#### 开始导航

1. 当用户按下回车，UI线程通知网络线程发起一个网络请求，来获取站点内容
2. 请求过程中，tab处于loading(转圈等待)状态

<img src=".\byte-images\image-20230419124416471.png" style="zoom:50%;" />

#### 读取响应

1. 网络线程接收到HTTP响应后，先检查响应头的媒体类型(MIME Type)
2. 如果响应主体是一个HTML文件，浏览器将内容交给渲染进程处理
3. 如果拿到的是其他类型文件，比如Zip、exe等，则交给下载管理器处理

<img src=".\byte-images\image-20230419124613005.png" style="zoom:50%;" />

#### 寻找渲染进程

1. 网络线程做完所有检查后，会告知主进程数据已准备完毕，主进程确认后为这个站点寻找一个渲染进程
2. 主进程通过IPC消息告知渲染进程去处理本次导航
3. 渲染进程开始接收数据并告知主进程自己已开始处理，导航结束，进入文档加载阶段

<img src=".\byte-images\image-20230419124758253.png" alt="image-20230419124758253" style="zoom:50%;" />

#### 渲染进程 - 资源加载

1. 收到主进程的消息后，开始加载HTML文档
2. 除此之外，还需要加载子资源，比如一些图片，CSS样式文件以及JavaScript脚本

<img src=".\byte-images\image-20230419125127692.png" alt="image-20230419125127692" style="zoom:50%;" />

#### 渲染进程 - 构建渲染树

1. 构建DOM树，将HTML文本转化成浏览器能够理解的结构
2. 构建CSSOM树，浏览器同样不认识CSS,需要将CSS代码转化为可理解的CSSOM
3. 构建渲染树，渲染树是DOM树和CSSOM树的结合

![image-20230419125252544](.\byte-images\image-20230419125252544.png)

#### 渲染进程 - 页面布局

1. 根据渲染树计算每个节点的位置和大小
2. 在浏览器页面区域绘制元素边框
3. 遍历渲染树，将元素以盒模型的形式写入文档流

<img src=".\byte-images\image-20230419125551219.png" alt="image-20230419125551219" style="zoom:33%;" />

#### 渲染进程 - 页面绘制

1. 构建图层：为特定的节点生成专用图层
2. 绘制图层：一个图层分成很多绘制指令，然后将这些指令按顺序组成一个绘制列表，交给**合成线程**(减少绘制次数)
3. 合成线程接收指令生成**图块**
4. 栅格线程将图块进行栅格化
5. 展示在屏幕上

<img src=".\byte-images\image-20230419140524397.png" alt="image-20230419140524397" style="zoom:33%;" />

#### 前端性能performance

> `performance`对象中常用的属性和方法：
>
> 1. `navigationStart`：浏览器开始导航到当前页面的时间戳。
> 2. `fetchStart`：浏览器开始检索文档的时间戳。
> 3. `responseStart`：浏览器收到第一个字节的时间戳。
> 4. `domLoading`：浏览器开始解析文档的时间戳。
> 5. `domInteractive`：浏览器完成解析文档的时间戳。
> 6. `domContentLoadedEventStart`：文档DOMContentLoaded事件开始的时间戳。
> 7. `domComplete`：文档解析完成的时间戳。
> 8. `loadEventStart`：文档加载事件开始的时间戳。
> 9. `loadEventEnd`：文档加载事件结束的时间戳。
>
> - 最常用的是`navigationStart`和`loadEventEnd`。`navigationStart`表示浏览器开始导航到当前页面的时间戳，`loadEventEnd`表示文档加载事件结束的时间戳。这两个时间戳之差就是页面的加载时间

1. 时间都花在哪？

2. 什么情况下卡顿

   卡顿一般是由于浏览器在渲染页面时遇到了阻塞或耗时操作导致的。以下是一些可能导致卡顿的情况：

   1. JavaScript执行时间过长：JavaScript的执行会阻塞页面的渲染。如果脚本执行时间过长，会导致页面无响应或卡顿现象。
   2. 大量的DOM操作：DOM操作会影响页面的渲染，特别是在大量DOM操作时。如果需要频繁地更新DOM元素，可以考虑使用虚拟DOM等技术来减少DOM操作次数。
   3. 大量的网络请求：浏览器在渲染页面时需要下载和解析HTML、CSS、JavaScript和图像等资源。如果有大量的资源需要下载，会导致页面加载时间过长。
   4. 大量的样式和布局计算：如果页面包含大量的样式和布局计算，会影响页面的渲染性能。
   5. 阻塞渲染的JavaScript：如果**脚本阻塞了页面的渲染，就会导致卡顿或页面无响应**。

![image-20230419142855570](.\byte-images\image-20230419142855570.png)

### 首屏优化

1. 压缩、分包、删除无用代码
   - 通过压缩代码、分包加载和删除无用代码等技术，可以减小页面的体积，加快页面的加载速度。
2. 静态资源分离
   - 将页面中的静态资源（如CSS、JavaScript和图像等）与HTML文档分离，可以使得浏览器可以并行加载这些资源，从而提高页面的加载速度
3. JS脚本非阻塞加载
   - 将JS脚本异步加载，可以减少页面的渲染阻塞，从而提高页面的加载速度。可以使用`defer`和`async`等属性来实现JS的非阻塞加载
4. 缓存策略
   - 合理地设置缓存策略，可以减少对服务器的请求，加快页面的加载速度。可以使用HTTP响应头中的`Cache-Control`和`Expires`等属性来设置缓存策略
5. SSR
   - 服务器端渲染（Server Side Rendering）可以在服务器端生成HTML文档，减少客户端渲染的工作量，从而提高页面的加载速度。SSR适用于复杂的单页面应用或对SEO有要求的应用
6. 预置loading、骨架屏
   - 在页面加载过程中，可以预置一个loading动画或骨架屏，以提高用户体验。这些技术可以在页面加载完成之前，先显示一些占位元素，给用户一个等待的感觉，从而减少用户等待的焦虑和不安

综合利用这些技术，可以大大提高页面的加载速度和用户体验。在进行首屏优化时，需要综合考虑页面的性能、体验和功能等方面，找到最合适的优化方案

### 渲染优化

1. GPU加速
   - 将复杂的图形处理任务交给GPU来处理，可以加快页面的渲染速度。可以使用CSS3的`transform`和`opacity`等属性来开启GPU加速
2. 减少回流、重绘
   - 回流和重绘是影响页面性能的主要因素之一。可以通过避免使用影响布局的属性、批量修改DOM元素等技术来减少回流和重绘操作
3. 离屏渲染
   - 离屏渲染是将页面中的部分内容在单独的图层中进行渲染，从而减少对主渲染线程的阻塞。可以使用CSS3的`transform`和`position`等属性来开启离屏渲染。
4. 懒加载
   - 将页面中的非必要资源（如图片和视频等）延迟加载，可以加快页面的加载速度。可以使用`Intersection Observer`和`Lazyload`等技术来实现懒加载

### JS优化

1. 防止内存泄漏

   - 有可能出现内存泄漏的场景
     - 全局变量：全局变量会一直存在于内存中，直到程序结束才会被释放。如果程序中定义了大量的全局变量，就会导致内存占用过多，从而导致内存泄漏。
     - 闭包：闭包会在函数中保存局部变量和参数，如果函数执行后，闭包中的变量没有被释放，就会导致内存泄漏。为了避免内存泄漏，应该合理使用闭包，并注意释放不需要的变量。
     - 循环引用：循环引用是指两个或多个对象之间相互引用，形成了一个死循环，导致内存无法释放。为了避免循环引用，应该及时释放不需要的引用，并使用垃圾回收机制来自动释放内存。
     - 定时器和事件监听器：定时器和事件监听器会持续占用内存，直到被清除或被解除绑定。如果程序中存在大量的定时器和事件监听器，就会导致内存占用过多，从而导致内存泄漏。
     - DOM节点：DOM节点也会占用内存空间，如果程序中存在大量的DOM节点，就会导致内存占用过多，从而导致内存泄漏。为了避免内存泄漏，应该及时清除不需要的DOM节点
   - 内存泄漏会导致不必要的内存占用和程序崩溃。可以使用`let`和`const`关键字声明变量，避免变量污染和内存泄漏

2. 循环尽早break

   - 在循环中，如果已经找到了需要的结果，可以使用`break`语句尽早结束循环，避免无用的迭代和计算

3. 合理使用闭包

   - 闭包可以在函数中保存局部变量和参数，避免全局变量的污染和泄漏。但是，如果使用不当，也会导致内存泄漏和性能下降

4. 减少Dom访问

   - DOM操作是JavaScript性能的一个瓶颈。可以使用缓存和批量操作等技术来减少DOM访问次数，从而提高JavaScript的性能

5. 防抖、节流

   > 1. 防抖（debounce）： 防抖是指在一定时间内，如果连续触发事件，那么只执行一次目标函数，理解为王者荣耀的回城。常用于输入框实时搜索、窗口大小调整等场景。防抖的实现原理是：设置一个定时器，在指定的延迟时间内，如果再次触发事件，则重新计时。只有在延迟时间内没有再次触发事件时，才会执行目标函数。
   > 2. 节流（throttle）： 节流是指在一定时间内，无论触发多少次事件，目标函数都只执行一次，理解为技能CD。常用于滚动事件、鼠标移动等场景。节流的实现原理是：设置一个间隔时间，在这个时间内，无论事件触发多少次，都只执行一次目标函数。一旦超过这个间隔时间，就会再次执行目标函数。
   >
   > 总结： 防抖和节流的主要区别在于，防抖是在一定时间内只执行一次目标函数，而节流是在一定时间内控制目标函数执行次数。它们都可以有效地减少函数执行频率，降低性能开销，提高用户体验。

   - 防抖和节流是用来控制函数调用频率的技术。可以使用`setTimeout`和`requestAnimationFrame`等API来实现防抖和节流(或者用第三方库也行)

6. Web Workers

   - Web Workers是一种在后台线程中执行JavaScript代码的技术。可以将耗时的计算任务和数据处理等操作放到Web Workers中执行，避免阻塞主线程，提高页面的响应速度

## 跨端容器

### 为什么需要跨端

1. 开发成本、效率
   - 跨端开发可以帮助降低成本和提高开发效率。使用跨端技术，开发者只需编写一份代码，就可以在多个平台（如iOS、Android和Web）上运行。这可以减少开发和维护的工作量，节省时间和资源。同时，开发团队可以更快地推出新功能和修复问题，因为他们只需关注一份代码库。
2. 一致性体验
   - 跨端开发可以确保在不同平台上提供一致的用户体验。使用跨端技术，开发者可以更容易地保持应用的外观和功能一致，无论用户在什么设备上使用。这有助于提高用户满意度和用户留存率。
3. 前端开发生态
   - 跨端开发受益于强大的前端生态系统。许多流行的前端框架和库，如React Native、Flutter和Ionic，都支持跨端开发。这些工具为开发者提供了丰富的资源和丰富的社区支持，帮助他们更轻松地实现跨端功能。

<img src=".\byte-images\image-20230419152521218.png" alt="image-20230419152521218" style="zoom:50%;" />

### 跨端方案

- webview
- 小程序
- RN/WeeX
- Lynx
- Flutter

<img src=".\byte-images\image-20230419211106710-1681914418588-31-1681914814540-179.png" alt="image-20230419211106710" style="zoom:50%;" />

#### 跨端容器 - WebView

> 跨端容器中的WebView是一种在移动应用中嵌入网页内容的组件。WebView允许开发者将HTML、CSS和JavaScript等Web技术直接嵌入到移动应用中，从而实现跨平台的应用开发。通过WebView，可以在原生应用中展示网页内容，同时为开发者提供了一些与原生功能交互的能力
>
> WebView的主要优点包括：
>
> 1. 跨平台兼容性：使用WebView，开发者可以利用Web技术为多个平台（如iOS和Android）创建应用。这可以节省开发时间和成本，一次开发处处使用，学习成本低，同时确保应用在各个平台上提供一致的用户体验。
> 2. 代码重用：WebView允许开发者在移动应用中重用现有的Web代码。这意味着，对于已有Web应用的公司来说，可以更容易地将其产品扩展到移动平台。
> 3. 简化更新过程：随时发布，即时更新，不用下载安装包。WebView使得应用内容的更新变得更加简单。因为WebView直接加载Web内容，开发者可以在服务器端更新应用内容，而无需重新提交整个应用到应用商店进行审核。
> 4. 通过JSBridge和原生系统交互，能够实现一些更加复杂的功能
>
> 然而，WebView也有一些局限性：
>
> 1. 性能：相比于原生应用，基于WebView的应用性能可能较差。这主要是因为WebView需要加载和运行Web内容，这会消耗更多的系统资源。在某些情况下，这可能导致较慢的加载速度和不流畅的用户体验。
> 2. 原生功能访问限制：虽然WebView提供了一些与原生功能交互的能力，但它仍然受到一定的限制。为了实现某些特定的原生功能，开发者可能需要编写额外的平台特定代码。
> 3. 用户体验差异：尽管WebView可以确保跨平台的一致性，但它可能无法完全符合不同平台的设计规范。因此，在某些情况下，基于WebView的应用可能无法提供与原生应用相同水平的用户体验。

1. Webview,即网页视图，用于加载网页Url,并展示其内容的控件
2. 可以内嵌在移动端App内，实现前端混合开发，大多数混合框架都是基于Webview的二次开发；比如lonic、Cordova

#### 跨端容器 - 常用WebView分类

- 常用webview、Android，IOS，国产Android

> 1. WebView（Android）：这是Android平台的原生WebView组件，用于在Android应用中加载并显示Web内容。根据Android版本和设备制造商的不同，WebView的表现可能会有所差异。这可能导致一些兼容性和性能问题。
> 2. X5 WebView（腾讯X5内核）：这是由腾讯公司推出的一种WebView解决方案，用于解决Android系统上WebView的碎片化问题。X5内核基于腾讯QQ浏览器的内核，提供了更稳定、更高性能的WebView组件。它可以在各种Android设备和系统版本上提供一致的表现，减少兼容性问题。
> 3. UIWebView（iOS，已弃用）：这是iOS平台的原生WebView组件，用于在iOS应用中加载并显示Web内容。UIWebView自iOS 2.0开始引入，但在iOS 8.0中被WKWebView取代。自iOS 12.0以来，UIWebView已被官方弃用，不再推荐使用。
> 4. WKWebView（iOS）：这是iOS平台上的新一代WebView组件，取代了已弃用的UIWebView。WKWebView自iOS 8.0开始引入，它具有更好的性能和更丰富的功能，如支持多进程、JavaScript性能改进等。苹果公司推荐开发者使用WKWebView来在iOS应用中加载并显示Web内容。

|  platform(平台)  | webview(组件) |      内核       |
| :--------------: | :-----------: | :-------------: |
|    Android4.4    |    webview    |     Webkit      |
|  Android4.4以上  |    webview    |    chromium     |
| Android 国内部分 |  X5 webview   | chromium 改造版 |
|       IOS        |   UIwebview   |     Webkit      |
|    IOS8 以上     |   WKwebview   |     Webkit      |

#### 跨端容器 -WebView使用原生能力

> Javascript调用Native

- API注入：Native获取]avascript3环境上下文，对其挂载的对象或者方法进行拦截
- 使用Webview URL Scheme跳转拦截
- IOS上window.webkit.messageHandler直接通信

> Native调用Javascript

- 直接通过webview暴露的API执行JS代码
- IOS webview.stringByEvaluatingJavaScriptFromString
- Android webview.evaluateJavascript

### 跨端容器 - WebView< - >Nactive 通信

1. JS环境中提供通信的 JSBridge
2. Native端提供 SDK 响应 JSBridge 发出的调用
3. 前端和客户端分别实现对应功能模块

<img src=".\byte-images\image-20230419213423384-1681914418588-32-1681914814540-180.png" style="zoom:50%;" />

### 跨端容器 - 实现一个简易JSBridge

> JSBridge是一种在WebView中实现原生与JavaScript之间通信的技术

![image-20230419213529716](.\byte-images\image-20230419213529716-1681914418588-33-1681914814540-181.png)

```js
// 创建一个空对象，用于存储回调函数
const callbacks = {};

// 生成唯一ID的函数，用于标识每个回调函数
function generateUniqueId() {
  return 'callback_' + Date.now() + '_' + Math.random().toString(36).substr(2, 6);
}

// 调用原生方法的函数
function callNative(action, params, callback) {
  // 生成唯一ID
  const callbackId = generateUniqueId();
  
  // 将回调函数存储到callbacks对象中
  callbacks[callbackId] = callback;
  
  // 创建一个包含操作、参数和回调ID的对象，用于发送给原生端
  const message = {
    action,
    params,
    callbackId,
  };
  
  // 将对象转换为JSON字符串
  const messageStr = JSON.stringify(message);
  
  // 调用原生方法（具体实现取决于平台和环境）
  // 在Android中，通常会使用JavaScriptInterface
  // 在iOS中，通常会使用window.webkit.messageHandlers.<handlerName>.postMessage
  window.AndroidBridge.postMessage(messageStr);
}

// 原生调用JS的函数
function nativeCallback(callbackId, result) {
  // 从callbacks对象中获取对应的回调函数
  const callback = callbacks[callbackId];
  
  // 判断回调函数是否存在
  if (callback) {
    // 调用回调函数并传入结果参数
    callback(result);
    
    // 从callbacks对象中删除已经调用过的回调函数
    delete callbacks[callbackId];
  }
}

// 将方法暴露给全局对象，以便原生代码调用
window.JSBridge = {
  callNative,
  nativeCallback,
};

//代码块内容由ChatGPT4给出
```

### 跨端容器 - 小程序

1. 微信、支付宝、百度小程序、小米直达号
2. 渲染层-webview
3. 双线程，多webview架构
4. 数据通信，Native转发

<img src=".\byte-images\image-20230419214204583-1681914418588-35-1681914814540-182.png" alt="image-20230419214204583" style="zoom:50%;" />

### 跨端容器 - React Native/WeeX

1. 原生组件渲染
2. React/Vue框架
3. virtual dom
4. JSBridge

> 1. React Native： React Native是Facebook开发的一个开源框架，它允许开发者使用React和JavaScript编写原生移动应用。React Native的主要特点是"Learn once, write anywhere"（学习一次，随处编写），这意味着开发者可以在不同平台（如iOS和Android）上使用相同的技术栈构建原生应用。
>
> React Native的优势包括：
>
> - 代码复用：React Native允许开发者在多个平台上复用大部分代码，从而减少开发时间和成本。
> - 热重载：React Native支持热重载功能，允许开发者在不重新编译整个应用的情况下查看代码更改的效果，提高开发效率。
> - 原生性能：虽然React Native使用JavaScript编写，但它将JavaScript代码转换为原生组件，从而提供了接近原生应用的性能。
> - 丰富的生态系统：React Native拥有庞大的社区支持，开发者可以利用许多第三方库和组件来加速开发过程。
>
> 1. Weex： Weex是由阿里巴巴开发的一个开源框架，它允许开发者使用Vue.js和JavaScript编写原生移动应用。Weex的目标是实现"Write once, run everywhere"（编写一次，随处运行），即使用一套代码构建多个平台的原生应用。
>
> Weex的优势包括：
>
> - 代码复用：类似于React Native，Weex也支持在不同平台上复用代码，从而降低开发成本。
> - 原生渲染：Weex将Vue.js组件转换为原生组件进行渲染，从而实现了较高的性能。
> - 插件系统：Weex提供了一个丰富的插件系统，开发者可以使用这些插件来轻松实现原生功能，如地图、支付等。
> - 模块化：Weex允许开发者将应用拆分为多个模块，这有助于实现高度模块化的开发过程，提高代码可维护性。

<img src=".\byte-images\image-20230419214458198-1681914418588-34-1681914814540-183.png" alt="image-20230419214458198" style="zoom:50%;" />

### 跨端容器 - Lynx

1. 基于Vue框架
   - Lynx采用Vue.js作为其基础框架，允许开发者使用Vue.js编写应用。Vue.js是一种渐进式JavaScript框架，它使开发者能够轻松地构建可扩展、高性能的应用。基于Vue.js的设计原则，Lynx可以提供简洁、模块化的代码结构，以及良好的开发体验
2. 绑定于JS Core / V8
   - Lynx选择JS Core（iOS平台）或V8（Android平台）作为其JavaScript引擎。这意味着Lynx应用在运行时，JavaScript代码将在高性能的JavaScript引擎中执行。通过使用这些优秀的JavaScript引擎，Lynx能够确保应用在不同平台上具有稳定、高性能的运行表现
3. JSBinding
   - Lynx使用JSBinding技术实现JavaScript与原生代码之间的通信。这种技术允许JavaScript直接调用原生方法，并使原生代码能够执行JavaScript回调。通过JSBinding，Lynx实现了高效的原生与JavaScript之间的通信，降低了性能损失
4. Native UI /Skia
   - Lynx使用Native UI组件和Skia作为其渲染引擎。Native UI组件意味着Lynx应用在运行时，界面将使用原生组件进行渲染。这可以确保应用具有接近原生应用的性能和用户体验。同时，Lynx还采用了Skia图形库，它是一种高性能的2D图形渲染引擎，用于绘制图形和文本。Skia使Lynx应用在渲染复杂界面时能够保持流畅的帧率和高质量的视觉效果

<img src=".\byte-images\image-20230419215126209-1681914418588-36-1681914814540-185.png" alt="image-20230419215126209" style="zoom:50%;" />

### 跨端容器 - Flutter

> Flutter是Google开发的一个开源UI工具包，旨在为开发者提供一种构建优美、高性能的跨平台应用的解决方案。Flutter具有一些独特的特点，包括基于Widget的设计、Dart VM以及使用Skia图形库

1. wideget
   - 在Flutter中，所有UI元素都被称为Widget。Widget是Flutter应用的基本构建块，它们可以嵌套、组合以及自定义，从而创建复杂的用户界面。Flutter提供了丰富的预制Widget，如文本、按钮、列表等，开发者可以直接使用这些Widget，也可以通过组合和扩展它们来构建自定义的Widget
2. dart vm
   - Flutter使用Dart语言进行开发，Dart是一种强类型、面向对象的编程语言，它既可以编译成JavaScript代码（用于Web应用），也可以编译成机器码（用于移动应用）
   - 在移动端，Flutter应用运行在Dart VM（虚拟机）中。Dart VM提供了即时编译（JIT）和预编译（AOT）两种编译方式。在开发过程中，Dart VM采用即时编译，这使得Flutter具有热重载功能，开发者可以在不重新编译整个应用的情况下查看代码更改的效果。在发布应用时，Dart VM会采用预编译，将Dart代码编译成高效的机器码，以提高应用的性能
3. skia图形库
   - Flutter使用Skia图形库进行UI渲染。Skia是一种高性能的2D图形渲染引擎，用于绘制图形和文本。由于Flutter直接使用Skia进行渲染，它无需依赖于原生UI组件，可以实现统一的跨平台UI渲染。这使得Flutter应用具有高度的可定制性，同时还保持了流畅的性能和优美的视觉效果。

<img src=".\byte-images\image-20230419215358209-1681914418588-39-1681914814540-184.png" alt="image-20230419215358209" style="zoom:50%;" />

### 跨端容器 - 通用原理

1. UI组件
2. 渲染引擎
3. 逻辑控制引擎
4. 通信桥梁
5. 底层API抹平表面差异

<img src=".\byte-images\image-20230419215524843-1681914418588-37-1681914814540-187.png" alt="image-20230419215524843" style="zoom:50%;" />

## 跨端方案对比

![image-20230419215633320](.\byte-images\image-20230419215633320-1681914418588-38-1681914814540-186.png)

## 跨端思考

1. 同样是基于webview渲染，为什么小程序体验比webview流畅？

这主要归功于小程序在架构和优化方面的一些特点。以下是一些关键因素：

1. 优化的渲染引擎： 小程序的渲染引擎经过针对性优化，以适应移动设备的性能特点。这使得小程序在处理动画、滚动等界面交互时更加流畅。此外，小程序还采用了一些高效的Web渲染技术，如硬件加速、层合成等，从而提高了渲染性能。
2. 分离的逻辑层和渲染层： 小程序将逻辑层（JavaScript代码）和渲染层（UI界面）分开处理。逻辑层运行在Web容器中，而渲染层则运行在一个独立的原生视图中。通过这种分离，小程序可以在不影响UI渲染的情况下执行JavaScript代码，从而提高了应用的流畅度。
3. 限制和优化的API： 小程序提供了一组限制和优化的API，这些API专门针对移动设备的性能和用户体验进行了调整。通过使用这些API，开发者可以更高效地编写适用于移动设备的应用，从而提高应用的流畅度。
4. 资源限制： 小程序对资源的大小和数量有严格的限制，这迫使开发者在设计应用时充分考虑性能优化。较小的资源包意味着更快的加载速度，从而提高了应用的启动速度和运行性能。
5. 预加载和缓存机制： 小程序具有预加载和缓存机制，这使得应用在首次启动时可以快速加载和渲染，从而提高了用户体验。此外，缓存机制还可以减少网络请求和数据传输，降低应用的延迟。

总之，虽然小程序和基于WebView的应用都是通过WebView渲染，但小程序在渲染引擎、架构设计、API、资源限制以及预加载和缓存等方面进行了针对性优化，这些优化使得小程序的体验比WebView应用更流畅。

2. 未来的跨端方案会是什么？

> ....

# 课程总结

![image-20230419221152234](.\byte-images\image-20230419221152234-1681914418588-40-1681914814545-188.png)