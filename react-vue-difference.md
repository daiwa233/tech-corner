# vue 和 react 的区别

vue 和 react 都是前端最流行的框架，那么在我们的项目开发中选择哪个框架之前，我们需要对这两个框架尽量充分的了解，本文就总结一下 vue 和 react 之间的区别。

- 使用体验
- 应用性能
- 实现上
  - 架构不同
  - 响应式原理不同
  - 模版语法不同
  - 逻辑复用手段不同
- roadMap

1. 架构不一样。vue 大部分遵循 MVVM 架构，react 是 ui = render(data)
2. **响应式**实现原理不同。vue 通过 Object.defineProperty / Proxy 实现数据劫持，而 react 则是组件状态更新时，遍历子组件树，来实现更新

3. Jsx 和 template
4. 逻辑复用的手段不同。vue2 是 mixins ，react 是 hoc/hooks