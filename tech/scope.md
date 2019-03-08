# 作用域

<!-- toc -->

## 作用域是什么

几乎所有语言的最基础模型之一就是在变量中存储值，并且在稍后取出或修改这些值的能力。但是在我们的程序中纳入变量，引出了我们现在将要解决的最有趣的问题：这些变量存活在哪里？换句话说，它们被存储在哪儿？而且，最重要的是，我们的程序如何在需要它们的时候找到它们？这就引出了作用域。

作用域是一组明确定义的规则，它定义如何在某些位置存储变量，以及如何在稍后找到这些变量。

在 JavaScript 内部，作用域和对象类似，可见的标识符都是它的属性。但是作用域“对象”无法通过 JavaScript 代码访问，它存在于 JavaScript 引擎内部。

## 理解作用域

### 编译原理

传统编译语言的流程中，程序中一段源码在执行之前会经历三步骤，统称为“编译”。
1. 分词/词法分析-将代码字符串分解为有意义的代码块（词法单元）。
2. 解析/语法分析-将词法单元流数组转换成一个AST（抽象语法树）。如 var a = 2;的抽象语法树可能会有一个叫VariableDeclaration的顶级节点，接下来是叫Identifier(值为a)的子节点，以及一个叫AssignmentExpression的子节点。AssignmentExpression节点有一个叫NumericLiteral(值为2)的子节点。
3. 代码生成-将AST转换为可执行代码。

大部分情况，JavaScript的编译发生在代码执行前的几微秒时间内，为了保证最佳性能，JavaScript对编译过程做了大量优化。

### 三要素

- 引擎-负责整个JavaScript程序的编译和执行过程。
- 编译器-负责词法分析、语法分析、代码生成。
- 作用域-负责收集并维护由所有声明的标志符组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。

对待代码，JavaScript引擎先让编译器将代码分解成词法单元并生成AST和作用域。

编译器生成可执行代码的过程则复杂一些。当遇到var声明的变量，编译器首先会询问作用域当前的作用域集合是否存在同名变量，如果存在则忽略该声明继续编译，若不存在则通知作用域在当前作用域集合声明一个该变量名的变量。同时，编译器会为引擎生成运行时所需代码。

### 作用域嵌套

作用域是根据名称查找变量的一套规则，这套规则用来管理引擎如何在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找。当一个块或函数嵌套在另一个块或函数中时，就发生了作用域的嵌套。因此，在当前作用域中无法找到某个变量时，引擎就会在外层嵌套的作用域中继续查找，直到找到该变量， 或抵达最外层的作用域(也就是全局作用域)为止。

浏览器中使用var在全局作用域定义变量时时，生成的变量是在全局作用域中，使用window可以访问，let则不同。在node.js环境中，module中var定义的变量处在当前module的环境中，node-cli环境下与浏览器行为一致。

### 引擎运行

引擎运行可执行代码时，当遇到变量赋值操作，会执行LHS、RHS两种查询。当变量出现在赋值操作符左侧时进行LHS查询，非左侧进行RHS查询。查询变量时，引擎首先会从当前作用域集合查找是否有该变量，若没找到则逐级向上查询，直到全局作用域。在RHS查询以及严格模式下的LRS查询中，若查询过程未找到变量，引擎会抛出一个ReferenceError错误。在非严格模式下的LRS查询中，若未找到变量，全局作用域中会创建一个该名称的变量并返还给引擎。

### 词法作用域 VS 动态作用域

Javascript使用词法作用域。词法作用域就是定义在词法阶段的作用域。
词法作用域意味着作用域是由书写代码时函数声明的位置来决定的。编译的词法分析阶段基本能够知道全部标识符在哪里以及是如何声明的，从而能够预测在执行过程中如何对它们进行查找。

