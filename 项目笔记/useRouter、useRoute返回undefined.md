## useRouter、useRoute返回undefined问题

### 问题描述

异步代码中使用useRouter、useRoute会返回undefined，跟是否只能在setup中或者setup顶层使用没关系。

因为在mounted钩子中使用useRouter、useRoute组合式api是正常的。

具体原因需查看具体源码实现。**=> 待查**