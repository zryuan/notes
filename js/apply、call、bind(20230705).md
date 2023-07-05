## apply、call、bind

apply、call、bind这些方法都是用来修改函数绑定的this指向。

不同之处在于：apply和call用来调用函数并修改函数this指向，而bind是返回一个绑定了this指向的函数。



### 手撕代码

#### apply

apply方法接收两个参数，第一个参数是调用函数时指定的this指向，如果该值为null、undefined时在非严格意义下会指向全局对象。

第二个参数是传递给函数的参数列表以数组的形式接收。

下面是myApply的实现

```javascript
Function.prototype.myApply = function (target = window, args = []) {
    const key = Symbol();
    target[key] = this;
    const val = target[key](...args);
    Reflect.deleteProperty(target, key);
    return val;
}
```

1. 在Function的prototype对象上实现一个myApply方法，那么所有的函数都会继承该方法。
2. this指向调用myApply方法的函数对象
3. 通过target对象调用该方法，从而修改函数的this指向。

#### call

call方法与apply方法是一样的，不同在于call方法除了第一个参数是this指针外，其余后面的参数都是需要传递给函数的参数。

myCall的实现

```javascript
Function.prototype.myCall = function (target = window, ...args) {
    const key = Symbol();
    target[key] = this;
    const val = target[key](...args);
    Reflect.deleteProperty(target, key);
    return val;
}
```

#### bind

bind参数与call方法一样，但是返回的是一个绑定了this指针的函数。

myBind的实现

```javascript
Function.prototype.myBind = function (target = window, ...args) {
   	const self = this;
    function fn(...args2) {
        if (this instanceof fn) {
            return new self(...args, ...args2);
        }
        const key = Symbol();
        target[key] = self;
        const val = target[key](...args, ...args2);
        Reflect.deleteProperty(target, key);
        return val;
    }

    return fn;
}
```

bind绑定this指针不会立即执行，而是返回一个函数。

所以需要注意的是函数也可以作为**构造函数**被new调用，因此就要判断函数是直接调用的还是使用new调用的。

如果是new调用的，按照bind的表现来看相当于直接new原函数，也**就是bind绑定的this被丢弃了**。

还有需要注意的是bind可以接收多个参数，bind方法返回的函数也可以接收参数，而传给原函数的参数是这两个接收的参数列表拼接起来的。

测试代码补充：

```javascript
function fn(a, b) {
    this.a = a;
    this.b = b;
}
fn.prototype.run = function () {
    console.log('running....')
}
const obj = {
    name: 'zhan',
    age: 18
}
const myFn = fn.myBind(obj,1,2);
const newObj = new myFn(5, 6)
console.log(newObj);
```

当然上面实现代码中都没有错误相关的判断，例如没有做apply、call和bind的第一个参数是否是一个对象的校验等等。

还有就是apply、call和bind方法**不适用于箭头函数**，因为箭头函数本身是没有this指针的，而是从它的执行上下文获取的。