动态作用域并不关心函数和作用域是如何声明以及在何处声明的，只关心它们从何处调用。换句话说，作用域链是基于调用栈的，而不是代码中的作用域嵌套。因此若要在函数foo()中无法找到 a 的变量引用时，会顺着调用栈在调用 foo() 的地方查找 a，而不是在嵌套的词法作用域链中向上查找。

### eval与with

#### eval

JavaScript 中的 eval(..) 函数可以接受一个字符串为参数，eval()可以运行字符串中的代码，它通常被用来执行动态创建的代码。

**默认情况下，如果 eval(..) 中所执行的代码包含有一个或多个声明(无论是变量还是函数)，就会对 eval(..) 所处的词法作用域进行修改。** 严格模式的程序中，eval(..) 在运行时有其自己的词法作用域，意味着其中的声明无法修改所在的作用域。

new Function(..) 函数的行为也很类似，最后一个参数可以接受代码字符串，并将其转化为动态生成的函数(前面的参数是这个新生成的函数的形参)。这种构建函数的语法比 eval(..) 略微安全一些，但也要尽量避免使用。

在程序中动态生成代码的使用场景非常罕见，因为它所带来的好处无法抵消性能上的损失。

#### with

with 通常被当作重复引用同一个对象中的多个属性的快捷方式，可以不需要重复引用对象本身。**with 声明实际上是根据你传递给它的对象凭空创建了一个全新的词法作用域。**

```
function foo(obj) {
	with (obj) {
		a = 2;
	}
}
var o1 = {
	a: 3
};
var o2 = {
	b: 3
};
foo( o1 );
console.log( o1.a ); // 2
foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2——不好，a 被泄漏到全局作用域上了
```

#### 性能

如果代码中大量使用 eval(..) 或 with，那么运行起来一定会变得非常慢。这两个机制的副作用是引擎无法在编译时对作用域查找进行优化，因为引擎只能谨慎地认为这样的优化是无效的。使用这其中任何一个机制都将导致代码运行变慢。不要使用它们。

JavaScript 引擎会在编译阶段进行数项的性能优化。其中有些优化依赖于能够根据代码的词法进行静态分析，并预先确定所有变量和函数的定义位置，才能在执行过程中快速找到标识符。但如果引擎在代码中发现了 eval(..) 或 with，它只能简单地假设关于标识符位置的判断都是无效的，因为无法在词法分析阶段明确知道 eval(..) 会接收到什么代码，这些代码会如何对作用域进行修改，也无法知道传递给 with 用来创建新词法作用域的对象的内容到底是什么。

## 函数作用域和块作用域

函数作用域的含义是指，属于这个函数的全部变量都可以在整个函数的范围内使用及复用(事实上在嵌套的作用域中也可以使用)。

### 隐藏内部实现

#### 好处

- 在软件设计中，应该最小限度地暴露必要内容，而将其他内容都“隐藏”起来，比如某个模块或对象的 API 设计，设计良好的软件都会依此进行实现。
- 可以避免同名标识符之间的冲突， 两个标识符可能具有相同的名字但用途却不一样，无意间可能造成命名冲突。冲突会导致变量的值被意外覆盖。

#### 规避冲突

- 在全局作用域中声明一个名字足够独特的变量，通常是一个对象。这个对象被用作库的命名空间，所有需要暴露给外界的功能都会成为这个对象(命名空间)的属性，而不是将自己的标识符暴漏在顶级的词法作用域中。
- 从众多模块管理器中挑选一个来使用。使用这些工具，任何库都无需将标识符加入到全局作用域中，而是通过依赖管理器的机制将库的标识符显式地导入到另外一个特定的作用域中。

#### 匿名和具名

匿名函数缺点：

1. 匿名函数在栈追踪中不会显示出有意义的函数名，使得调试很困难。
2. 如果没有函数名，当函数需要引用自身时只能使用已经过期的arguments.callee引用， 比如在递归中。另一个函数需要引用自身的例子，是在事件触发后事件监听器需要解绑自身。
3. 匿名函数省略了对于代码可读性/可理解性很重要的函数名。一个描述性的名称可以让代码不言自明。


