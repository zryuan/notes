## pushState

pushState 方法向游览器的会话历史栈添加一个条目

该方法接受三个参数

```javascript
pushState(state,unused,url)
```

### state

该历史条目的状态对象，可以是任何可以序列化的对象，常常用来保存一些状态属性，以便popState事件触发时使用。

通过**history.state**可以查看当前历史条目的state。

### unused

历史原因无用值，传一个空字符串

### url

新历史条目的url，可以是相对路径也可以说绝对路径，如果是相对路径则会相对于当前url进行解析，如果是绝对路径那么该url必须与当前url同源，不然会抛出异常。

游览器不会在调用pushState之后尝试加载该url的资源，也就是说调用pushState向历史栈加入一条历史条目后，游览器不会去加载资源进行页面跳转，而是保持当前页面只是游览器地址栏的url变为了新条目的url。

当是手动刷新或者重启游览器等方法重新访问该url时，还是会加载该url对应的资源，所以服务器上还是需要有相应的解决方案，保证通过url都能进行页面访问。

url是一个可选值，如果没传默认为当前页面url。

## popstate

window上的一个事件，在不同历史条目进行切换时触发，回调接收一个事件对象event

```javascript
window.addEventListener('popstate', (event) => {})
```

 event对象上存在state字段是当前历史条目state的拷贝。

调用pushState和replaceState方法不会触发popstate事件，popstate事件只会在游览器的某些行为下触发，例如点击前进后退按钮，或者调用history中进行历史条目切换的方法（go、back、forward等）还有location.href方法也会对历史条目进行切换。

包括手动在游览器修改url的hash值，也会向历史栈中添加一条历史条目，所以也会触发popstate事件。

## replaceState

replaceState跟pushState方法一样，不同点在pushState方法向历史栈中添加一个历史条目，而replaceState是将当前的历史条目进行替换，一个是添加一个替换。

## 参考

[MDN pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState)

[MDN popstate](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/popstate_event)
