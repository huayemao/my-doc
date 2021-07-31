# 事件循环

https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide

### 微任务：

- process.nextTick
- MutationObserver
- Promise.then catch finally

### 宏任务：

- I/O
- setTimeout
- setInterval
- setImmediate
- requestAnimationFrame

### Event Loop

浏览器和 Node.js 中的 JavaScript 执行流都是基于事件循环的。事件循环 是 JS 引擎等待任务，执行任务和休眠，等待更多任务的无限循环

引擎的大致算法：

1. 当有任务时：

   - 从最先进入的任务开始执行。

2. 休眠直到出现任务，然后转到第 1 步。

以上就是我们浏览页面时看到的东西的形式化描述。JS 引擎大多数时候不执行任何操作，它仅在脚本/处理程序/事件激活时执行。

任务的例子:

- 外部脚本 `<script src="...">` 加载时执行脚本.
- 用户移动过鼠标时，派发 mousemove 鼠标事件并执行事件处理程序
- setTimeout 排期的事件到时，执行其回调
- 等等……

任务被设定 —— 引擎处理任务 —— 等待更多任务（此时引擎会进入休眠，几乎不消耗 CPU 资源）。

一个任务到来时，引擎可能正处于繁忙状态，那么这个任务就会被排入队列。多个任务组成了一个队列，即所谓的“宏任务队列”（v8 术语）：

例如，当引擎正在忙于执行一段 script 时，用户可能会移动鼠标而产生 mousemove 事件，setTimeout 或许也刚好到期，还可能会有其他任务，这些任务组成了一个队列。队列中的任务按照 “先来先服务” 的原则处理。当浏览器引擎执行完 script 后，它会处理 mousemove 事件，然后处理 setTimeout 处理程序，依此类推。

两个细节：

- 引擎执行任务时永远不会进行渲染。即使任务执行需要很长时间。只有任务完成后才会绘制对 DOM 的更改。
- 如果一项任务执行花费的时间过长，浏览器将无法执行其他任务，例如处理用户事件。因此，在一定时间后，浏览器会弹出 “页面未响应” 这样的警告框，建议用户终止这个任务。这发生在有大量复杂的计算或导致死循环的程序错误时。

### 宏任务和微任务

微任务仅来自于代码，通常由 promise 创建：对 .then/catch/finally 处理程序的执行会成为微任务。 await 的背后也有用到微任务，因为它是 promise 处理的另一种形式。还有一个特殊的函数 queueMicrotask(func)，可以将 func 入队到微任务队列中执行。

每个宏任务之后，引擎会在执行其他的宏任务、渲染或进行其他任何操作之前立即执行微任务队列中的所有任务。

示例：

```js
setTimeout(() => alert("timeout"));

Promise.resolve().then(() => alert("promise"));

alert("code");
```

执行顺序：

1. code 首先显示，因为它是常规的同步调用。
2. promise 第二个出现，因为 then 会通过微任务队列，并在当前代码之后执行。
3. timeout 最后显示，因为它是一个宏任务。

更详细的事件循环图示如下（顺序是从上到下，即：首先是脚本，然后是微任务，渲染等）：

微任务会在执行任何其他事件处理、渲染及执行任何其他宏任务之前完成。这确保了微任务之间的应用程序环境基本相同（没有鼠标坐标更改，没有新的网络数据等）。

> 如果想要异步执行一个函数，但是要在更改被渲染或新事件被处理之前执行，那么我们可以使用 queueMicrotask 来对其进行安排（schedule）。

这是一个与前面那个例子类似的，带有“计数进度条”的示例，但是它使用了 queueMicrotask 而不是 setTimeout。你可以看到它在最后才渲染。就像写的是同步代码一样：

### 总结

更详细的事件循环算法：

1. 从宏任务队列（例如“script”）中出队并执行最早的任务。
2. 执行所有微任务：
   - 当微任务队列非空时：出队并执行最早的微任务。
3. 执行渲染。
4. 如果宏任务队列为空，则休眠直到出现宏任务。
5. 转到步骤 1。

排期（schedule）一个新的宏任务：

- 使用零延迟的 setTimeout(f)：
  - 用处：可将繁重的计算任务拆分成多个部分，以使浏览器能够对用户事件作出反应，并在任务的各部分之间显示任务进度。
  - 此外，也被用于在事件处理程序中，将一个动作（action）安排（schedule）在事件被完全处理（冒泡完成）后。

排期一个新的微任务：

- 使用 queueMicrotask(f)。
- promise 处理程序也会通过微任务队列。

在微任务之间没有 UI 或网络事件的处理：它们一个立即接一个地执行。所以，我们可以使用 queueMicrotask 来在保持环境状态一致的情况下，异步地执行一个函数。

## 并发模型与事件循环

