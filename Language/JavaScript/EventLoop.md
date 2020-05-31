javascript是一门单线程语言，它的缺点是任务执行会阻塞，表现到网页里是：发起数据请求 --> http延迟 --> 等待完成，等待的过程中，其他操作无法执行，导致页面长时间无响应。我们会用**异步回调（asynchronous callback）**去解决这些问题。实现异步回调的特性，其实是基于Event Loop（事件循环）。

### 一、JavaScript的 Engine 和 Runtime

简单来说，为了让 JavaScript 运行起来，要完成两部分工作（当然实际比这复杂得多）：

- Engine（JS引擎）：编译并执行 JavaScript 代码，完成内存分配、垃圾回收等；
- Runtime（运行时）：为 JavaScript 提供一些对象或机制，使它能够与外界交互。

Chrome浏览器 和 Node.js 都使用了 V8 Engine。V8 实现并提供了 ECMAScript 标准中的所有数据类型、操作符、对象和方法（注意并没有 DOM）。 但它们的 Runtime 并不一样：Chrome 提供了 window、DOM，而 Node.js 则是 require、process 等等。

### 二、浏览器的线程

现代浏览器的一个 tab ，其中的线程包括但不局限于：

- GUI 渲染线程
- JS引擎线程
- 事件触发线程
- 定时器触发线程
- 异步http请求线程

JavaScript中的异步回调是通过 WebAPIs 去支持的，常见的有 XMLHttpRequest，setTimeout，事件回调（onclik, onscroll等）。而这几个 API 浏览器都提供了单独的线程去运行，所以才会有合适的地方去处理定时器的计时、各种请求的回调。即当代码中出现这几个定义的异步任务，是由浏览器实现了它们与JS引擎的通信，与JS引擎不属与同一个线程。

另外，GUI 渲染和JavaScript执行是互斥的。虽然两者属于不同的线程，但是由于JavaScript执行结果可能会对页面产生影响，所以浏览器对此做了处理，大部分情况下JavaScript线程执行，执行渲染（render）的线程就会暂停，等JavaScript的同步代码执行完再去渲染。

### 三、Event loop的定义

Event Loop（事件循环） 是让 JavaScript 做到既是单线程，又绝对不会阻塞的核心机制，也是 JavaScript 并发模型（Concurrency Model）的基础，是用来协调各种事件、用户交互、脚本执行、UI 渲染、网络请求等的一种机制。Event Loop的作用很简单： 监控调用栈和任务队列，如果调用栈是空的，它就会取出队列中的第一个"callback函数"，然后将它压入到调用栈中，然后执行它。

总的来说，Event Loop 是实现异步回调的一种机制而已。

#### 3.1 Event Loop 分两种

Event Loop 分为两种，一种存在于 Browsing Context 中，还有一种在 Worker 中。

1. Browsing Context 是指一种用来将 Document（文档）展现给用户的环境。例如浏览器中的 tab，window 或 iframe 等，通常都包含 Browsing Context。
2. Worker 是指一种独立于 UI 脚本，可在后台执行脚本的 API。常用来在后台处理一些计算密集型的任务。

*Event Loop 并不是在 ECMAScript 标准中定义的，而是在 HTML 标准中定义的。在 JS引擎中（以V8为例），只是实现了 ECMAScript 标准，而并不关心什么 Event Loop。也就是说 Event Loop 是属于 JavaScript Runtime 的，是由宿主环境（比如浏览器）提供的。*

#### 3.2 独立

每个”线程“都有自己的 Event Loop。所以，每个 web worker 拥有独立的 Event Loop，它们都可以独立运行；同源的 windows 共享一个 Event Loop，它们之间可以互相通信。

### 四、内存模型

从 JavaScript 内存模型的角度，我们可以将内存划分为调用栈（Call Stack）、堆（Heap）以及任务队列（Queue）等几个部分：

#### 4.1 调用栈

调用栈会记录所有的函数调用信息，当我们调用某个函数时，会将其参数与局部变量等以栈帧的形式压入栈中（入栈）；在执行完毕后，会弹出栈顶的帧。

#### 4.2 堆

堆则则存放了大量的非结构化数据，譬如程序分配的变量与对象。

#### 4.3 任务队列

任务队列包含了一系列待处理的信息与相关联的回调函数。任务队列又分为 MacroTask Queue 和 MicroTask Queue 两种。

##### 4.3.1 MacroTask Queue（宏任务队列）

Event Loop 会有一个或多个 MacroTask Queue，这是一个先进先出（FIFO）的有序列表，存放着来自不同 Task Source（任务源）的 Task（也即MacroTask）。

```
关于 Task，常有人通俗地称它为 MarcoTask，但其实 HTML 标准中并没有这种说法。然而，为了方便理解，本书仍沿用通俗的称谓MacroTask。
```

在 HTML 标准中，定义了几种常见的 Task Source：