#### 立即执行函数表达式(IIFE)

两种书写形式：

- (function foo(){ .. })()
- (function(){ .. }())

示例：

```
(
	function IIFE( global ) {
		var a = 3;
		console.log( a ); // 3 console.log( global.a ); // 2
  })( window );


(
	function IIFE( def ) {
		def( window );
})(
	function def( global ) {
	var a = 3;
	console.log( a ); // 3 console.log( global.a ); // 2
});
```

#### 块作用域

- with
- try/catch
	- try/catch 的 catch 分句会创建一个块作用域，其中声明的变量仅在 catch 内部有效。

	```
try {
	undefined(); // 执行一个非法操作来强制制造一个异常
}
catch (err) {
	console.log( err ); // 能够正常执行!
}
     console.log( err ); // ReferenceError: err not found
	```
- let
	- let 关键字可以将变量绑定到所在的任意作用域中，只要声明是有效的，在声明中的任意位置都可以使用 { .. } 括号来为 let 创建一个用于绑定的块。
	- 但是使用 let 进行的声明不会在块作用域中进行提升。
	- 块作用域可以打消垃圾回收的顾虑。
	- for 循环头部的 let 不仅将 i 绑定到了 for 循环的块中，事实上它将其重新绑定到了循环的每一个迭代中，确保使用上一个循环迭代结束时的值重新进行赋值。
	```
	if (foo) {
		{ // <-- 显式的快
			let bar = foo * 2;
			bar = something( bar );
			console.log( bar );
		}
	}

	{
		console.log( bar ); // ReferenceError! let bar = 2;
	}

	function process(data) {
		// 在这里做点有趣的事情
	}
	// 在这个块中定义的内容可以销毁了!
	{
	let someReallyBigData = { .. }; process( someReallyBigData );
	}
	var btn = document.getElementById( "my_button" );
	btn.addEventListener( "click", function click(evt){
	   console.log("button clicked");
	}, /*capturingPhase=*/false );
	```
- const


### 小结

函数是 JavaScript 中最常见的作用域单元。本质上，声明在一个函数内部的变量或函数会在所处的作用域中“隐藏”起来，这是有意为之的良好软件的设计原则。

但函数不是唯一的作用域单元。块作用域指的是变量和函数不仅可以属于所处的作用域，也可以属于某个代码块(通常指 { .. } 内部)。

从 ES3 开始，try/catch 结构在 catch 分句中具有块作用域。

在 ES6 中引入了 let 关键字(var 关键字的表亲)，用来在任意代码块中声明变量。if (..) { let a = 2; } 会声明一个劫持了 if 的 { .. } 块的变量，并且将变量添加到这个块中。

有些人认为块作用域不应该完全作为函数作用域的替代方案。两种功能应该同时存在，开发者可以并且也应该根据需要选择使用何种作用域，创造可读、可维护的优良代码。

## 提升

我们习惯将var a = 2;看作一个声明，而实际上JavaScript引擎并不这么认为。它将var a 和 a = 2 当作两个单独的声明，第一个是编译阶段的任务，而第二个则是执行阶段的任务。

声明函数优先：若存在同名函数及变量，忽略变量。后面同名函数会覆盖前面函数。

## 作用域闭包

当函数可以记住并访问所在的词法作用域，即使函数是在当前词法作用域之外执行，这时就产生了闭包。

```
function wait(message) {
  setTimeout( function timer() {
      console.log( message );
	}, 1000 );
}
wait( "Hello, closure!" );
```

深入到引擎的内部原理中，内置的工具函数 setTimeout(..) 持有对一个参数的引用，这个参数也许叫作 fn 或者 func，或者其他类似的名字。引擎会调用这个函数，在例子中就是内部的 timer 函数，而词法作用域在这个过程中保持完整。

