# 性能优化

## 避免 Reconciliation

React 构建并维护了要渲染的 UI 视图的内部表示。包含了组件返回的 React 元素。以此来避免不必要的 DOM 节点的创建和访问，因为操作 DOM 相比于操作 JavaScript 对象要慢。有时候这被称为“虚拟 DOM”，但是它在 React Native 工作原理与此相同。

当一个组件的 props 或 state 改变时，React 会将最新返回的元素与之前渲染的元素进行对比，以此决定是否有必要更新真实的 DOM。当它们不相同时，React 会更新该 DOM。

即使 React 只更新改变了的 DOM 节点， **重新渲染** 仍然花费了一些时间。大部分情况下这不是问题，不过如果它已经慢到让人注意了，可以通过重写生命周期方法 `shouldComponentUpdate` 来进行提升速度。它会在重新渲染的步骤开始之前被触发。其默认实现返回 true，让 React 执行更新：

```js
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

如果已经知道组件不需要更新，可以在 `shouldComponentUpdate` 中返回 false 来跳过整个渲染过程。包括 render 的调用以及之后的操作。

大多数时候，不用手写 `shouldComponentUpdate()`。可以继承 `React.PureComponent` ，它等价于将 shouldComponentUpdate() 实现为新旧 props 和 state 的浅比较。

## shouldComponentUpdate 实例

这是一棵由组件构成的子树

- C2: shouldComponentUpdate 在 C2 为根的子树处返回了 false，React 不会尝试重新渲染 C2

  - 于是甚至不会调用 C4 和 C5 的 shouldComponentUpdate

- C1 和 C3, SCU 返回 true, 于是 React 需要向下检查子节点，
  - C6： SCU 返回 true, 而且渲染的元素不相等，于是 React 需要更新 DOM
  - 对于 C8。React 需要渲染这个组件，但是其返回的 React 元素和之前渲染的相同，所以不需要更新 DOM。

注意 React 只需为 C6 修改 DOM。对于 C8，通过对比了渲染的 React 元素跳过了更新。而对于 C2 的子树和 C7，由于 shouldComponentUpdate 返回 false ，甚至不需要对比元素， render 并没有被调用。