- DOM manipulation（DOM 操作）；
- User interaction（用户交互）；
- Networking（网络请求）；
- History traversal（History API 操作）；

MacroTask Source 的定义非常的宽泛，常见的鼠标、键盘事件，AJAX，数据库操作（例如 IndexedDB），以及定时器相关的 setTimeout、setInterval 等等都属于 Task Source，所有来自这些 MacroTask Source 的 MacroTask 都会被放到对应的 MacroTask Queue 中等待处理。

对于 MacroTask、MacroTask Queue 和 Task Source，有如下规定：


1. 来自相同 Task Source 的 MacroTask，必须放在同一个 MacroTask Queue 中；
2. 来自不同 Task Source 的 MacroTask，可以放在不同的 MacroTask Queue 中；
3. 同一个 MacroTask Queue 内的 MacroTask 是按顺序执行的；
4. 但对于不同的 MacroTask Queue（Task Source），浏览器会进行调度，允许优先执行来自特定 Task Source 的 MacroTask。

```
例如，鼠标、键盘事件和网络请求都有各自的 MacroTask Queue，当两者同时存时，浏览器可以优先从用户交互相关的 MacroTask Queue 中挑选 MacroTask 并执行，比如这里的鼠标、键盘事件，从而保证流畅的用户体验。
```

##### 4.3.2 MicroTask Queue（微任务队列）

MicroTask Queue 与 MacroTask Queue 类似，也是一个有序列表。不同之处在于，一个 Event Loop 只有一个 MicroTask Queue。

在 HTML 标准中，并没有明确规定 MicroTask Source，通常认为有以下几种：

- Promise
- MutationObserver
- Object.observe (已废弃)

*在 Promises/A+ Note 3.1 中提到了 then、onFulfilled、onRejected 的实现方法，但 Promise 本身属于平台代码，由具体实现来决定是否使用 Microtask，因此在不同浏览器上可能会出现执行顺序不一致的问题。不过好在目前的共识是用 Microtask 来实现事件队列。*

*Node.js 的 process.nextTick 和 Microtask ，虽然两者层级（运行时机）非常接近，但并不是同一个东西。process.nextTick 是 Node.js 自身定义实现的一种机制，有自己的 nextTickQueue，与 HTML 标准中的 MicroTask 不是一回事。在 Node.js 中，process.nextTick 会先于 Microtask Queue 被执行。*

##### 4.3.3 二者关系图例

Event Loop中，每一次循环称为tick，每一次tick的任务细节如下：

- 调用栈选择最先进入队列的MacroTask（通常是script整体代码），如果有则执行；
- 检查是否存在 MicroTask，如果存在则不停的执行，直至清空 MicroTask Queue；
- 浏览器更新渲染（render），每一次事件循环，浏览器都可能会去更新渲染；
- 重复以上步骤；

![MacroTask Queue和MicroTask Queue](./images/010.png)

如图所示，二者互相穿插：MacroTask --> MicroTask Queue --> MacroTask。

**一个Event Loop会有一个或多个 MacroTask Queue，而 Event Loop 仅有一个 MicroTask Queue。**

每个 MacroTask Queue 都保证按照回调函数（callback）入队列的顺序依次执行MacroTask，在 MacroTask 或者 MicroTask 中产生的新 MicroTask 会被压入到 MicroTask Queue中并执行。而在 执行两个MacroTask之间，也即在执行下一个MacroTask之前，会优先执行完所有MicroTask，也即会优先清空已有的 MicroTask Queue。

因此，图中第二个MicroTask Queue产生的时候，第一个MicroTask Queue其实已经被清空了。所以Event Loop实际上仅有一个MicroTask Queue。

### 五、JavaScript Runtime 的运行机制

从相对宏观的角度了解一下 JavaScript Runtime 的运行机制：

1. 主线程不断循环；
2. 对于同步任务，创建执行上下文（Execution Context），按顺序进入调用栈；
3. 对于异步任务：
    - 与步骤 2 相同，同步执行这段代码；
    - 将相应的 MacroTask（或 Microtask）添加到任务队列；
    - 由其他线程来执行具体的异步操作。
    ```
    其他线程是指：尽管 JavaScript 是单线程的，但浏览器内核是多线程的，它会将 GUI 渲染、定时器触发、HTTP 请求等工作交给专门的线程来处理。另外，在 Node.js 中，异步操作会优先由 OS 或第三方系统提供的异步接口来执行，然后才由线程池处理。
    ```
4. 当主线程执行完当前调用栈中的所有任务，就会去读取 Event Loop 的任务队列，取出并执行任务；
5. 重复以上步骤。

### 参考

- https://juejin.im/post/59e85eebf265da430d571f89
- https://github.com/coffe1891/frontend-hard-mode-interview/blob/master/1/1.2.8.md
- https://juejin.im/post/5b6d58fee51d45194b195544