不同于比如 C 和 Java 等语言，JS 的并发模型是基于事件循环的。事件循环负责执行代码、收集和处理事件以及执行队列中的子任务。

### 运行时概念

#### 堆

对象被分配在堆中，堆是用来表示一大块（通常是非结构化的）内存区域。

#### 队列

JavaScript 运行时维护一个待处理消息的消息队列。每一个消息都关联了一个用以处理这个消息的回调函数。在事件循环的某个时刻，运行时会将消息队列中队首的消息出队，并作为输入参数来调用与之关联的函数。为其创造一个新的栈帧，处理函数，直到执行栈再次为空为止；然后事件循环将会处理队列中的下一个消息（如果还有的话）。

### 事件循环

事件循环之所以称之为事件循环，是因为它经常按照类似如下的方式来被实现：

```js
// queue.waitForMessage() 会同步地等待消息到达(如果当前没有任何消息等待被处理)
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

每个浏览器标签页都运行在单独的进程中——事件循环中。这个循环执行浏览器相关的事物，即所谓的任务。任务通过任务队列来输送。一些任务的例子：

- Parsing HTML
- Executing JavaScript code in script elements
- Reacting to user input (mouse clicks, key presses, etc.)
- Processing the result of an asynchronous network request

Items 2–4 are tasks that run JavaScript code, via the engine built into the browser. They terminate when the code terminates. Then the next task from the queue can be executed. The following diagram (inspired by a slide by Philip Roberts [1]) gives an overview of how all these mechanisms are connected.

The event loop is surrounded by other processes running in parallel to it (timers, input handling, etc.). These processes communicate with it by adding tasks to its queue.

#### "run-to-completion"

不会造成竞争条件

https://exploringjs.com/es6/ch_async.html#sec_browser-event-loop

JavaScript 拥有所谓 run-to-completion 的特性: 当前任务总会在下一个任务执行之前完成，这意味着每个任务都对当前状态有完全的控制权，不用担心并发修改（concurrent modification）的问题

#### Blocking the event loop

As we have seen, each tab (in some browers, the complete browser) is managed by a single process – both the user interface and all other computations. That means that you can freeze the user interface by performing a long-running computation in that process. The following code demonstrates that.

You avoid blocking the event loop in two ways:

First, you don’t perform long-running computations in the main process, you move them to a different process. This can be achieved via the Worker API.

Second, you don’t (synchronously) wait for the results of a long-running computation (your own algorithm in a Worker process, a network request, etc.), you carry on with the event loop and let the computation notify you when it is finished. In fact, you usually don’t even have a choice in browsers and have to do things this way. For example, there is no built-in way to sleep synchronously (like the previously implemented sleep()). Instead, setTimeout() lets you sleep asynchronously.

The next section explains techniques for waiting asynchronously for results.

这个模型的一个缺点在于当消息需要很长时间才能处理完时，Web 应用程序就无法处理与用户的交互，例如点击或滚动。为了缓解这个问题，浏览器一般会弹出一个“a script is taking too long to run”的对话框。一个良好的习惯是缩短单个消息处理时间，并在可能的情况下将一个消息拆分成多个消息。

#### 添加消息

在浏览器中，每当一个已绑定监听器的事件发生时，消息队列中就会添加一个消息。如果没有事件监听器，这个事件将会丢失。

setTimeout 这个函数接受两个参数：待加入队列的消息和一个时间值（可选，默认为 0）。这个时间值代表了消息被实际加入到队列的最小延迟时间。如果队列中没有其它消息并且栈为空，在这段延迟时间过去之后，消息会被马上处理。但是，如果有其它消息，setTimeout 消息必须等待其它消息处理完。因此第二个参数仅仅表示最少延迟时间，而非确切的等待时间。

Browsers have timers. setTimeout() creates a timer, waits until it fires and then adds a task to the queue. It has the signature:

setTimeout(callback, ms)
After ms milliseconds, callback is added to the task queue. It is important to note that ms only specifies when the callback is added, not when it actually executed. That may happen much later, especially if the event loop is blocked (as demonstrated later in this chapter).

setTimeout() with ms set to zero is a commonly used work-around to add something to the task queue right away. However, some browsers do not allow ms to be below a minimum (4 ms in Firefox); they set it to that minimum if it is.

#### 多个运行时互相通信

web worker 和跨域的 iframe 都有自己的栈、堆和消息队列。两个不同的运行时只能通过 postMessage 方法进行通信。如果另一个运行时监听了 message 事件，则此方法会向该运行时添加消息。

#### 永不阻塞

JavaScript 的事件循环模型拥有永不阻塞的特性。 处理 I/O 通常通过事件和回调来执行，应用在等待 IndexedDB 查询返回或者 XHR 请求返回的过程中，仍然可以处理其它事情，比如用户输入。由于历史原因有一些例外，如 alert 或者同步 XHR，但应该尽量避免使用它们。
