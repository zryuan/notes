## defineProperty、proxy

在vue2中使用了defineProperty方法实现数据的响应式，而在vue3中替换成了proxy代理的方式实现响应式。



### defineProperty

Object.defineProperty方法，按照字面意思就是用来定义属性的。

该方法可以定义新的属性也可以修改现有属性。

语法格式如下：

```javascript
Object.defineProperty(obj,prop,descriptor)
```

obj：要定义属性的目标对象

prop：需要定义属性的key

descriptor：属性的描述符对象

实例：

```javascript
const person = {
    name: 'zhangsan',
    age: 18
}

Object.defineProperty(person, 'height', {
    value: 183
});

console.log(person);
```

上面实例中使用defineProperty方法给person对象定义了一个新的属性height，value就是定义height属性的一个描述符，描述该属性的值。

#### 描述符对象属性

除了value外还有以下描述符

```javascript
{
    value: undefined,
    writable: false,
    get: undefined,
    set: undefined,
    configurable: false,
    enumerable: false
}
```

value：属性值，默认值**undefined**

writable：属性值是否允许修改，默认值**false**

get：getter函数，当访问该属性时触发，返回值就是该属性的值，如果没有定义getter函数默认值是undefined

set：setter函数，当修改属性值时触发，接收一个参数（修改后的值），如果没有定义setter函数默认值是undefined

configurable：是否允许修改对象属性的描述符，默认false

emumerable:  定义该属性是否可以被枚举，默认是false



以上描述符被分为**数据描述符**和**访问器描述符**

数据描述符：value、writable

访问器描述符：get、set

在定义属性时两种描述符不允许同时存在，因为它们的作用是一样的，同时存在会发生冲突

```javascript
Object.defineProperty(person, 'height', {
    value: 183,
    get() {
        return 190
    }
});

//报错
//defineProperty.js:6 Uncaught TypeError: Invalid property descriptor. Cannot both specify accessors and a value or writable attribute
```

除了避免以上两种描述符同时存在外，其他描述符都可以随意使用

#### writable

默认值为false，也就是如果没有配置该描述符那么该属性是不可被修改的

```javascript
Object.defineProperty(person, 'height', {
    value: 183,
});
person.height = 190
console.log(person); // {name: 'zhangsan', age: 18, height: 183}

//改为
Object.defineProperty(person, 'height', {
    value: 183,
    writable: true
});
person.height = 190
console.log(person); //{name: 'zhangsan', age: 18, height: 190}
```

#### get

getter函数返回值就是该属性的值

```javascript
let height = 183;
Object.defineProperty(person, 'height', {
    get() {
        return height;
    }
});
person.height = 190
console.log(person.height); // 183
```

当访问person的height属性时会触发get函数，该函数的返回值就是该属性的值

这里因为没有定义set，所以该属性值也是无法被修改的。

#### set

setter函数，用来自定义该属性值的修改

当该属性值被重新赋值时触发

```javascript
let height = 183;
Object.defineProperty(person, 'height', {
    get() {
        return height;
    },
    set(val) {
        console.log(val); // 190
        height = val;
    }
});
person.height = 190
console.log(person.height); // 190
```

接收一个参数，该参数的值就是被赋值的值。

通过get和set可以更方便我们控制属性的读和写，以及做一些额外的事情

#### enumerable

enumerable 描述符定义了该属性是否可被枚举，默认值是false

一般以下几种方法需要对象属性进行枚举：

1. Object.keys、Object.values、Object.entries：这些方法只会获取对象中可枚举的属性及值
2. 展开运算符：使用展开运算符去拷贝或者合并对象时，只有可枚举的属性才会被拷贝
3. for...in...：for in 循环也只会遍历对象的可枚举属性
3. JSON.stringify：只会对对象的可枚举属性进行序列化
4. [更多](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)

实例：

```javascript
// 1、默认值
Object.defineProperty(person, 'height', {
    value: 183,
})

const keys = Object.keys(person);
const values = Object.values(person);
const entries = Object.entries(person);
const newPerson = { ...person }

console.log("keys", keys) //keys (2) ['name', 'age']
console.log("values", values) // values (2) ['zhangsan', 18]
console.log("entries", entries) // entries (2) [Array(2), Array(2)]
console.log('newPerson', newPerson) // newPerson {name: 'zhangsan', age: 18}

// 添加 enumerable: true
Object.defineProperty(person, 'height', {
    value: 183,
    enumerable: true
})

...

//打印结果：
//keys (3) ['name', 'age', 'height']
//values (3) ['zhangsan', 18, 183]
//entries (3) [Array(2), Array(2), Array(2)]
//newPerson {name: 'zhangsan', age: 18, height: 183}
```

