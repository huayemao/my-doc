---
sidebar_position: 1
---

# Reconciliation

## 生命周期与两个 phase

从概念上讲，react 的工作可以分为两个阶段：

- `The render phase` 决定什么变化需要应用到例如 DOM 上. 在这个阶段，React 调用 render，然后将结果与之前的 render 进行比较。
- `The commit phase` 是 React 应用变化的阶段。 (以 React DOM 来说, 这是 React 插入、更新和移除 DOM 节点的阶段)，在这一阶段 React 也会调用 componentDidMount 和 componentDidUpdate 这样的生命周期方法.

<iframe src="https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/"
        width="100%" height="800" frameborder="0" loading="lazy"
        allowfullscreen sandbox>
</iframe>

- https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/
- https://reactjs.org/docs/glossary.html#lifecycle-methods
- https://reactjs.org/docs/react-component.html#mounting
- https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html
- https://reactjs.org/docs/strict-mode.html#detecting-unexpected-side-effects
- https://techdoma.in/react-16-tutorial/what-are-render-phase-and-commit-phase-in-react

## 设计动机

使用 React 时，可以认为 render() 方法在某一时间节点会创建一棵由 React 元素组成的树。在下一次 state 或 props 更新时，相同的 render() 方法会返回一棵不同的树。React 需要基于这两棵树之间的差别来判断如何高效地更新 UI，以匹配最新的树。

这个 `以最少的操作次数将一棵树转换成另一棵树` 的算法上的问题，有一些通用的解决方案。然而，即使最优的算法，复杂程度仍为 O(n 3 )，其中 n 是树中元素的数量。

如果在 React 中使用该算法，那么展示 1000 个元素则需要进行 10 亿数量级次的比较。这个开销实在是太过高昂。于是 React 基于两个假设提出了一套 O(n) 的启发式算法：

1. 两个不同类型的元素会产生出不同的树；
2. 开发者可以通过设置 key 属性，来告知哪些子元素在不同的渲染之间可以保持不变；

在实践中，以上假设在几乎所有实用的场景下都成立。

## Diffing 算法

当对比两棵树时，React 首先比较两棵树的根节点。对比的行为视根元素类型的情况而异（需要考虑两方面：前后元素类型是否相同、是 react DOM 节点或是 Component 类型节点 ）。

### 类型不同的元素

当根节点元素类型不同时，React 会

- 拆卸原有的树：销毁旧的 DOM 节点、组件实例将执行 componentWillUnmount()
- 从头建立起新的树：新的 DOM 节点会被插入到 DOM 中。组件实例将执行 UNSAFE_componentWillMount() ，紧接着 componentDidMount() 。原有的树关联的状态会销毁。

在根节点以下的组件也会被卸载，它们的状态会被销毁。

例：

```jsx
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

React 会销毁 Counter 组件并且重新装载一个新的组件。

### 类型相同的元素

#### React DOM 元素

保留 DOM 节点，仅更新变化了的属性

例：

```jsx
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

在这两个元素之间进行转换时，React 知道只需要修改 color 样式，无需修改 fontWeight。

在处理完当前 DOM 节点之后，React 继续对子节点进行递归

#### 组件元素

组件更新时，组件实例会保持不变，因此 state 会在不同的渲染之间被保持。
React 将

- 更新该组件实例的 props 以匹配最新的元素
- 调用该实例的 UNSAFE_componentWillReceiveProps()、UNSAFE_componentWillUpdate() 以及 componentDidUpdate() 方法。

下一步，调用 render() 方法，diff 算法将递归地比对之前的结果以及新的结果。

### 对子节点进行递归

默认情况下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation。

当在子元素列表末尾新增元素时，更新开销比较小。比如：

```jsx
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

React 会先匹配两个 `<li>first</li>` 对应的树，然后匹配第二个元素 `<li>second</li>`对应的树，最后插入第三个元素的 `<li>third</li>` 树。

但如果只是天真地这样实现，将新增元素插入到表头时会性能会变差。比如：

```jsx
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

React 并不会意识到应该保留 `<li>Duke</li>` 和 `<li>Villanova</li>`，而是会重建每一个子元素。这种情况会带来性能问题。

### Keys

为了解决上述问题，React 引入了 key 属性。当子元素拥有 key 时，React 使用 key 来匹配原有树上的子元素以及最新树上的子元素。为之前的低效的例子中新增 key ，将使得树的转换更高效：

```jsx
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

现在 React 知道 key 为 '2014' 的元素是新元素，而 key 为 '2015' 以及 '2016' 的元素仅仅时位置移动了。

实际开发中，找到一个 key 并不困难。你要展现的元素可能已经有了一个唯一 ID，于是 key 可以直接从你的数据中提取：

```jsx
<li key={item.id}>{item.name}</li>
```

当以上情况不成立时，你可以新增一个 ID 字段到 model 中，或者利用一部分内容生成哈希值来作为 key。这个 key 不需要全局唯一，但在列表中需要保持唯一。

不得已的情况下，也可以使用数组中下标作为 key。这在元素不进行重排时比较合适，如果有重排，diff 就会变慢。也会导致重排时组件的状态出问题。由于组件实例是基于 key 来进行更新和复用的，如果 key 是下标，移动一个元素时就会修改 key 的值，
此时组件用来管理非受控的 inputs 的状态可能会相混，出现无法预期的更新。

在 Codepen 有两个例子，分别为 展示使用下标作为 key 时导致的问题，以及不使用下标作为 key 的例子的版本，修复了重新排列，排序，以及在列表头插入的问题。

## 权衡

请谨记协调算法是一个实现细节。React 可以在每个 action 之后对整个应用进行重新渲染，得到的最终结果也会是一样的。这里说的重新渲染表示调用所有组件的 render 方法，而非 React 会卸载并重新装载它们。React 只会应用将按照以上提到的规则得到的差异。

我们定期优化启发式算法，让常见用例更高效地执行。在当前的实现中，可以理解为一棵子树能在其兄弟之间移动，但不能移动到其他位置。在这种情况下，算法会重新渲染整棵子树。

由于 React 依赖启发式算法，因此当以下假设没有得到满足，性能会有所损耗。

该算法不会尝试匹配不同组件类型的子树。如果你发现你在两种不同类型的组件中切换，但输出非常相似的内容，建议把它们改成同一类型。在实践中，我们没有遇到这类问题。
Key 应该具有稳定，可预测，以及列表内唯一的特质。不稳定的 key（比如通过 Math.random() 生成的）会导致许多组件实例和 DOM 节点被不必要地重新创建，这可能导致性能下降和子组件中的状态丢失。