如果将函数(访问它们各自的词法作用域)当作第一级的值类型并到处传递，你就会看到闭包在这些函数中的应用。在定时器、事件监听器、 Ajax 请求、跨窗口通信、Web Workers 或者任何其他的异步(或者同步)任务中，只要使用了回调函数，实际上就是在使用闭包!

### 循环和闭包

```
for (var i=1; i<=5; i++) {
	setTimeout( function timer() {
      console.log( i );
  }, i*1000 );
}

for (var i=1; i<=5; i++) {
	(function() {
		var j = i;
		setTimeout(
			function timer() {
				console.log( j );
      }, j*1000 );
  })();
}
```
使用let： 本质上这是将一个块转换成一个可以被关闭的作用域。for 循环头部的 let 声明还会有一个特殊的行为。这个行为指出变量在循环过程中不止被声明一次，每次迭代都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

```
for (var i=1; i<=5; i++) {
	let j = i; // 是的，闭包的块作用域!
	setTimeout( function timer() {
      console.log( j );
  }, j*1000 );
}

for (let i=1; i<=5; i++) {
	setTimeout(
		function timer() {
      console.log( i );
  }, i*1000 );
}
```

### 模块

模块模式需要具备两个必要条件：
1. 必须有外部的封闭函数，该函数必须至少被调用一次(每次调用都会创建一个新的模块实例)。
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

#### 现代的模块机制

```
var MyModules = (
	function Manager() {
		var modules = {};
		function define(name, deps, impl) {
			for (var i=0; i<deps.length; i++) {
          deps[i] = modules[deps[i]];
      }
      modules[name] = impl.apply( impl, deps );
  }
	function get(name) { return modules[name];
	}
	return {
		define: define,
		get: get
	};
})();


MyModules.define( "bar", [], function() {
	function hello(who) {
		return "Let me introduce: " + who;
	}
		return {
			hello: hello
		};
	}
);
MyModules.define( "foo", ["bar"], function(bar) {
  var hungry = "hippo";
	function awesome() {
			console.log( bar.hello( hungry ).toUpperCase() );
	}
	return {
		awesome: awesome
	};
});
var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );
console.log(
   bar.hello( "hippo" )
); // Let me introduce: hippo foo.awesome(); // LET ME INTRODUCE: HIPPO
```

#### ES6模块

ES6 的模块没有“行内”格式，必须被定义在独立的文件中(一个文件一个模块)。浏览器或引擎有一个默认的“模块加载器”可以在导入模块时异步地加载模块文件。

import 可以将一个模块中的一个或多个 API 导入到当前作用域中，并分别绑定在一个变量上。module 会将整个模块的 API 导入并绑定到一个变量上。export 会将当前模块的一个标识符(变量、函数)导出为公共 API。这些操作可以在模块定义中根据需要使用任意多次。

模块文件中的内容会被当作好像包含在作用域闭包中一样来处理。

## 问题

**为什么不直接使用 IIFE 来创建作用域而使用try/catch？**

首先，try/catch 的性能的确很糟糕，但技术层面上没有合理的理由来说明 try/catch 必须这么慢，或者会一直慢下去。自从 TC39 支持在 ES6 的转换器中使用 try/catch 后， Traceur 团队已经要求 Chrome 对 try/catch 的性能进行改进，他们显然有很充分的动机来做这件事情。

其次，IIFE 和 try/catch 并不是完全等价的，因为如果将一段代码中的任意一部分拿出来用函数进行包裹，会改变这段代码的含义，其中的 this、return、break 和 contine 都会发生变化。IIFE 并不是一个普适的解决方案，它只适合在某些情况下进行手动操作。

## 资料

* [You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20&%20closures/README.md#you-dont-know-js-scope--closures)
* [Parsing JavaScript - better lazy than eager? | JSConf EU 2017](https://www.youtube.com/watch?v=Fg7niTmNNLg&t=784s)
* [JavaScript variables lifecycle: why let is not hoisted](https://dmitripavlutin.com/variables-lifecycle-and-why-let-is-not-hoisted/)
* [Function(MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)