默认值是false，所以使用Object.keys等方法进行枚举时会忽略这个属性。

for in 以及展开运算符等也是同样的效果

#### configurable

configurable 描述符定义该属性是否可以从对象中删除以及是否可以修改其他描述符的值

```javascript
//默认值 false
Object.defineProperty(person, 'height', {
    value: 183,
});

Reflect.deleteProperty(person, 'height');
//delete person.height
console.log('height' in person) // true

Object.defineProperty(person, 'height', {
    enumerable: true
})
const keys = Object.keys(person)
console.log(keys) //Uncaught TypeError: Cannot redefine property: height
// at Function.defineProperty
```

当不配置configurable值时，默认是false。这时是不允许删除该属性的，同时修改该属性的其他描述符时会抛出错误，不允许修改

当值改为true时：

```javascript
Object.defineProperty(person, 'height', {
    value: 183,
    configurable: true
});
Object.defineProperty(person, 'height', {
    enumerable: true
})
const keys = Object.keys(person)
console.log(keys) //  ['name', 'age', 'height']
Reflect.deleteProperty(person, 'height');
//delete person.height
console.log('height' in person) // false
```

当configurable设为true时，height属性的enumerable描述符的值就允许被修改，该属性也可以被删除了

当configurable值为false时存在两种特殊情况：

1、当writable的值为true时，value值也可以被修改

例如：

```javascript
Object.defineProperty(person, 'height', {
    value: 183,
    writable: true,
    configurable: false
});
Object.defineProperty(person, 'height', {
    value: 190
})
//person.height = 190;
console.log(person.height) // 190
```

2、当writable的值为true时，其值也可修改为false

例如：

```javascript
Object.defineProperty(person, 'height', {
    value: 183,
    writable: true,
    configurable: false
});
Object.defineProperty(person, 'height', {
    writable: false
})
person.height = 190;
console.log(person.height) // 183
```

赋值是没有成功的，因为该属性又变成了不可重写。

所以在configurable为false，writable为true时，存在以上两种特殊情况。

#### 常规创建属性

当使用例如以下的方式创建属性时：

```javascript
// 直接创建一个对象
const person = {
    name: 'zhangsan',
    age: 18
}
// 给对象直接赋值
person.height = 190

...
```

它们的属性描述其实如下所示

```javascript
Object.defineProperty(person, xxx, {
    value: xxx,
    writable: true,
    configurable: true,
    enumerable: true
});
```

### Proxy

proxy用来创建一个对象的代理，从而实现基本操作的拦截和自定义

proxy构造器接收两个参数

```javascript
const proxyObj = new Proxy(target,handler)
```

target：代理的目标对象

handler：捕捉器对象，定义各种捕捉器例如get、set和has等，更多可以查看MDN文档

实例：

```javascript
const person = {
    name: 'zhangsan',
    age: 19
}
const personProxy = new Proxy(person, {
    get(target, key, receiver) {
        return target[key]
    },
    set(target, key, val, receiver) {
        target[key] = val
    }
})
```

get和set参数：

1. target：代理的目标对象
2. key：访问代理对象具体的属性
3. val：赋值的属性值
4. receiver：生成的代理对象，这里就是personProxy

### defineProperty 和 Proxy 的区别

1. defineProperty和Proxy有着本质的不同，defineProperty是用来定义对象属性的，proxy是用来做对象代理的。
2. defineProperty会修改原始数据，而proxy不会，它会生成一个新的代理对象
3. 数据响应式：defineProperty需要深层次递归遍历对象，劫持每一个属性，进行数据绑定，而proxy是在对象层面进行代理不需要对对象进行遍历。
4. 对于新增属性：需要重新调用defineProperty方法进行数据绑定，而proxy不需要
5. proxy除了可以代理get和set之外，还能代理对象的其他行为，例如 in delete等等



### 参考：

[MDN Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

[MDN defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)







