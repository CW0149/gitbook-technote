# JS难点

## 你不知道的JS笔记

* [作用域](../tech/scope.md)
* [this和对象原型](../tech/this-and-object-prototypes.md)
* [类型和语法](../tech/Types-and-Grammar.md)

## [this](../tech/this-and-object-prototypes.md#this)

> In browsers, the top-level scope is the global scope. This means that within the browser var something will define a new global variable. In Node.js this is different. The top-level scope is not the global scope; var something inside a Node.js module will be local to that module.

### 判断this的绑定

- 函数是否在new中调用(new绑定)?如果是的话this绑定的是新创建的对象。 var bar = new foo()
- 函数是否通过call、apply(显式绑定)或者硬绑定调用?如果是的话，this绑定的是 指定的对象。 var bar = foo.call(obj2)
- 函数是否在某个上下文对象中调用(隐式绑定)?如果是的话，this 绑定的是那个上 下文对象。 var bar = obj1.foo()
- 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到 全局对象。 var bar = foo()

```
// 1
var obj = {
  say: function () {
    function _say() {
      console.log(this) // window
    }
    return _say.bind(obj)
  }()
}
obj.say()

// 2
var obj = {}
obj.say = function () {
  function _say() {
    console.log(this) // obj
  }
  return _say.bind(obj)
}()
obj.say()

// 3
var obj = {
	name: 'Alice',
	foo: function() {
		console.log(this.name)
	}
}
var anotherObj = {
	name: 'Lily',
	foo: obj.foo
}
obj.foo() // Alice
anotherObj.foo() // Lily

// 4
var name = 'Alice'
var obj = {
	name: 'Lily',
	foo: function() {
		console.log(this.name)
	}
}
obj.foo() // 'Lily'
var f1 = obj.foo
f1() // 'Alice'

// 5
var name = 'Bob'
var obj = {
	name: 'Alice',
	foo: function() {
		console.log(this.name)
	}
}
var anotherObj = {
	name: 'Lily',
	foo: function() {
		var f1 = obj.foo
		f1()
	}
}
anotherObj.foo() // 'Bob'

// 6
var name = 'Tom'
var obj = {
	name: 'Alice',
	showName: function() {
		console.log(this.name)
	},
	foo: function() {
		(function(cb) {
			cb()
		})(obj.showName)
	}
}
obj.foo() // Tom

```

### bind

使用call(...)和apply(...)方法，可以将this指定绑定到特定对象上。apply支持的是数组参数。

```
// 实现softBind
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function(obj) {
    var fn = this;
    // 捕获所有 curried 参数
    var curried = [].slice.call( arguments, 1 );
    var bound = function() {
        return fn.apply(
            (!this || this === (window || global)) ?
            obj : this
            curried.concat.apply( curried, arguments )
        );
    };
    bound.prototype = Object.create( fn.prototype );
    return bound;
  };
}
```

### 箭头符号

```
// es6
var a = {
	fn: function() {
    	fn1:() => {
        	console.log(this)
        }
    }
}

// es5
"use strict";

var a = {
  fn: function fn() {
    var _this = this;

    fn1: (function () {
      console.log(_this);
    });
  }
};

// es6
var a = {
	fn: () => {
    	fn1:() => {
        	console.log(this)
        }
    }
}

// es5
"use strict";

var _this = void 0;

var a = {
  fn: function fn() {
    fn1: (function () {
      console.log(_this);
    });
  }
};
```

## [作用域](../tech/scope.md)

```
// 1
foo() // 2
var foo = function() {console.log(1)}
function foo() {console.log(2)}

foo() // 3
function foo() {console.log(2)}
function foo() {console.log(3)}
var foo = function() {console.log(1)}

var foo = function() {console.log(1)}
function foo() {console.log(2)}
foo() // 1

// 2
if (someVar === undefined) {
	var someVar = 2
	conosle.log('条件判断不影响声明提升')
}
```

## [对象、原型、继承](../tech/this-and-object-prototypes.md#对象)

```
// es6实现一个能够监听绑定事件的“类”
class Base {
	constructor() {
		this.eventMap = {};

	}
	on(event, fn) {
		(this.eventMap[event] = this.eventMap[event] || []).push(fn)
	}
	trigger(event, value) {
		const eventFn = this.eventMap[event];
		if (eventFn) {
			eventFn.forEach(fn => {
				fn.call(this, value);
			});
		}
	}
}

class View extends Base {
  constructor(options) {
    super(options)
  }
}

// 以上es5的实现
function Base() {
	this.eventMap = {};
}

Base.extend = function (proto, stat) {
	const Constructor = this;
	const Fn = function() {
		Constructor.call(this);
	};
	Fn.prototype = Object.create(this.prototype);
	Fn.extend = Base.extend;

	proto && Object.keys(proto).forEach(function(key) {
		Fn.prototype[key] = proto[key];
	});

	stat && Object.keys(stat).forEach(function(key) {
		Fn[key] = stat[key];
	});

	return Fn;
}

Base.prototype.on = function(event, fn) {
	(this.eventMap[event] = this.eventMap[event] || []).push(fn)
}

Base.prototype.trigger = function(event, value) {
	const eventFn = this.eventMap[event];
	const self = this;
	if (eventFn) {
		eventFn.forEach(function(fn) {
			fn.call(self, value);
		});
	}
}

var MyClass = Base.extend({
  getVal: function () {
    return 'hello world'
  }
}, {
  say: function (word) {
    return word
  }
})

// 测试
var myclass = new MyClass
myclass.getVal() // 'hello world'
MyClass.say('haha') // 'haha'
myclass instanceof MyClass // true
myclass instanceof Base // true
```

## 资料

* [Babel在线转码](https://babeljs.io/repl)
* [JavaScript深入之bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)