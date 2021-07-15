---
sidebar_position: 1
---

https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path

web 性能包含服务端请求和响应、资源加载、脚本解析、rendering, layout, 和屏幕上像素的 painting。

网页的请求开始于 HTML 请求。服务器返回后：
1. 浏览器开始解析 HTML，将接受到的字节转换成 DOM 树。浏览器不断解析 HTML、发送请求、构建 DOM，直至 HTML 末尾。
    + 每次遇到样式表、脚本或嵌入图片的引用等外部资源的链接时都会发起请求。一些请求是阻塞的， HTML剩余部分的解析会被阻塞直到引入的资源处理完成。
2. 浏览器会构建 CSS 对象模型。
3. DOM 和 CSSOM 都构建完成后，浏览器构建出渲染树计算出所有可见内容的样式。
4. 进行layout， 确定渲染树中所有元素的位置和大小。
5. 页面将会被绘制在屏幕上

### Document Object Model

DOM 的构建是增量的。HTML 响应被转换成 DOM nodes，然后构成 DOM 树。节点的数量越多，一下 crp 中的事件所花费的事件越长。Measure! A few extra nodes won't make a difference, but divitis can lead to jank.

### CSS Object Model

DOM 包含了页面的内容， CSSOM 包含了页面的所有样式，即有关如何为 DOM 添加样式的信息。 DOM 的构建是增量的，但 CSSOM 不是，它是渲染阻塞的: 在接收到并处理完所有的 CSS 之前，浏览器会阻塞页面的渲染。因为 CSS 规则会被覆盖，所以 CSSOM 构建完成之前页面内容不会被渲染。CSS 有它自己的一套识别有效的 tokens 的规则。C 代表 'Cascade'。 CSS 规则会向下层叠：即解析器将 tokens 转换成节点时，其子孙节点会继承样式。正因为之后的规则可能覆盖之前的，所以 HTML 那样增量处理的特点不适用于 CSS。

#### selector performance

不太明确的选择器比明确的快。因为更明确的规则需要遍历 DOM 树中更多的节点 如 `.foo {}`比 `.bar .foo {}`快，因为第二中情况下浏览器找到 .foo 时，还需要检查 .foo 是否有 .bar 的祖先元素。但CSS的选择器 penalty 不怎么值得优化。针对其的性能优化可能只有微秒级的提升空间。

优化 CSS 的方法有：
+ minification
+ separating deferred CSS into non-blocking requests by using media queries.

https://developer.mozilla.org/en-US/docs/Learn/Performance/CSS


### Render Tree

渲染树捕获了内容和样式: DOM 树 CSSOM 树结合成为渲染树。浏览器从 DOM 树的根节点开始检查每个节点，确定应用（attached）哪些 CSS 规则应用到各个节点上。渲染树只捕捉可见的内容 The head section (generally) 不包含可见信息，于是不被包含在渲染树中如果某个元素用 display:none 这个规则，则其和其子节点都不在渲染树中。

### Layout
Layout is dependent on the size of screen.  layout 这个步骤决定元素在页面上的位置、宽高，以及元素之间的相互关系。
What is the width of an element? Block level elements, by definition, have a default width of 100% of the width of their parent. An element with a width of 50%, will be half of the width of its parent. Unless otherwise defined, body 宽度为 100%，这意味这其占据视口宽度的 100% 。This width of the device impacts layout.

The viewport meta tag defines the width of the layout viewport, impacting the layout. Without it, the browser uses the default viewport width, which on by-default full screen browsers is generally 960px. On by-default full screen browsers, like your phone's browser, by setting `<meta name="viewport" content="width=device-width">`, the width will be the width of the device instead of the default viewport width. The device-width changes when a user rotates their phone between landscape and portrait mode. Layout happens every time a device is rotated or browser is otherwise resized.

Layout 的性能受 DOM 影响 -- 节点数量越多， layout 花费的时间越长。滚动和其他动画时需要进行 Layout 可能成为性能瓶颈 导致 jank 。 While a 20ms delay on load or orientation change may be fine, it will lead to jank on animation or scroll. 每当渲染树被通过诸如 添加节点、改变内容，或更新节点上的 box model styles 的方式修改时 ，layout 就会发生。
为了减小 layout 事件的频率和 duration，应该进行批量更新，并避免对`盒模型属性`应用动画。


### Paint
渲染树构建完成，并进行了 layout 之后， the pixels 就能被 painted to the screen. 加载时，整个屏幕被绘制，此后只有受影响的屏幕区域会被重绘，因为浏览器被优化到只重回所需的最小区域。paint 时间取决于何种更新应用到了渲染树While painting is 是非常快的步骤，不太可能成为性能优化中最值得关注的影响最大的地方，但 it is important to remember to allow for both layout and re-paint times when measuring how long an animation frame may take. The styles applied to each node increase the paint time, but removing style that increases the paint by 0.001ms may not give you the biggest bang for your optimization buck. Remember to measure first. Then you can determine whether it should be an optimization priority.

### Optimizing for CRP
Improve page load speed by prioritizing which resources get loaded, controlling the order in which they are loaded, and reducing the file sizes of those resources. Performance tips include 
1. minimizing the number of critical resources by deferring their download, marking them as async, or eliminating them altogether
2. optimizing the number of requests required along with the file size of each request
3. optimizing the order in which critical resources are loaded by prioritizing the downloading critical assets, shorten the critical path length.