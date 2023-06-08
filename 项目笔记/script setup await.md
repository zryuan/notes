## script setup 中使用await 页面白屏

script setup 中允许在顶层直接使用await，结果代码会编译成async setup。

但是在实际使用中会发现页面白屏。

后来经过多次测试发现，async setup中使用awiat会阻塞组件渲染，所以导致页面白屏的问题。

### 问题猜测

在vue官方文档有说过在渲染一颗组件树时，每一个组件都有自己的加载状态。

因为setup是组件被初始化时最先执行的钩子，如果存在await就需要等待异步完成后执行后续代码，而这时组件还没被创建也不会进行挂载。

所以可能父组件已经加载完成并已经挂在Dom上进行渲染，而子组件因为没有挂载所以没有在页面上渲染出来。

### 解决

1. 避免顶层直接使用await，而是将需要异步的内容包装成async函数，在函数内使用await。
2. 官方文档中指出如果使用async setup必须也[Suspense 内置组件](https://cn.vuejs.org/guide/built-ins/suspense.html)搭配使用。[顶层 await](https://cn.vuejs.org/api/sfc-script-setup.html#top-level-await)