## keep-alive缓存失效问题

### 问题描述

在项目中使用keep-alive缓存某个组件，在页面返回时保持原有状态，结果效果是组件没有缓存成功，导致页面看上去像进行了重新刷新。

### 原因

在keep-alive组件使用了include props，原本以为include接收参数值是路由**配置项中的name**属性，但其实是**组件的name选项**。

```jsx
 <keep-alive :include="['myMPFList', 'localTest']">
    <component :is="Component" />
 </keep-alive>
```

keep-alive组件如果没有定义include/exclude props，则默认会缓存所有keep-alive内部组件，这时是不需要显式的声明组件name选项。

keep-alive组件如果定义include/exclude props进行条件缓存，则需要显式的定义组件的name选项，以告知keep-alive那些组件需要进行缓存。

在vue3 中script setup中是没法定义组件的名称，需要使用普通的script 额外导出name选项

```javascript
<script>
export default {
  name: 'myMPFList',
};
</script>
```

在 3.2.34 或以上的版本中，使用 script setup 的单文件组件会自动根据文件名生成对应的 name 选项，无需再手动声明。[keep-alive](https://cn.vuejs.org/guide/built-ins/keep-alive.html#include-exclude)