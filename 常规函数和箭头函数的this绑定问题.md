# 常规函数和箭头函数的this绑定问题

## 核心要点
**function(){} 和 () => {} 的最大区别在于前者会在运行时绑定this对象，后者不会**

由于function(){}在运行时在内部自动绑定this对象，则不会访问外部作用域this
由于() => {}不能自动绑定this对象，则只能访问外部作用域this

如果要使用动态绑定的this，则使用function(){}
如果要使用外部作用域的this，则使用() => {}

> 估计ES6就是因为function(){}总是自动绑定this造成嵌套function时需要let self = this,然后需要在内层function使用self.xxx调用外层this，
> 才新添加了一个不会自动绑定this的函数：箭头函数 () => {}

## function(){}的this绑定规则
function(){}在运行时（被调用的时候）会基于绑定规则动态进行this绑定

有两点强调：
- this是在运行时绑定的，不是定义时
- 要看this的指向，关键在于看函数调用的位置和方式

根据函数调用位置和方式不同，有四种绑定策略

1. 默认绑定
如果直接调用函数，则将this绑定到全局对象，在Node中是global，在浏览器中是window
```javascript
function foo () {
    console.log(this)
}

// 运行时绑定
foo()
```

2. 隐式绑定
如果有上下文对象调用函数，则将this绑定到上下文对象
```javascript
const obj = {
    foo: function () {
        console.log(this)
    }
}

// 运行时绑定
obj.foo()
```

3. 显式绑定
通过调用函数的call,apply,bind方法进行绑定
```javascript
function foo () {
    console.log(this);
}
const obj = { a: 1 }

// 运行时绑定
foo.call(obj)
foo.apply(obj)
foo.bind(obj)()
```

4. new绑定
new的过程也叫函数的“构造调用”，经历了以下几个步骤：
    1. 创建一个空对象obj
    2. 将对象obj的Prototype关联到构造函数的Prototype
    3. 将对象绑定到构造函数的this
    4. 如果构造函数没有返回对象，则返回对象obj
```javascript
function Foo () {
    console.log(this);
}

// 运行时绑定
foo = new Foo()
```

## 常见场景归纳

1. 对象方法定义：使用function(){} 以获得this绑定
2. 回调函数场景：使用() => {} 以访问外部作用域this
```javascript
const dog = {
    name: "旺仔",
    eat: function (callback) {
        // 这里的this采用隐式绑定策略
        // 因为调用方式是this.eat() (run和sleep方法中)
        setTimeout(() => {
            console.log(`${this.name}吃饱了`)
            callback()
        }, 3000);
    },
    run: function () {
        // 这里的this无法进行绑定，只能从外层作用域获得,在这里指向最外层（模块级别）this
        // 在node中模块级别this指向module.exports
        // node是模块化的，模块本身存在独立作用域，要和浏览器的全局概念区分
        this.eat(() => {
            console.log(`${this.name}开始跑步`);
        })
    },
    sleep: function () {
        // 这里的this采用默认绑定策略
        // 因为调用方式是callback() (eat方法中)
        this.eat(function () {
            console.log(`${this.name}准备睡觉`);
        })
    }
}
```

3. 事件监听场景：使用function(){} 以获得this绑定
```javascript
// 事件监听器调用仍然是上下文调用，socket.handle()
socket.on('data', function handle () {
    console.log(this)
})
```