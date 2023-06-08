## vue3 生命周期的执行顺序

### 组件的加载顺序

1、父组件的setup

2、父组件的onBeforeMount

3、子组件的setup

4、子组件的onBeforeMount

5、如果父组件存在多个子组件，每个子组件依次执行3和4

6、当所有子组件都执行完3和4后，所有子组件依次进行挂载执行子组件的onMounted

7、当所有同步子组件都完成挂载，父组件进行挂载，执行父组件onMounted

### 组件的更新顺序

单组件状态发生变更，只会触发自身组件的onBeforeUpdate、onUpdated

当一个状态存在于子父组件时，状态发生变更，先执行父的onBeforeUpdate -> 子onBeforeUpdate -> 子onUpdated -> 父onUpdated

### 组件的卸载顺序

1、父组件的onBeforeUnmount

2、子组件的onBeforeUnmount

3、存在多个子组件时，每个子组件依次执行2

4、当所有子组件都完成了onBeforeUnmount，依次进行卸载执行子组件的onUnmounted

5、当所有子组件都完成卸载，父组件进行卸载执行父组件的onUnmounted